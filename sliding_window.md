# Sliding Window Technique

## What is the Sliding Window Technique?

The sliding window technique is a powerful algorithmic approach used to solve problems involving arrays, strings, or other sequential data structures. It's particularly effective when you need to find a contiguous subset (subarray or substring) that satisfies certain conditions.

The technique gets its name from the concept of a "window" that slides across the data structure, dynamically adjusting its size based on the problem requirements. This approach transforms what would typically be an O(n²) or O(n³) brute force solution into a more efficient O(n) algorithm.

## When to Use Sliding Window

The sliding window technique is ideal for problems that involve:
- Finding subarrays or substrings with specific properties
- Optimizing over all contiguous subarrays of a certain size
- Problems where you need to track elements within a range
- Scenarios involving "all subarrays" or "all substrings"

## Key Concepts

- **Window**: A contiguous portion of the array/string being examined
- **Left/Start Pointer**: Marks the beginning of the current window
- **Right/End Pointer**: Marks the end of the current window
- **Window Size**: Can be fixed or variable depending on the problem
- **Window State**: The current condition or properties of elements within the window

## Visual Understanding

### Basic Sliding Window Concept

```
Array: [1, 4, 2, 9, 5, 10, 3]
Window Size: 3

Step 1: Initial window
[1, 4, 2] 9  5  10  3
 ^     ^
left  right
Window sum = 1 + 4 + 2 = 7

Step 2: Slide window right
 1 [4, 2, 9] 5  10  3
    ^     ^
   left  right
Window sum = 4 + 2 + 9 = 15

Step 3: Continue sliding
 1  4 [2, 9, 5] 10  3
       ^     ^
      left  right
Window sum = 2 + 9 + 5 = 16

Step 4: Keep sliding
 1  4  2 [9, 5, 10] 3
          ^      ^
         left   right
Window sum = 9 + 5 + 10 = 24

Step 5: Final position
 1  4  2  9 [5, 10, 3]
             ^      ^
            left   right
Window sum = 5 + 10 + 3 = 18
```

### Two-Pointer Approach Visualization

```
Problem: Find longest substring without repeating characters
String: "abcabcbb"

Step 1: Start with empty window
a b c a b c b b
^
left/right (window = "")

Step 2: Expand window
a b c a b c b b
^ ^
L R (window = "a")

Step 3: Continue expanding
a b c a b c b b
^   ^
L   R (window = "ab")

Step 4: Keep expanding
a b c a b c b b
^     ^
L     R (window = "abc")

Step 5: Found duplicate 'a', shrink window
a b c a b c b b
  ^   ^
  L   R (window = "bca")

Step 6: Continue process...
a b c a b c b b
    ^ ^
    L R (window = "a")
```

## Step-by-Step Explanation

1. **Define the Window Boundaries**
   - Start with two pointers, typically called `start` and `end`, that mark the boundaries of the current window.
2. **Expand the Window**
   - Move the `end` pointer to include new elements until the window meets the desired condition.
3. **Process the Current Window**
   - Once the window satisfies the condition, record or update results based on the contents of the window.
4. **Shrink the Window**
   - Move the `start` pointer forward to reduce the window size when it no longer meets the requirement or when you want to look for smaller windows.
5. **Repeat**
   - Continue expanding and shrinking until the `end` pointer reaches the end of the data structure.

This approach keeps computations efficient because each element is visited at most twice, once by `end` and once by `start`.

## Types of Sliding Window Problems

### 1. Fixed-Size Window
The window size remains constant throughout the algorithm.

**Common Problems:**
- Maximum sum of subarray of size K
- Average of all subarrays of size K
- First negative number in every window of size K

**Template:**
```
for i from 0 to n-k:
    process window from i to i+k-1
```

### 2. Variable-Size Window (Two Pointers)
The window size changes dynamically based on certain conditions.

**Common Problems:**
- Longest substring without repeating characters
- Minimum window substring
- Longest subarray with sum ≤ K

**Template:**
```
left = 0
for right from 0 to n-1:
    add arr[right] to window
    while window is invalid:
        remove arr[left] from window
        left++
    update result with current window
```

### 3. Sliding Window Maximum/Minimum
Uses additional data structures (like deque) to efficiently track max/min in the current window.

