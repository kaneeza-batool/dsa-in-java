### 1. Problem Statement

Given an array of strings `strs`, group the strings that are anagrams of each other and return the groups as a list of lists. Two strings are anagrams if they contain the same characters with the same counts, in any order. The groups, and the words inside each group, may be returned in any order.

**Key constraints:**

- `1 <= strs.length <= 10^4`
- `0 <= strs[i].length <= 100`, so an empty string is valid input.
- Strings contain only lowercase English letters.

**Example:**

- Input: `["eat","tea","tan","ate","nat","bat"]`
- Output: `[["bat"],["nat","tan"],["ate","eat","tea"]]`

### 2. Key Observations

- Anagrams contain exactly the same letters with the same frequencies. Only the order of the letters differs.
- Sorting the letters removes the order difference. Every anagram produces the same sorted string: `"eat"`, `"tea"` and `"ate"` all become `"aet"`.
- The sorted string can therefore act as a unique label (key) for each group.
- Comparing every word with every other word would be too slow for up to 10^4 words. A key-based lookup avoids pairwise comparison.
- Each key must collect multiple words, and the number of words per group is not known in advance. The value must be a growable list, not a count and not a fixed-size array.

### 3. Approach

1. Create a `HashMap<String, List<String>>`. The key is the sorted word and the value is the list of original words in that group.
2. Loop over every word in `strs`.
3. Build the key for the current word:
    1. Convert the word to a character array with `toCharArray()`.
    2. Sort the array in place with `Arrays.sort()`.
    3. Convert the sorted array back to a String with `new String(chars)`.
4. If the map does not contain the key, insert the key with a new empty `ArrayList`.
5. Retrieve the list for the key and add the original word (not the sorted key).
6. After the loop, return all map values wrapped in a new `ArrayList`.

### 4. Why This Approach Works

- Two strings are anagrams if and only if their sorted forms are identical. Sorting puts identical multisets of characters into one identical order, and different multisets can never produce the same sorted string.
- Every word is mapped to exactly one key, so every word lands in exactly one group.
- **Invariant:** after processing each word, every list in the map contains only words whose sorted form equals that list’s key, and every processed word appears in exactly one list.
- At the end of the loop, the map values are exactly the required anagram groups.

### 5. Java Solution

```java
class Solution {
    public List<List<String>> groupAnagrams(String[] strs) {
        Map<String, List<String>> map = new HashMap<>();

        for (String str: strs) {
            char[] chars = str.toCharArray();
            Arrays.sort(chars);
            String key = new String(chars);

            if (!map.containsKey(key)) {
                map.put(key, new ArrayList<>());
            }

            map.get(key).add(str);
        }

        return new ArrayList<>(map.values());
    }
}
```

### 6. Dry Run

Input: `["eat","tea","tan","ate","nat","bat"]`

| Step | Word | Sorted Key | Key Already in Map? | Map After Step |
| --- | --- | --- | --- | --- |
| 1 | `eat` | `aet` | No, new list created | `aet → [eat]` |
| 2 | `tea` | `aet` | Yes | `aet → [eat, tea]` |
| 3 | `tan` | `ant` | No, new list created | `aet → [eat, tea]`, `ant → [tan]` |
| 4 | `ate` | `aet` | Yes | `aet → [eat, tea, ate]`, `ant → [tan]` |
| 5 | `nat` | `ant` | Yes | `aet → [eat, tea, ate]`, `ant → [tan, nat]` |
| 6 | `bat` | `abt` | No, new list created | `aet → [eat, tea, ate]`, `ant → [tan, nat]`, `abt → [bat]` |

`map.values()` returns `[eat, tea, ate]`, `[tan, nat]` and `[bat]`.

**Final output:** `[["eat","tea","ate"],["tan","nat"],["bat"]]`. The order may differ, which is allowed.

### 7. Complexity Analysis

Let `n` be the number of words and `k` be the length of the longest word.

- **Time Complexity: O(n · k log k)**
    - Sorting one word costs O(k log k), and this is done once per word.
    - Converting to and from a character array costs O(k) per word.
    - Hashing a key of length k costs O(k) per map operation.
    - Sorting dominates.
- **Auxiliary Space Complexity: O(n · k)**
    - The map stores every original word plus one key per group, which is at most n · k characters in total.
    - The temporary character array per word is O(k).

### 8. Mistakes I Made

| Mistake | Cause | Correction | Lesson |
| --- | --- | --- | --- |
| Proposed `Integer` as the map value type. | Carried over the counting habit from Valid Anagram. | Changed to `List<String>`. | The value must hold the actual words, because the output returns words, not counts. |
| Then proposed `String`, then an array, as the value type. | Did not yet connect the value to the inner part of the return type. | Used `List<String>`, matching the inner type of `List<List<String>>`. | A single String holds one word. A Java array has a fixed size. A growable `ArrayList` fits an unknown group size. |
| Wrote `chars = Arrays.sort(chars);` | Assumed `sort` returns the sorted array. | Wrote `Arrays.sort(chars);` on its own line. | `Arrays.sort()` sorts in place and returns `void`. Assigning its result causes a compile error. |
| Wrote `Array.sort` instead of `Arrays.sort`. | Misremembered the class name. | Used `Arrays` (with s). | The utility class is `java.util.Arrays`. |
| Wrote `map.values` without parentheses. | Treated a method as a field. | Wrote `map.values()`. | Methods always need `()`, the same as `toCharArray()` and `containsKey()`. |
| Did not know how to convert a String to a character array, or a character array back to a String. | First time needing these conversions. | Used `toCharArray()` and `new String(chars)`. | These two conversions are needed whenever characters must be sorted or rearranged. |
| Did not know how to add an item to a list stored inside a map. | First time using a map and a list together. | Used `map.get(key).add(str);` | `map.get(key)` returns the list object, so a list method can be called on it directly. |

### 9. What I Learned

- **Sorted-key grouping pattern:** transform each item into a canonical form so that all items that belong together produce the same key, then group them with a HashMap.
- `HashMap<String, List<String>>` stores many values under one key: check the key, create the list once, then add to it.
- `str.toCharArray()` converts a String to a `char[]`.
- `Arrays.sort(arr)` sorts in place and returns nothing.
- `new String(chars)` or `String.valueOf(chars)` converts a `char[]` back to a String.
- `chars.toString()` does not return the characters. It returns an object reference such as `[C@1b6d3586`.
- `new ArrayList<>(map.values())` collects all map values into a list that matches the return type.
- The key is only an internal label. The output contains the original words, never the keys.