### 1. Problem Statement

Design a class `MyHashMap` without using any built-in hash table library. It must support three operations:

- `put(key, value)`: insert the pair. If `key` already exists, update its value.
- `get(key)`: return the value mapped to `key`, or `1` if `key` has no mapping.
- `remove(key)`: remove `key` and its value if present.

**Key constraints:**

- `0 <= key, value <= 10^6`
- At most `10^4` calls in total to `put`, `get` and `remove`.

**Example:**

- Operations: `put(1,1)`, `put(2,2)`, `get(1)`, `get(3)`, `put(2,1)`, `get(2)`, `remove(2)`, `get(2)`
- Results: `get(1) → 1`, `get(3) → -1`, `get(2) → 1`, `get(2) → -1` after the removal.

### 2. Key Observations

- Keys are non-negative integers in a small, fixed range (`0` to `10^6`). Each key can be given its own array slot, using the key as the index. This is the same direct-address idea as Design HashSet (705).
- Unlike a set, each slot must store a value, not just presence. So the array holds `int`, not `boolean`.
- A missing key must return `1`. Values are never negative, so `1` can never be a real value and works safely as the “no mapping” marker (a sentinel).
- `put` on an existing key simply overwrites the slot, which gives the required update behaviour for free.

### 3. Approach

1. Declare an `int[]` field so all methods share it.
2. In the constructor, create the array with size `1000001`, covering keys `0` to `1,000,000` inclusive.
3. Fill every slot with `1` using `Arrays.fill`, so the map starts empty.
4. `put(key, value)`: set `map[key] = value`.
5. `get(key)`: return `map[key]`. This is `1` if the key was never added or was removed.
6. `remove(key)`: set `map[key] = -1`.

### 4. Why This Approach Works

- **Invariant:** for every valid key, `map[key]` equals the current value for that key, or `1` if the key has no mapping.
- The constructor establishes the invariant: every slot starts at `1`, and the map is empty.
- `put` stores the latest value, which covers both inserting and updating. `remove` restores `1`. Both keep the invariant true.
- `get` returns the slot directly, which by the invariant is either the correct value or `1`.
- The sentinel is unambiguous only because every legal value is `>= 0`, so a stored value can never equal `1`.
- Each key has its own unique index, so keys never collide.

### 5. Java Solution

java

```java
class MyHashMap {

    int[] map;

    public MyHashMap() {
        map = new int[1000001];

        Arrays.fill(map, -1);
    }

    public void put(int key, int value) {
        map[key] = value;
    }

    public int get(int key) {
        return map[key];
    }

    public void remove(int key) {
        map[key] = -1;
    }
}

/**
 * Your MyHashMap object will be instantiated and called as such:
 * MyHashMap obj = new MyHashMap();
 * obj.put(key,value);
 * int param_2 = obj.get(key);
 * obj.remove(key);
 */
```

### 6. Dry Run

Only the slots that change are shown. Every other slot stays `-1`.

| Step | Operation | Action on Array | Relevant Slots After | Returned |
| --- | --- | --- | --- | --- |
| 1 | `MyHashMap()` | Create array, fill with `-1` | all `-1` | none |
| 2 | `put(1,1)` | `map[1] = 1` | `map[1]=1` | none |
| 3 | `put(2,2)` | `map[2] = 2` | `map[1]=1, map[2]=2` | none |
| 4 | `get(1)` | Read `map[1]` | unchanged | `1` |
| 5 | `get(3)` | Read `map[3]` | unchanged | `-1` |
| 6 | `put(2,1)` | `map[2] = 1` (update) | `map[1]=1, map[2]=1` | none |
| 7 | `get(2)` | Read `map[2]` | unchanged | `1` |
| 8 | `remove(2)` | `map[2] = -1` | `map[1]=1, map[2]=-1` | none |
| 9 | `get(2)` | Read `map[2]` | unchanged | `-1` |

### 7. Complexity Analysis

- **Time Complexity: O(1)** for `put`, `get` and `remove`. Each is a single array access by index.
- **Construction: O(U)**, where `U = 1,000,001`, because the array is allocated and filled with `1` once.
- **Auxiliary Space Complexity: O(U).** About 4 MB of `int` values, regardless of how many keys are stored.
- **Limitation:** this works only because keys are small and non-negative, and values are non-negative. Large or negative keys need a real hash map (a hash function, buckets and collision handling). Negative values would break the `1` sentinel.

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| Could not derive an approach. | Did not transfer the direct-address idea from Design HashSet (705), solved just before. | Learned the approach from a video lecture. | A map is a set that also stores a value. When a closely related problem was just solved, start from its solution and ask what must change. |
| Initialised the array with a manual loop. | Did not know the built-in method. | Replaced it with `Arrays.fill(map, -1);` | `Arrays.fill` sets every element of an array to one value in a single call. |
| Did not work out why `-1` is a safe “missing” marker. | Did not attempt the reasoning. | The value constraint `0 <= value` guarantees `-1` is never a real value. | A sentinel is safe only if the constraints guarantee it can never be a real value. Always check the value range before choosing one. |

### 9. What I Learned

- A direct-address array works for a map as well as a set. The slot stores the value instead of `true`/`false`.
- **Sentinel values:** a special value such as `1` can mark “missing”, but only when the constraints prove it can never be a real value.
- If values could be negative, presence would have to be tracked separately, for example with a parallel `boolean[]`.
- `Arrays.fill(array, value)` initialises an entire array in one line.
- Overwriting a slot on `put` naturally handles both insert and update.
- Design HashSet and Design HashMap differ in one idea: store presence versus store a value.