**Common Problems:**
- Sliding window maximum
- Sliding window minimum

## Time and Space Complexity

### Time Complexity
- **Fixed Window**: O(n) where n is the length of the array/string
- **Variable Window**: O(n) where each element is visited at most twice
- **With Additional Data Structures**: O(n) to O(n log n) depending on the data structure used

### Space Complexity
- **Basic Sliding Window**: O(1) using only pointers
- **With Hash Map/Set**: O(k) where k is the size of the character set or window
- **With Deque/Stack**: O(k) where k is the maximum window size

### Why is it Efficient?
1. **Eliminates Redundant Computations**: Instead of recalculating everything for each window, we incrementally add/remove elements
2. **Single Pass**: Most sliding window problems can be solved in one pass through the data
3. **Optimal Substructure**: Results from previous windows can be used to compute the current window

## Practical Code Examples

### Example 1: Maximum Sum of Subarray of Size K (Fixed Window)

```python
def max_sum_subarray(arr, k):
    if len(arr) < k:
        return -1
    
    # Calculate sum of first window
    window_sum = sum(arr[:k])
    max_sum = window_sum
    
    # Slide the window
    for i in range(k, len(arr)):
        # Remove leftmost element and add rightmost element
        window_sum = window_sum - arr[i - k] + arr[i]
        max_sum = max(max_sum, window_sum)
    
    return max_sum

# Example usage
arr = [1, 4, 2, 9, 5, 10, 3]
k = 3
print(max_sum_subarray(arr, k))  # Output: 24 (9 + 5 + 10)
```

**Step-by-Step Diagram for Fixed Window:**

```
Array: [1, 4, 2, 9, 5, 10, 3], k = 3

Initial State:
╔═══╤═══╤═══╗───┬───┬────┬───
║ 1 │ 4 │ 2 ║ 9 │ 5 │ 10 │ 3
╚═══╧═══╧═══╝───┴───┴────┴───
 0   1   2   3   4    5   6
window_sum = 7, max_sum = 7

Step 1: i = 3 (k = 3)
    ╔═══╤═══╤═══╗───┬────┬───
 1  ║ 4 │ 2 │ 9 ║ 5 │ 10 │ 3
    ╚═══╧═══╧═══╝───┴────┴───
Remove arr[0] = 1, Add arr[3] = 9
window_sum = 7 - 1 + 9 = 15, max_sum = 15

Step 2: i = 4
        ╔═══╤═══╤═══╗────┬───
 1   4  ║ 2 │ 9 │ 5 ║ 10 │ 3
        ╚═══╧═══╧═══╝────┴───
Remove arr[1] = 4, Add arr[4] = 5
window_sum = 15 - 4 + 5 = 16, max_sum = 16

Step 3: i = 5
            ╔═══╤═══╤════╗───
 1   4   2  ║ 9 │ 5 │ 10 ║ 3
            ╚═══╧═══╧════╝───
Remove arr[2] = 2, Add arr[5] = 10
window_sum = 16 - 2 + 10 = 24, max_sum = 24 ← Maximum!

Step 4: i = 6
                ╔═══╤════╤═══╗
 1   4   2   9  ║ 5 │ 10 │ 3 ║
                ╚═══╧════╧═══╝
Remove arr[3] = 9, Add arr[6] = 3
window_sum = 24 - 9 + 3 = 18, max_sum remains 24

Final Answer: 24
```

### Example 2: Longest Substring Without Repeating Characters (Variable Window)

```python
def longest_substring_without_repeating(s):
    char_set = set()
    left = 0
    max_length = 0
    
    for right in range(len(s)):
        # Shrink window until no repeating characters
        while s[right] in char_set:
            char_set.remove(s[left])
            left += 1
        
        # Add current character to window
        char_set.add(s[right])
        max_length = max(max_length, right - left + 1)
    
    return max_length

# Example usage
s = "abcabcbb"
print(longest_substring_without_repeating(s))  # Output: 3 ("abc")
```

**Step-by-Step Diagram for Variable Window:**

