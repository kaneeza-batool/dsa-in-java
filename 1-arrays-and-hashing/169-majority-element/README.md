### 1. Problem Statement

Given an integer array `nums` of size `n`, return the majority element. The majority element appears **more than** `n / 2` times. The problem guarantees that a majority element always exists.

**Key constraints:**

- `1 <= n <= 5 * 10^4`
- `10^9 <= nums[i] <= 10^9`, so values can be negative and too large to index an array by value.
- A majority element is always present.

**Examples:**

- Input: `[3,2,3]` → Output: `3`
- Input: `[2,2,1,1,1,2,2]` → Output: `2`

### 2. Key Observations

- The answer depends only on how many times each value appears, so this is a frequency-counting problem.
- The value range is huge and includes negatives. A fixed-size frequency array (like `int[26]` in Valid Anagram) will not work, so a `HashMap` is needed.
- “More than n/2” means only one value can ever satisfy the condition.
- As soon as any value’s count exceeds `n / 2`, it must be the answer, so the scan can stop early.

### 3. Approach

1. Create a `HashMap<Integer, Integer>` mapping each value to its count.
2. Loop over every `num` in `nums`.
3. Increase the count for `num` by one, using `getOrDefault(num, 0)` so a first occurrence starts from 0.
4. If the updated count is greater than `nums.length / 2`, return `num` immediately.
5. After the loop, return `1`. This line is never reached for valid input, but Java requires it.

### 4. Why This Approach Works

- **Invariant:** after processing each element, the map holds the exact count of every value seen so far.
- A count only increases. The first moment any value’s count exceeds `n / 2`, that value appears more than half the time in the whole array, so it is the majority element.
- The guarantee ensures the majority element’s count crosses `n / 2` at some point during the loop, so a value is always returned from inside the loop.
- Integer division makes `nums.length / 2` round down. Using strict `>` correctly matches “more than half” for both odd and even `n`.

### 5. Java Solution

java

```java
class Solution {
    public int majorityElement(int[] nums) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int num:nums) {
            map.put(num, map.getOrDefault(num, 0)+1);

            if(map.get(num) > nums.length/2) {
                return num;
            }
        }

        return -1;
    }
}
```

### 6. Dry Run

Input: `[2,2,1,1,1,2,2]`. `n = 7`, so the threshold is `7 / 2 = 3`, and a count must be greater than 3.

| Step | num | Count After Update | Count > 3? | Map After Step |
| --- | --- | --- | --- | --- |
| 1 | 2 | 1 | No | `{2:1}` |
| 2 | 2 | 2 | No | `{2:2}` |
| 3 | 1 | 1 | No | `{2:2, 1:1}` |
| 4 | 1 | 2 | No | `{2:2, 1:2}` |
| 5 | 1 | 3 | No | `{2:2, 1:3}` |
| 6 | 2 | 3 | No | `{2:3, 1:3}` |
| 7 | 2 | 4 | Yes | `{2:4, 1:3}` |

**Return:** `2`

### 7. Complexity Analysis

Let `n` be the length of `nums`.

- **Time Complexity: O(n).** A single pass, and each `put`, `get` and `getOrDefault` is O(1) on average.
- **Auxiliary Space Complexity: O(n).** In the worst case the map stores up to about `n / 2` distinct values.
- **Optimality:** O(n) time is optimal, since every element may need to be read. The space is not optimal. The LeetCode follow-up asks for O(1) space, which is achievable with the Boyer-Moore Voting Algorithm (not yet studied).

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| Did not know what `map.get()` returns. | Limited prior use of HashMap methods. | `map.get(key)` returns the value stored for the key, or `null` if the key is absent. | Know what `get` returns for a missing key. Using `map.get(x) + 1` on a missing key throws a `NullPointerException`. |
| Thought the final `return -1;` was needed for edge cases. | Mixed up the problem’s guarantee with the compiler’s rules. | The compiler requires a return on every path. It cannot know that the loop always returns early. | A non-`void` Java method must return a value on every code path. Otherwise compilation fails with “missing return statement”. |

No mistakes in the solution logic itself.

### 9. What I Learned

- `map.get(key)` returns the value for the key, or `null` when the key does not exist.
- `map.getOrDefault(key, 0)` is the safe way to read a count before incrementing it.
- Counting pattern: `map.put(x, map.getOrDefault(x, 0) + 1);`
- Use a `HashMap` for frequency counting when values are large, negative or unbounded. Use a fixed-size array only when the value range is small and known.
- Returning early, as soon as the condition is met, avoids unnecessary work.
- Java requires a `return` on every path of a non-`void` method, even when the problem guarantees the path is never taken.
- An O(1) space solution exists (Boyer-Moore Voting) and should be studied as a follow-up.