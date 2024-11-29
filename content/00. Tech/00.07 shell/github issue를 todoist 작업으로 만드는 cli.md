---
date: 2024-11-29
publish: false
tags:
---
```table-of-contents
title: 
style: nestedList # TOC style (nestedList|nestedOrderedList|inlineFirstLevel)
minLevel: 0 # Include headings from the specified level
maxLevel: 0 # Include headings up to the specified level
includeLinks: true # Make headings clickable
hideWhenEmpty: false # Hide TOC if no headings are found
debugInConsole: false # Print debug info in Obsidian console
```
## Installation

```sh
brew install rust
cargo install doist
brew install gh
```


```sh
#!/usr/bin/env bash
# ~/.local/bin/doissue

# Show usage if no argument provided
if [[ -z "$1" ]]; then
    echo "Usage: doissue <issue-number>"
    exit 1
fi

# Validate input is a number
if ! [[ "$1" =~ ^[0-9]+$ ]]; then
    echo "Error: Issue number must be a positive integer"
    exit 1
fi

issue_num="$1"

# Check if gh command exists
if ! command -v gh >/dev/null 2>&1; then
    echo "Error: GitHub CLI (gh) is not installed"
    exit 1
fi

# Check if doist command exists
if ! command -v doist >/dev/null 2>&1; then
    echo "Error: doist command is not installed"
    exit 1
fi

# Fetch issue information
if ! result=$(gh issue view "$issue_num" --json title,url,state); then
    echo "Error: Unable to fetch issue #$issue_num"
    exit 1
fi

# Debug output
if [[ -n "$DEBUG" ]]; then
    echo "Raw JSON output:"
    echo "$result"
fi

# Extract fields using awk
title=$(echo "$result" | awk -F'"title":"' '{print $2}' | awk -F'","' '{print $1}')
url=$(echo "$result" | awk -F'"url":"' '{print $2}' | awk -F'","' '{print $1}' | awk -F'"}' '{print $1}')
state=$(echo "$result" | awk -F'"state":"' '{print $2}' | awk -F'"' '{print $1}')
repo=$(echo "$url" | awk -F'github.com/' '{print $2}' | awk -F'/issues/' '{print $1}')

# Verify we got all required fields
if [[ -z "$title" || -z "$url" || -z "$state" ]]; then
    echo "Error: Missing required fields from GitHub issue"
    echo "Title: $title"
    echo "URL: $url"
    echo "State: $state"
    exit 1
fi

if [[ "$state" != "OPEN" ]]; then
    echo "Warning: Issue #$issue_num is $state"
    read -p "Continue anyway? (y/N) " -n 1 -r
    echo
    if [[ ! $REPLY =~ ^[Yy]$ ]]; then
        exit 1
    fi
fi

formatted_title="[$repo] $title"

# Execute the Doist command with properly quoted arguments
if ! doist a "$formatted_title" --desc "$url"; then
    echo "Error: Failed to create Doist task"
    exit 1
fi

echo "Successfully created Doist task for issue #$issue_num"
```