```
String: "abcabcbb"
Goal: Find longest substring without repeating characters

Initial State:
a b c a b c b b
^
L=R=0, window="", char_set={}, max_length=0

Step 1: Add 'a'
a b c a b c b b
^ ^
L R   window="a", char_set={'a'}, max_length=1

Step 2: Add 'b' (no duplicate)
a b c a b c b b
^   ^
L   R   window="ab", char_set={'a','b'}, max_length=2

Step 3: Add 'c' (no duplicate)
a b c a b c b b
^     ^
L     R   window="abc", char_set={'a','b','c'}, max_length=3

Step 4: Try to add 'a' (DUPLICATE found!)
a b c a b c b b
^       ^
L       R   'a' already in char_set!

Step 5: Shrink window (remove left 'a')
a b c a b c b b
  ^     ^
  L     R   window="bc", char_set={'b','c'}

Step 6: Now add 'a' (no duplicate)
a b c a b c b b
  ^     ^
  L     R   window="bca", char_set={'b','c','a'}, max_length=3

Step 7: Try to add 'b' (DUPLICATE found!)
a b c a b c b b
  ^       ^
  L       R   'b' already in char_set!

Step 8: Shrink window (remove left 'b')
a b c a b c b b
    ^     ^
    L     R   window="ca", char_set={'c','a'}

Step 9: Now add 'b' (no duplicate)
a b c a b c b b
    ^     ^
    L     R   window="cab", char_set={'c','a','b'}, max_length=3

Continue this process...

Final State:
Maximum length found: 3 (substring "abc")

Window Expansion/Contraction Visualization:
═══════════════════════════════════════════
Expand →  ╔═══╤═══╤═══╗
          ║ a │ b │ c ║ a b c b b    Valid window (no duplicates)
          ╚═══╧═══╧═══╝

Conflict!  ╔═══╤═══╤═══╤═══╗
          ║ a │ b │ c │ a ║ b c b b   Invalid! 'a' appears twice
          ╚═══╧═══╧═══╧═══╝

Shrink ←      ╔═══╤═══╤═══╗
          a  ║ b │ c │ a ║ b c b b    Valid again
              ╚═══╧═══╧═══╝

Expand →      ╔═══╤═══╤═══╤═══╗
          a  ║ b │ c │ a │ b ║ c b b   Invalid! 'b' appears twice
              ╚═══╧═══╧═══╧═══╝

Shrink ←          ╔═══╤═══╤═══╗
          a  b   ║ c │ a │ b ║ c b b   Valid again
                  ╚═══╧═══╧═══╝
```

### Example 3: Minimum Window Substring (Advanced Variable Window)

```python
def min_window_substring(s, t):
    from collections import Counter, defaultdict
    
    if len(s) < len(t):
        return ""
    
    # Count characters in t
    dict_t = Counter(t)
    required = len(dict_t)
    
    # Sliding window
    left = right = 0
    formed = 0
    window_counts = defaultdict(int)
    
    # Answer tuple (window length, left, right)
    ans = float("inf"), None, None
    
    while right < len(s):
        # Add character to window
        character = s[right]
        window_counts[character] += 1
        
        # Check if frequency matches
        if character in dict_t and window_counts[character] == dict_t[character]:
            formed += 1
        
        # Try to shrink window
        while left <= right and formed == required:
            character = s[left]
            
            # Update answer if this window is smaller
            if right - left + 1 < ans[0]:
                ans = (right - left + 1, left, right)
            
            # Remove leftmost character
            window_counts[character] -= 1
            if character in dict_t and window_counts[character] < dict_t[character]:
                formed -= 1
            
            left += 1
        
        right += 1
    
    return "" if ans[0] == float("inf") else s[ans[1]:ans[2] + 1]

# Example usage
s = "ADOBECODEBANC"
t = "ABC"
print(min_window_substring(s, t))  # Output: "BANC"
```

**Advanced Window Expansion/Contraction Diagram:**

