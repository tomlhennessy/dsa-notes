# Array & Linked List Problems



## Maximum Subarray: Kadane's Algorithm

### Problem Summary
Find the `maximum sum` of any `contiguous subarray` in a given array of integers (may include negatives).


### Key Pattern / Idea
Use a `sliding window (implicit)` to track a current running subarray sum. At each step, either:
- **extend the current subarray** by including the current element, or
- **start a new subarray** from the current element (if the previous subarray is harmful i.e. sum becomes worse with it)
Track the best results seen so far with a global max.

### Code
```java
int currentMax = nums[0];
int globalMax = nums[0];

for (int i = 1; i < nums.length; i++) {
  currentMax = Math.max(currentMax + nums[i], nums[i]);
  if (currentMax > globalMax) {
    globalMax = currentMax;
  }
}

return globalMax;
```

### Common Pitfalls
- Starting at loop `i = 0` instead of `1`
- Not intialising `currentMax` and `globalMax` to `nums[0]` (using 0 will break all-negative arrays)


### Transferable Takeaway
Use local **greedy choices** and **global tracking** when:
- you can define a step-wise decision (e.g. extend vs reset)
- each step's optimal subsolution helps build the global solution




## Majority Element: Boyer-Moore Voting Algorithm

### Problem Summary
Given an array of size `n`, find the element that appears more than `[n / 2]` times.
You may assume such an element always exists.

### Key Pattern / Idea
> If one element appears more than half the time, it can "cancel out" all other elements and still be left standing.

Loop through the array with:
- candidate (element that's being tracked)
- count (how many net votes that candidate has)

> Think of votes: +1 for your candidate, -1 for others. <br>
> The majority element can't be outvoted if it appears more than [n / 2] times.

### Code
```java
int candidate = nums[0];
int count = 0;

for (int num : nums) {
  if (count == 0) {
    candidate = num;
  }

  if (num == candidate) {
    count++;
  } else {
    count--;
  }
}

return candidate;
```

### Common Pitfalls
- Updating count before assigning a new candidate when `count = 0`. Causes voting against the candidate you just selected.
- Forgetting that this algorithm works only works when a **majority element** is guaranteed. Otherwise you must do a second pass to verify.

### Transferable Takeaway
Use **greedy cancel-out** logic when:
- you can treat the problem as "voting" or "pairing off" competing values
- one element is guaranteed to have more than all the others combined

This approach shows up in variants like:
- elements showing up more than [n / 3] times (extended Boyer-Moore)
- cancelling frequency-based dominance in stream processing



## Reverse a Linked List (Iterative)

### Problem Summary
Reverse a singly linked list in-place. The tail becomes the new head, and every `.next` pointer is reversed

### Key Pattern / Idea
Use a 3-pointer strategy (`prev`, `curr`, `next`) to walk the list and reverse links in-place. Think of it like walking forward while turning the arrows backward.

### Code
```java
ListNode prev = null;
ListNode curr = head;

while (curr != null) {
  ListNode next = curr.next;  // save next node
  curr.next = prev;           // reverse pointer
  prev = curr;                // advance prev
  curr = next;                // advance curr
}

return prev;                  // new head of reversed list
```

### Common Pitfalls
- Forgetting to store `curr.next` before overwriting it
- Returning `curr` instead of `prev`
- Accidentally using `.prev` in a singly linked list

### Transferable Takeaway
Reversal problems often need:
- Careful pointer tracking to avoid data loss
- Clear seperation between reading (`next`) and writing (`.next`) steps



## Reverse a Linked List (Recursive)

### Problem Summary
Reverse a singly linked list using recursion. The tail becomes the new head, and all .next pointers are reversed.


### Code
```java
public ListNode reverseList(ListNode head) {
  if (head == null || head.next == null) {
    return head;
  }

  ListNode newHead = reversList(head.next);

  head.next.next = head;  // Point next node back to current
  head.next = null;       // Sever original forward link

  return newHead;         // always return tail (new head)
}
```

### How It Works
- Recursively reverse the rest of the list starting from `head.next`
- Once recursion returns:
  - Point `head.next.next` back to `head`
  - Sever `head.next` to avoid cycle
- Propogate the same `newHead` (original tail) up at each level


### Common Pitfalls
- Forgetting to set `head.next = null` (leads to cycles)
- Confusing `head` with `newHead`
- Thinking a new head is created at each step (**only the tail becomes the new head**)

### Transferable Takeaway
Use recursion when:
- you can solve the rest of the problem and connect the result to the current piece
- you're rebuilding structures from the **bottom-up** (classic in linked-lists, trees, and recursion heavy problems)
Think in terms of **solve the subproblem, then attach the current node correctly**



## Detect Cycle in Linked List: Floyd's Cycle Detection Algorithm

### Problem Summary
Determine if a singly linked list contains a cycle - that is, if any node's .next pointer eventually loops back to an earlier node instead of reaching null.

### Key Pattern / Idea
Use two pointers moving at different speeds:
 - if there is a cycle, the fast pointer will eventually lap the slow pointer -> they will meet
 - if there is no cycle, fast will hit null and terminate

This works because:
  - in a loop, the relative distance between fast and slow shrinks each round
  - once fast catches slow, you've proven a cycle exists


### Code
```java
ListNode slow = head;
ListNode fast = head;

while (fast != null && fast.next != null) {
  slow = slow.next;
  fast = fast.next.next;

  if (slow == fast) {
    return true;
  }
}

return false;
```

### Common Pitfalls
- Not checking `fast != null` and `fast.next != null`
- Putting `slow == fast` at the start - need to move before checking
- Using extra space (e.g. HashSet) when O(1) is possible with Floyd's

### Transferable Takeaway
When traversing cyclic or repeating structures, consider **fast/slow pointer techniques**.
This applies to:
- cycle detection in graphs
- detecting repetition in numeric sequences (e.g. happy numbers)
- detecting loops in data structures without modifying them or using extra space
