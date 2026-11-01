### 1. Problem Statement

Given an integer array `nums` and an integer `k`, return the `k` most frequent elements. The answer may be returned in any order.

**Key constraints:**

- `1 <= nums.length <= 10^5`
- `10^4 <= nums[i] <= 10^4`, so values can be negative.
- `k` is between 1 and the number of unique elements.
- The answer is guaranteed to be unique.

**Example:**

- Input: `nums = [1,1,1,2,2,3]`, `k = 2`
- Output: `[1,2]`

**Follow-up:** the algorithm's time complexity must be better than O(n log n).

### 2. Key Observations

- The answer depends only on how often each value appears, so this starts as a frequency-counting problem.
- Values can be negative, so a fixed-size array indexed by value will not work. A `HashMap` is needed.
- `k` is an input and can be any size. Fixed variables like `mostFrequent` and `secondMostFrequent` only work when k = 2. I need a structure that can return any number of results.
- Once the counts are known, the problem becomes: order the unique values by count and take the top k.
- A count can never be more than `nums.length`. This allows a faster bucket approach (see section 7).

### 3. Approach

1. Create a `HashMap<Integer, Integer>` mapping each value to its count.
2. Loop over every `num` in `nums` and increase its count with `map.getOrDefault(num, 0) + 1`.
3. Copy all unique values (the map keys) into a list with `new ArrayList<>(map.keySet())`.
4. Sort the list by count, highest first, using the comparator `(a, b) -> map.get(b) - map.get(a)`.
5. Create an `int[]` of size `k` and copy the first `k` values from the sorted list into it.
6. Return the array.

### 4. Why This Approach Works

- **Invariant:** after the counting loop, the map holds the exact count of every unique value.
- After sorting by count in descending order, the first `k` values in the list are the `k` values with the highest counts.
- The problem guarantees a unique answer, so ties at the k-th position never change the result.
- The comparator `map.get(b) - map.get(a)` returns a positive number when `b` has a higher count than `a`, which puts `b` first. That gives descending order.

### 5. Java Solution

```java
class Solution {
    public int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> map = new HashMap<>();

        for (int num: nums) {
            map.put(num, map.getOrDefault(num, 0) + 1);
        }

        // Convert keys into list
        List<Integer> list = new ArrayList<>(map.keySet());

        list.sort((a, b) -> map.get(b) - map.get(a));

        int[] result = new int[k];

        for (int i=0; i<k; i++) {
            result[i] = list.get(i);
        }

        return result;
    }
}
```

### 6. Dry Run

Input: `nums = [1,1,1,2,2,3]`, `k = 2`

| Step | Action | State After Step |
| --- | --- | --- |
| 1 | Count `1` | `{1:1}` |
| 2 | Count `1` | `{1:2}` |
| 3 | Count `1` | `{1:3}` |
| 4 | Count `2` | `{1:3, 2:1}` |
| 5 | Count `2` | `{1:3, 2:2}` |
| 6 | Count `3` | `{1:3, 2:2, 3:1}` |
| 7 | Copy keys to list | `[1, 2, 3]` (order may vary) |
| 8 | Sort by count, highest first | `[1, 2, 3]` (counts 3, 2, 1) |
| 9 | Take first `k = 2` | `result = [1, 2]` |

**Return:** `[1, 2]`

### 7. Complexity Analysis

Let `n` be the length of `nums` and `u` be the number of unique values (`u <= n`).

- **Time Complexity: O(n + u log u), which is O(n log n) in the worst case.**
    - Counting is one pass: O(n).
    - Sorting the unique values: O(u log u). When every value is different, u = n.
    - Copying k values: O(k).
- **Auxiliary Space Complexity: O(n).** The map and the list each hold up to `u` entries.
- **Optimality:** not optimal. The follow-up asks for better than O(n log n). **Bucket sort** achieves O(n):
    - A count can never be more than `n`, so create `List<Integer>[] buckets = new List[n + 1]`, where index = count.
    - Put each value into `buckets[count]`.
    - Walk from index `n` down to 1 and collect values until there are `k`.
    - Time O(n), space O(n). This is the version to use on the re-solve.

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| Wrote `getOrDefault(num, 0)` without the map. | Treated it as a standalone function. | Wrote `map.getOrDefault(num, 0)`. | `getOrDefault` is a `Map` method. It must be called on the map object, the same as `map.get()` and `map.put()`. |
| Planned two variables, `mostFrequent` and `secondMostFrequent`. | Built the solution around the example where k = 2. | Used a list sorted by count, so any `k` works. | When an input like `k` controls how many results are needed, the result must live in a structure that can grow, not in fixed variables. |
| Did not use the bucket approach. | Sorting was easier to think of. | Sorting passes, but bucket sort is the expected O(n) answer. | Read the follow-up before choosing an approach. A constraint like "better than O(n log n)" rules out sorting. |

### 9. What I Learned

- **Top-K pattern:** count with a HashMap, then pick the top k by count. Three ways to pick: sort (O(n log n)), min-heap of size k (O(n log k)), bucket sort (O(n)).
- `new ArrayList<>(map.keySet())` copies all map keys into a list.
- `list.sort((a, b) -> ...)` sorts a list with a custom rule. A lambda comparator returns negative to put `a` first, positive to put `b` first.
- `(a, b) -> map.get(b) - map.get(a)` sorts by count in **descending** order. Swapping `a` and `b` gives ascending order.
- Subtraction in a comparator can overflow for very large numbers. `Integer.compare(map.get(b), map.get(a))` is the safe version. Here counts are at most 10^5, so subtraction is fine.
- Bucket sort works when the values being sorted (here, counts) have a small known maximum. Index = value, so no comparison sort is needed.
- `List<Integer>[] buckets = new List[n + 1];` creates an array of lists. Each slot starts as `null` and needs a new `ArrayList` before adding.