```
Problem: Minimum Window Substring
String s = "ADOBECODEBANC", Pattern t = "ABC"
Need to find: Minimum window containing all characters A, B, C

Phase 1: Expand until window is valid
═══════════════════════════════════════════

Step 1: Expand to find all required characters
A D O B E C O D E B A N C
^                     ^
L                     R
Window: "ADOBECODEBANC" ✓ Contains A, B, C (but not minimal)

Phase 2: Contract while maintaining validity
════════════════════════════════════════════

Step 2: Try to shrink from left
A D O B E C O D E B A N C
  ^                   ^
  L                   R
Window: "DOBECODEBANC" ✗ Missing 'A'

Step 3: Must include 'A', try different approach
A D O B E C O D E B A N C
^               ^
L               R
Window: "ADOBECODEB" ✗ Missing 'C'

Step 4: Expand to include 'C'
A D O B E C O D E B A N C
^                     ^
L                     R
Window: "ADOBECODEBANC" ✓ Valid but long

Step 5: Slide window to find "BANC"
A D O B E C O D E B A N C
                  ^     ^
                  L     R
Window: "BANC" ✓ Valid and minimal!

Final Visualization:
════════════════════
Original: A D O B E C O D E B A N C
Result:   ╔═════════════════════════╗
          ║     Found: "BANC"      ║  ← Minimum window
          ╚═════════════════════════╝

Window State Transitions:
═══════════════════════════
INVALID → EXPAND → VALID → CONTRACT → OPTIMAL
   ↓         ↓        ↓         ↓         ↓
  Empty   Growing   Found    Shrinking  Minimal
```

## Window Behavior Patterns

```
Fixed Window Pattern:
═══════════════════════
[■■■]□□□□□□  →  □[■■■]□□□□□  →  □□[■■■]□□□□  →  □□□[■■■]□□□
 Window      Window         Window         Window
 size = 3    slides right   slides right   slides right

Variable Window Pattern:
═══════════════════════════
Expanding:
[■]□□□□□□□□  →  [■■]□□□□□□□  →  [■■■]□□□□□□  →  [■■■■]□□□□□

Contracting (when condition violated):
[■■■■■]□□□□  →  □[■■■■]□□□□  →  □□[■■■]□□□□  →  □□□[■■]□□□□

Dynamic Expansion/Contraction:
Window grows and shrinks based on problem conditions
═══════════════════════════════════════════════════════
Valid:   [■■■]          Invalid:  [■■■■■]
         ↓ expand                 ↓ contract
       [■■■■]                   [■■■■]
         ↓ expand more            ↓ contract more
       [■■■■■]                  [■■■]
       Invalid!                 Valid again!
```

## Common Problem Patterns and Variations

### 1. Maximum/Minimum in Fixed Window
- **Pattern**: Track max/min value in every window of size K
- **Key Insight**: Use deque to maintain elements in decreasing/increasing order
- **Examples**: Sliding Window Maximum, Sliding Window Minimum

### 2. Subarray with Target Sum
- **Pattern**: Find subarray with sum equal to target
- **Key Insight**: Use prefix sum and hash map, or two pointers for positive numbers
- **Examples**: Subarray Sum Equals K, Maximum Size Subarray Sum Equals K

### 3. Longest Subarray/Substring with Condition
- **Pattern**: Find longest contiguous sequence satisfying a condition
- **Key Insight**: Expand window when condition is met, shrink when violated
- **Examples**: Longest Substring Without Repeating Characters, Longest Subarray with At Most K Distinct Characters

### 4. Shortest Subarray/Substring with Condition
- **Pattern**: Find minimum length contiguous sequence satisfying a condition
- **Key Insight**: Expand until condition is met, then shrink while maintaining condition
- **Examples**: Minimum Window Substring, Shortest Subarray with Sum At Least K

### 5. Count of Subarrays/Substrings
- **Pattern**: Count number of contiguous sequences satisfying a condition
- **Key Insight**: For each ending position, count valid starting positions
- **Examples**: Number of Subarrays with Bounded Maximum, Count Number of Nice Subarrays

### 6. Permutation in String
- **Pattern**: Check if any permutation of pattern exists in text
- **Key Insight**: Use character frequency matching with sliding window
- **Examples**: Permutation in String, Find All Anagrams in a String

### 7. Substring with All Characters
- **Pattern**: Find substring containing all characters from a set
- **Key Insight**: Track character frequencies and required character count
- **Examples**: Minimum Window Substring, Smallest Range Covering Elements from K Lists

## Tips and Best Practices

