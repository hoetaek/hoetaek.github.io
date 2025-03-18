---
date: 2025-03-05
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
https://news.hada.io/topic?id=19564&utm_source=slack&utm_medium=bot&utm_campaign=T02ATH4Q4

- 반복적인 작업을 사람이 직접 수행하는 것은 비효율적이며, 자동화가 필요함
- 예를 들어, macOS의 기본 .MOV 동영상을 .MP4로 변환하는 과정이 번거로웠음
    - 기존에는 GUI 변환 앱을 사용했지만, 여러 단계를 거쳐야 함
    - 이를 해결하기 위해 **macOS 폴더 액션(Folder Actions)**을 활용하여 자동 변환 시스템을 구축
- 추가로 한 작업들: 비디오-to-GIF 변환, 이미지-to-WebP 변환, Youtube 비디오 다운로드, Twitter 비디오 다운로드, Youtube 비디오-to-MP3 변환

## 폴더 액션을 이용한 자동 변환

- 특정 폴더에 파일을 드롭하면 자동으로 변환 작업이 실행됨
- 예제:
    - .MOV → .MP4 변환
    - .JPG → .WEBP 변환
    - Twitter 및 YouTube 동영상 다운로드
- **폴더에 파일을 넣는 것만으로 변환이 완료됨**, 원본 파일도 자동 삭제됨

## macOS 폴더 액션 설정 방법

### 주의사항

- 폴더 액션을 설정한 후 폴더 이름을 변경하면 동작하지 않음.
- 폴더 이름을 변경하면 다시 액션을 재설정해야 함.

### 설정 절차

1. **터미널에서 폴더 생성**
2. **Automator 실행 후 새 "Folder Action" 생성**
3. **"Get Selected Finder Items" 및 "Run Shell Script" 추가**
    - **Pass input:** "as arguments" 설정
4. **변환 스크립트 입력**
    
    - 예제: .MOV → .MP4 변환
    
    ```bash
    for f in "$@"; do  
        /opt/homebrew/bin/ffmpeg -n -loglevel error -i "$f" -vcodec libx264 -crf 23 -preset ultrafast -tune film "/Users/alexander/Library/Mobile\ Documents/com\~apple\~CloudDocs/Downloads/$(date +"%Y_%m_%d_%I_%M_%p_%s").mp4";  
        rm -f "$f"  
    done  
    ```
    
5. **저장 후 종료**
6. **폴더에 .MOV 파일을 드래그 앤 드롭하면 자동 변환 실행**
    - 실행 중에는 메뉴바에 **기어 아이콘**이 표시됨.

## 추가 폴더 액션 예제

### 동영상 → GIF 변환

```bash
for f in "$@"; do  
    /opt/homebrew/bin/ffmpeg -n -loglevel error -i "$f" -vf "fps=18,scale=720:-1:flags=lanczos" "/Users/alexander/Library/Mobile Documents/com~apple~CloudDocs/Downloads/$(date +"%Y_%m_%d_%I_%M_%p_%s").gif";  
    rm -f "$f"  
done  
```

### 이미지 → WEBP 변환

```bash
for f in "$@"; do  
    /opt/homebrew/bin/cwebp -q 70 "$f" -o "/Users/alexander/Library/Mobile Documents/com~apple~CloudDocs/Downloads/$(date +"%Y_%m_%d_%I_%M_%p_%s").webp";  
    rm -f "$f"  
done  
```

### YouTube 동영상 다운로드

브라우저에서 이 폴더로 그냥 URL을 Drag & Drop 하면 다운로드 시작

```bash
for f in "$@"; do  
    url=$(grep -o '<string>.*</string>' "$f" | sed 's/<string>\(.*\)<\/string>/\1/')  
    if [ -n "$url" ]; then  
        /opt/homebrew/bin/yt-dlp -P "~/Downloads" "$url"  
        if [ $? -eq 0 ]; then  
            rm -f "$f"  
        fi  
    fi  
done  
```

### Twitter 동영상 다운로드

```bash
for f in "$@"; do  
    url=$(grep -o '<string>.*</string>' "$f" | sed 's/<string>\(.*\)<\/string>/\1/')  
    if [ -n "$url" ]; then  
        /opt/homebrew/bin/yt-dlp -P "~/Downloads" "$url"  
        if [ $? -eq 0 ]; then  
            rm -f "$f"  
        fi  
    fi  
done  
```

### YouTube → MP3 변환

```bash
brew install yt-dlp; brew install ffmpeg  
```

```bash
for f in "$@"; do  
    url=$(grep -o '<string>.*</string>' "$f" | sed 's/<string>\(.*\)<\/string>/\1/')  
    if [ -n "$url" ]; then  
        /opt/homebrew/bin/yt-dlp -x --audio-format mp3 --audio-quality 0 --ffmpeg-location /opt/homebrew/bin/ffmpeg -P "~/Downloads" "$url"  
        if [ $? -eq 0 ]; then  
            rm -f "$f"  
        fi  
    fi  
done  
```

### 폴더 액션 변경 방법

- 폴더 액션을 수정하려면 **폴더에서 우클릭 → "Folder Action Setup" 선택**
- 저장된 모든 액션은 다음 경로에 있음:
    
    ```undefined
    Macintosh HD / Users / YourName / Library / Workflows / Applications / Folder Actions/  
    ```
    
- 이 시스템을 활용하면 **각각의 폴더를 터미널 명령어의 인터페이스로 변환 가능**
- 덕분에 **데스크톱이 훨씬 더 유용한 작업 공간이 됨**