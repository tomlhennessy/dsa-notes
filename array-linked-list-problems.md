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



## Reverse a Linked List

### Problem Summary
Reverse a singly-linked list so that the tail becomes the new head, and each node's `.next` pointer is flipped to point to the previous node.

### Iterative Approach
```java
ListNode prev = null;
ListNode curr = head;

while (curr != null) {
  ListNode next = curr.next;
  curr.next = prev;
  prev = curr;
  curr = next;
}

return prev;
```

### Common Pitfalls (iterative)
- Forgetting to store `curr.next` before overwriting it
- Returning `curr` instead of `prev`
- Getting `prev`/`curr` pointer updates in the wrong order
