### 1. Problem Statement

Given an integer array `nums` and an integer `val`, remove every occurrence of `val` **in place** and return `k`, the number of elements that are not equal to `val`. The first `k` positions of `nums` must hold the kept elements. Their order may change, and whatever remains after position `k` does not matter.

**Key constraints:**

- `0 <= nums.length <= 100`
- `0 <= nums[i] <= 50`
- `0 <= val <= 100`
- The array must be modified in place. No extra array may be returned.

**Examples:**

- Input: `nums = [3,2,2,3]`, `val = 3` → Output: `2`, with `nums` starting `[2,2,...]`
- Input: `nums = [0,1,2,2,3,0,4,2]`, `val = 2` → Output: `5`, with `nums` starting with `0, 1, 3, 0, 4` in any order

### 2. Key Observations

- Only the first `k` positions are checked. The elements after them can be left as they are, so nothing has to be physically deleted or shifted.
- The task can be reframed: copy every element that should be kept to the front of the array, in the order it is found.
- Two positions are needed. One reads every element, and one marks where the next kept element should be written.
- The write position can never move ahead of the read position. Overwriting at the write position is therefore always safe.
- Shifting the remaining elements left after each removal works in principle, but it is slow (O(n²)) and easy to get wrong.

### 3. Approach

1. Initialise a write pointer `k = 0`.
2. Loop over the array with a read pointer `i` from `0` to `nums.length - 1`.
3. If `nums[i]` is not equal to `val`:
    1. Copy it to the write position: `nums[k] = nums[i]`.
    2. Move the write pointer forward: `k++`.
4. If `nums[i]` equals `val`, skip it. Only `i` moves forward.
5. After the loop, return `k`.

### 4. Why This Approach Works

- **Invariant:** before each iteration, `nums[0..k-1]` contains exactly the kept elements found among `nums[0..i-1]`, in their original order.
- Each kept element increases `k` by one, and each `val` element is skipped. Positions `0` to `k-1` therefore only ever receive kept elements.
- `k` increases at most once per step of `i`, so `k <= i` always holds. A write to `nums[k]` only overwrites a position that `i` has already read, so no unread element is ever lost. When `k == i`, the element is copied onto itself.
- When the loop ends, `i` has read every element. By the invariant, the first `k` positions hold every kept element, and `k` is their count.

### 5. Java Solution

java

```java
class Solution {
    public int removeElement(int[] nums, int val) {

        int k = 0;
        for(int i=0; i<nums.length; i++) {
            if (nums[i] != val) {
                nums[k] = nums[i];
                k++;
            }
        }

        return k;
    }
}
```

### 6. Dry Run

Input: `nums = [3,2,2,3]`, `val = 3`

| Step | i | nums[i] | Kept? | Action | k After | nums After |
| --- | --- | --- | --- | --- | --- | --- |
| 1 | 0 | 3 | No | Skip | 0 | `[3,2,2,3]` |
| 2 | 1 | 2 | Yes | `nums[0] = 2` | 1 | `[2,2,2,3]` |
| 3 | 2 | 2 | Yes | `nums[1] = 2` | 2 | `[2,2,2,3]` |
| 4 | 3 | 3 | No | Skip | 2 | `[2,2,2,3]` |

**Return:** `2`. The first 2 elements are `[2,2]`, which is correct.

### 7. Complexity Analysis

Let `n` be the length of `nums`.

- **Time Complexity: O(n).** A single pass. Each element is read once, and at most one write happens per element.
- **Auxiliary Space Complexity: O(1).** Only two integer variables are used. The array is modified in place.
- **Comparison:** the first attempt, which shifted elements left on every match, was O(n²) time.

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| The first attempt shifted all remaining elements left each time `val` was found. | Took “remove” literally, as physically deleting from the array. | Switched to copying kept elements forward with a write pointer. | In-place removal usually means overwriting the front of the array, not deleting. |
| After shifting, the loop still ran `i++`, so the element shifted into `nums[i]` was never checked. For `[2,2,3]` with `val = 2`, the result still contained a `2` in the first `k` positions. | Did not account for the array changing under the loop index. | Removed the shifting entirely, which removed the bug. | When a loop modifies the array at the current index, check whether the new value at that index still needs processing. A dry run on a case with consecutive targets exposes this. |
| The shifting approach was O(n²). | Every removal moved up to n elements. | The two-pointer approach is O(n). | Nested loops that move elements are a signal to look for a pointer-based single pass. |
| Described `k` as “always behind” `i`. | Missed the equality case. | `k <= i`: behind or equal. | When stating an invariant, include the boundary case. |

### 9. What I Learned

- **Two pointers (read/write pattern):** one pointer reads every element, and a second, slower pointer writes the elements to keep. This filters an array in place in a single pass.
- The returned count `k` doubles as the write pointer. No separate counter is needed.
- Overwriting in place is safe as long as the write pointer never passes the read pointer.
- Problems that check only the first `k` elements allow the rest of the array to be ignored, which removes the need for shifting.
- Testing with consecutive target values, such as `[2,2,3]`, is a good way to catch skipped-element bugs.