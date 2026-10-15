### 1. Problem Statement

Design a class `MyHashSet` without using any built-in hash table library. It must support three operations:

- `add(key)`: insert `key` into the set.
- `remove(key)`: remove `key` from the set. If `key` is absent, do nothing.
- `contains(key)`: return `true` if `key` is in the set, otherwise `false`.

A set stores each key at most once. Adding an existing key has no extra effect.

**Key constraints:**

- `0 <= key <= 10^6`
- At most `10^4` calls in total to `add`, `remove` and `contains`.

**Example:**

- Operations: `add(1)`, `add(2)`, `contains(1)`, `contains(3)`, `add(2)`, `contains(2)`, `remove(2)`, `contains(2)`
- Results: `contains(1) → true`, `contains(3) → false`, `contains(2) → true`, `contains(2) → false` after the removal.

### 2. Key Observations

- Keys are non-negative integers in a small, fixed, known range (`0` to `10^6`).
- A set only needs to answer one question per key: present or not present. That is a single `true`/`false` value.
- Because the range is fixed, every possible key can be given its own position in an array, using the key itself as the index. This is the same idea as the `int[26]` frequency array in Valid Anagram, applied to a larger range.
- Duplicate adds are handled automatically: setting `true` twice leaves the value `true`.
- An array of size 1,000,001 booleans is about 1 MB, which is acceptable for this range.

### 3. Approach

1. Declare a `boolean[]` field in the class so all methods share it.
2. In the constructor, create the array with size `1000001`, covering keys `0` to `1,000,000` inclusive. Java initialises every element to `false`, meaning the set starts empty.
3. `add(key)`: set `set[key] = true`.
4. `remove(key)`: set `set[key] = false`.
5. `contains(key)`: return `set[key]`.

### 4. Why This Approach Works

- **Invariant:** for every key in the valid range, `set[key]` is `true` exactly when the key is currently in the set.
- The constructor establishes the invariant: all values start `false`, and the set is empty.
- `add` makes the value `true`. `remove` makes it `false`. Each operation keeps the invariant true.
- `contains` reads that single value, which by the invariant is the correct answer.
- Each key maps to its own unique index, so different keys never interfere with each other. No collision handling is needed.

### 5. Java Solution

java

```java
class MyHashSet {

    boolean[] set;

    public MyHashSet() {
        set = new boolean[1000001];
    }

    public void add(int key) {
        set[key] = true;
    }

    public void remove(int key) {
        set[key] = false;
    }

    public boolean contains(int key) {
        return set[key];
    }
}

/**
 * Your MyHashSet object will be instantiated and called as such:
 * MyHashSet obj = new MyHashSet();
 * obj.add(key);
 * obj.remove(key);
 * boolean param_3 = obj.contains(key);
 */
```

### 6. Dry Run

Only the slots that change are shown. Every other slot stays `false`.

| Step | Operation | Action on Array | Relevant Slots After | Returned |
| --- | --- | --- | --- | --- |
| 1 | `MyHashSet()` | Create array of 1,000,001 `false` | all `false` | none |
| 2 | `add(1)` | `set[1] = true` | `set[1]=true` | none |
| 3 | `add(2)` | `set[2] = true` | `set[1]=true, set[2]=true` | none |
| 4 | `contains(1)` | Read `set[1]` | unchanged | `true` |
| 5 | `contains(3)` | Read `set[3]` | unchanged | `false` |
| 6 | `add(2)` | `set[2] = true` (already true) | unchanged | none |
| 7 | `contains(2)` | Read `set[2]` | unchanged | `true` |
| 8 | `remove(2)` | `set[2] = false` | `set[1]=true, set[2]=false` | none |
| 9 | `contains(2)` | Read `set[2]` | unchanged | `false` |

### 7. Complexity Analysis

- **Time Complexity: O(1)** for `add`, `remove` and `contains`. Each one is a single array access by index.
- **Construction: O(U)**, where `U = 1,000,001`, because Java allocates and zeroes the whole array once.
- **Auxiliary Space Complexity: O(U).** The array size depends on the key range, not on how many keys are stored. Even a set with one key uses about 1 MB.
- **Limitation:** this works only because the key range is small, fixed and non-negative. For large or negative keys, a real hash set (a hash function mapping keys into buckets, with collision handling) is required.

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| Could not derive an approach and had no starting idea. | First design-style problem. Did not connect it to the fixed-range array idea from Valid Anagram. | Learned the direct-address boolean array from guided explanation and a video lecture. | When the value range is small and known, consider using the value itself as an array index. |

No mistakes in the final code.

### 9. What I Learned

- **Direct-address table:** when keys fall in a small, fixed, non-negative range, an array indexed by the key gives O(1) insert, delete and lookup.
- A set only needs presence information, so a `boolean[]` is enough. A count is not needed.
- In Java, a new `boolean[]` has every element initialised to `false` automatically.
- For an inclusive range `0` to `N`, the array size must be `N + 1`. Otherwise index `N` throws `ArrayIndexOutOfBoundsException`.
- In a design problem, data shared by several methods is declared as a class field and created in the constructor.
- This approach trades memory for speed. It is not true hashing. A real HashSet uses a hash function and buckets, which is the next concept to study for this problem.