### 1. Identify the Problem Type
- **Fixed Window**: Look for keywords like "size K", "window of length K"
- **Variable Window**: Look for "longest", "shortest", "maximum", "minimum" with conditions
- **Multiple Pointers**: Problems involving finding pairs, triplets, or multiple subarrays

### 2. Choose the Right Data Structure
- **Basic problems**: Use simple variables to track window state
- **Character frequency**: Use hash map or array (for ASCII characters)
- **Max/Min in window**: Use deque or monotonic stack
- **Multiple conditions**: Combine multiple data structures

### 3. Handle Edge Cases
- Empty input arrays/strings
- Window size larger than input size
- Single element arrays
- All elements are the same
- Negative numbers (for sum-based problems)

### 4. Optimization Techniques
- **Early termination**: Stop when optimal solution is found
- **Preprocessing**: Use prefix sums or other preprocessed data
- **Character arrays**: Use arrays instead of hash maps for fixed character sets (ASCII)
- **Avoid redundant operations**: Cache frequently used calculations

### 5. Debugging Tips
- Print window boundaries and current state
- Verify that both pointers move correctly
- Check if window expansion/contraction logic is correct
- Ensure all edge cases are handled
- Test with small examples first

### 6. Common Mistakes to Avoid
- **Off-by-one errors**: Be careful with array indices and window boundaries
- **Infinite loops**: Ensure pointers always make progress
- **Wrong window size**: Double-check window size calculations
- **Missing updates**: Don't forget to update result variables
- **Incorrect conditions**: Verify window validity conditions

### 7. Problem-Solving Approach
1. **Understand the problem**: Identify what you're looking for
2. **Choose window type**: Fixed vs variable size
3. **Define window state**: What information to track
4. **Write the template**: Start with basic sliding window structure
5. **Handle specifics**: Add problem-specific logic
6. **Test thoroughly**: Use various test cases

## Practice Problems

### Beginner Level
1. **Maximum Average Subarray I** - Find contiguous subarray of length k with maximum average
2. **Contains Duplicate II** - Check if array contains duplicate within k distance
3. **Repeated DNA Sequences** - Find all 10-letter DNA sequences that occur more than once
4. **Longest Harmonious Subsequence** - Find longest subsequence where max and min differ by exactly 1

### Intermediate Level
1. **Longest Substring Without Repeating Characters** - Classic variable window problem
2. **Longest Repeating Character Replacement** - Replace at most k characters to get longest repeating substring
3. **Permutation in String** - Check if permutation of string s1 exists in s2
4. **Find All Anagrams in a String** - Find all anagram occurrences of pattern in text
5. **Minimum Size Subarray Sum** - Find minimum length subarray with sum ≥ target
6. **Fruit Into Baskets** - Pick fruits from at most 2 types of trees

### Advanced Level
1. **Minimum Window Substring** - Find minimum window containing all characters of pattern
2. **Sliding Window Maximum** - Find maximum in every window of size k
3. **Longest Substring with At Most K Distinct Characters** - Self-explanatory
4. **Subarrays with K Different Integers** - Count subarrays with exactly k distinct integers
5. **Minimum Number of K Consecutive Bit Flips** - Complex bit manipulation with sliding window
6. **Constrained Subsequence Sum** - Dynamic programming with sliding window optimization

### LeetCode Problem Numbers
- Easy: 643, 219, 187, 594
- Medium: 3, 424, 567, 438, 209, 904, 76, 239, 340, 992
- Hard: 995, 1425

## Summary

The sliding window technique is a fundamental algorithmic approach that can dramatically improve the efficiency of problems involving contiguous subarrays or substrings. By maintaining a window that slides across the data structure and avoiding redundant computations, it transforms many O(n²) problems into O(n) solutions.

Key takeaways:
- **Identify the pattern**: Look for problems involving contiguous elements with conditions
- **Choose the right approach**: Fixed vs variable window size
- **Master the templates**: Practice the basic patterns until they become second nature
- **Handle edge cases**: Always consider boundary conditions and special cases
- **Practice regularly**: The more problems you solve, the easier it becomes to recognize when to use this technique

Remember, sliding window is not just about the algorithm itself, but about recognizing when it's the right tool for the job. With practice, you'll develop an intuition for these types of problems and be able to solve them efficiently.
