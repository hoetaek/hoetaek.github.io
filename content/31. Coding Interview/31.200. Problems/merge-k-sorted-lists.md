leetcode

링크: https://leetcode.com/problems/merge-k-sorted-lists/submissions/

You are given an array of `k` linked-lists `lists`, each linked-list is sorted in ascending order.

_Merge all the linked-lists into one sorted linked-list and return it._

**Example 1:**

**Input:** lists = \[\[1,4,5],\[1,3,4],\[2,6]]
**Output:** \[1,1,2,3,4,4,5,6]
**Explanation:** The linked-lists are:
[
  1->4->5,
  1->3->4,
  2->6
]
merging them into one sorted list:
1->1->2->3->4->4->5->6

**Example 2:**

**Input:** lists = \[]
**Output:** \[]

**Example 3:**

**Input:** lists = \[\[]]
**Output:** \[]

**Constraints:**

- `k == lists.length`
- `0 <= k <= 104`
- `0 <= lists[i].length <= 500`
- `-104 <= lists[i][j] <= 104`
- `lists[i]` is sorted in **ascending order**.
- The sum of `lists[i].length` will not exceed `104`.

## code
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def mergeKLists(self, lists: List[Optional[ListNode]]) -> Optional[ListNode]:
        if not lists or len(lists) == 0:
            return

        def sort(lists, L, R):
            if L == R:
                return lists[L]
            
            M = (L + R) // 2
            left = sort(lists, L, M)
            right = sort(lists, M + 1, R)

            return self.merge(left, right)
        
        return sort(lists, 0, len(lists) - 1)



    def merge(self, left, right):
        dummy = ListNode()
        curr = dummy
        while left and right:
            if left.val < right.val:
                curr.next = left
                left = left.next
            else:
                curr.next = right
                right = right.next
            curr = curr.next
        
        if left:
            curr.next = left

        if right:
            curr.next = right

        return dummy.next
```
