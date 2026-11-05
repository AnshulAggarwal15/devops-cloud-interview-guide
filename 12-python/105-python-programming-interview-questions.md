<!-- #genai: Python programming interview problems with worked solutions, from chanduthedev Medium series + high-frequency additions, for Docusign R1 coding round -->

# Python Interview Questions — Programming (Part 2 of 2)

> Worked coding problems from the chanduthedev Medium series (real questions from Altimetrik, Foraysoft,
> TrackMind, Crisil, Quentelli) plus common additions. Pairs with
> [04-python-conceptual-interview-questions.md](./04-python-conceptual-interview-questions.md).
>
> **How to use each problem:** state brute force out loud → optimize (can a dict/set kill the nested loop?) →
> code → test 1 normal + 1 edge case → state time/space. The interviewer scores *reasoning*, not just the
> answer (see the [Altimetrik experience](https://chanduthedev.medium.com/my-interview-experience-with-altimetrik-for-python-developer-e315c7948d8d) — clues are expected).
>
> Sources: [Programming Part 1](https://chanduthedev.medium.com/python-interview-questions-in-programming-part-1-6535fcf6cc4e) ·
> [Programming Part 2](https://chanduthedev.medium.com/python-interview-questions-in-programming-part2-23c298bc89b9)

---

## A. String problems

### 1. Remove all consecutive equal pairs, repeat until none remain (Altimetrik)
Use a **stack**: if the top equals the current char, pop; else push. O(n).
```python
def remove_adjacent_pairs(s):
    stack = []
    for ch in s:
        if stack and stack[-1] == ch:
            stack.pop()          # pair cancels out
        else:
            stack.append(ch)
    return "".join(stack)

# "aabbcc" -> ""   |   "abba" -> ""   |   "abbac" -> "ac"
```

### 2. Is one string a subsequence of another (same order)? (Altimetrik)
Letters appear in order but not necessarily contiguous. Two pointers / iterator trick. O(n).
```python
def is_subsequence(sub, main):
    it = iter(main)
    return all(ch in it for ch in sub)

# is_subsequence("ace", "abcde") -> True   |   "aec" -> False
```

### 3. Check palindrome (Foraysoft)
```python
def is_palindrome(s):
    return s == s[::-1]
```
Two-pointer variant if they forbid slicing:
```python
def is_palindrome(s):
    i, j = 0, len(s) - 1
    while i < j:
        if s[i] != s[j]:
            return False
        i, j = i + 1, j - 1
    return True
```

### 4. Most frequent and 2nd most frequent char + counts (Foraysoft)
```python
from collections import Counter

def top_two_chars(s):
    return Counter(s).most_common(2)   # [(char, count), (char, count)]
```

### 5. Are two strings anagrams?
```python
from collections import Counter

def are_anagrams(a, b):
    return Counter(a) == Counter(b)    # or sorted(a) == sorted(b)
```

### 6. From an alphanumeric string, extract integers + unique count (Quentelli)
```python
import re

def integers_and_unique(s):
    nums = [int(n) for n in re.findall(r"\d+", s)]
    return nums, len(set(nums))

# "a12b3c12" -> ([12, 3, 12], 2)
```

### 7. Round-robin interleave of a list of strings (Foraysoft)
Take the 1st char of each, then the 2nd of each, etc.
```python
from itertools import zip_longest

def round_robin(strings):
    out = []
    for column in zip_longest(*strings, fillvalue=""):
        out.extend(column)
    return "".join(out)

# ["abc", "de", "fghi"] -> "adfbegchi"
```

---

## B. List / array problems

### 8. Rearrange so all negatives come first, then positives (order not required) (Altimetrik)
Brute force = sort O(n log n). Optimal = **two pointers**, O(n), in place. (The interviewer's clue was
"use pointers.")
```python
def negatives_first(nums):
    i, j = 0, len(nums) - 1
    while i < j:
        if nums[i] < 0:
            i += 1
        elif nums[j] >= 0:
            j -= 1
        else:
            nums[i], nums[j] = nums[j], nums[i]
            i, j = i + 1, j - 1
    return nums
```

### 9. Group consecutive-number runs (TrackMind-style)
`[1,2,3,6,8,9,5,6] -> [[1,2,3],[8,9],[5,6]]` (single numbers dropped).
```python
def consecutive_runs(nums):
    runs = []
    for x in nums:
        if runs and x == runs[-1][-1] + 1:
            runs[-1].append(x)
        else:
            runs.append([x])
    return [r for r in runs if len(r) > 1]
```

### 10. Position-wise combine three equal-length lists (TrackMind)
`a=[1,2,3], b=[4,5,6], c=[7,8,9] -> [[1,4,7],[2,5,8],[3,6,9]]`
```python
def zip_positions(a, b, c):
    return [list(t) for t in zip(a, b, c)]
```

### 11. Sort numerator & denominator lists by fraction value (Altimetrik)
Basic version:
```python
def sort_by_fraction(nums, denoms):
    pairs = sorted(zip(nums, denoms), key=lambda p: p[0] / p[1])
    n, d = zip(*pairs)
    return list(n), list(d)
```
**Hard mode (no built-in sort, no division):** compare fractions by **cross-multiplication**
(`a/b < c/d  ⇔  a*d < c*b`, for positive denominators) inside a manual sort:
```python
def sort_by_fraction_manual(nums, denoms):
    pairs = list(zip(nums, denoms))
    for i in range(len(pairs)):                 # simple bubble/selection sort
        for j in range(len(pairs) - 1 - i):
            a, b = pairs[j]
            c, d = pairs[j + 1]
            if a * d > c * b:                   # a/b > c/d  -> swap
                pairs[j], pairs[j + 1] = pairs[j + 1], pairs[j]
    n, d = zip(*pairs)
    return list(n), list(d)
```

### 12. Second largest element
```python
def second_largest(nums):
    first = second = float("-inf")
    for x in nums:
        if x > first:
            first, second = x, first
        elif first > x > second:
            second = x
    return second
```

### 13. Remove duplicates preserving order
```python
def dedupe(items):
    seen = set()
    return [x for x in items if not (x in seen or seen.add(x))]
```

### 14. Flatten a nested list (recursion)
```python
def flatten(lst):
    out = []
    for x in lst:
        out.extend(flatten(x) if isinstance(x, list) else [x])
    return out
```

---

## C. Dictionary problems

### 15. Print all keys of a deeply nested dict (recursion) (TrackMind)
`{1:{2:{3:"hi"},4:{5:"Hello"}}} -> [1,2,3,4,5]`
```python
def all_keys(d):
    keys = []
    for k, v in d.items():
        keys.append(k)
        if isinstance(v, dict):
            keys.extend(all_keys(v))
    return keys
```

### 16. Sort a dict by the nth element of its list-value (TrackMind)
`{"a":[1,10,3], "b":[5,0,7], "c":[7,1,9]}` sorted by index 1 → `{"b":..., "c":..., "a":...}`
```python
def sort_by_nth(d, n):
    return dict(sorted(d.items(), key=lambda kv: kv[1][n]))
```

### 17. Word frequency counter (very common; DevOps log flavor)
```python
from collections import Counter

def word_counts(text):
    return Counter(text.split())
```

---

## D. Recursion / generators

### 18. Fibonacci with recursion + memoization (Crisil)
```python
from functools import lru_cache

@lru_cache(maxsize=None)
def fib(n):
    return n if n < 2 else fib(n - 1) + fib(n - 2)
```
Manual memo (if they forbid `lru_cache`):
```python
def fib(n, memo={}):
    if n < 2:
        return n
    if n not in memo:
        memo[n] = fib(n - 1, memo) + fib(n - 2, memo)
    return memo[n]
```
Talking track: naive recursion is O(2ⁿ); memoization makes it **O(n)** by caching subproblems.

### 19. Infinite sequence with a generator
```python
def naturals():
    n = 1
    while True:
        yield n
        n += 1
```

### 20. Implement a simple decorator (asked in almost every interview)
```python
def my_decorator(func):
    def wrapper(*args, **kwargs):
        print("before")
        result = func(*args, **kwargs)
        print("after")
        return result
    return wrapper

@my_decorator
def greet(name):
    print(f"Hello, {name}")
```

---

## E. Extra high-frequency problems (added — expect these)

### 21. Two Sum — the "hash map kills the nested loop" classic
```python
def two_sum(nums, target):
    seen = {}                       # value -> index
    for i, n in enumerate(nums):
        if target - n in seen:
            return [seen[target - n], i]
        seen[n] = i
    return []
```

### 22. First non-repeating character
```python
from collections import Counter

def first_unique(s):
    counts = Counter(s)
    for ch in s:
        if counts[ch] == 1:
            return ch
    return None
```

### 23. Valid parentheses (stack)
```python
def valid_parens(s):
    pairs = {")": "(", "]": "[", "}": "{"}
    stack = []
    for ch in s:
        if ch in "([{":
            stack.append(ch)
        elif not stack or stack.pop() != pairs[ch]:
            return False
    return not stack
```

### 24. Group anagrams
```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)
    for w in words:
        groups["".join(sorted(w))].append(w)
    return list(groups.values())
```

### 25. FizzBuzz (warm-up they still ask)
```python
def fizzbuzz(n):
    for i in range(1, n + 1):
        print("FizzBuzz" if i % 15 == 0 else
              "Fizz" if i % 3 == 0 else
              "Buzz" if i % 5 == 0 else i)
```

### 26. DevOps-flavored: count HTTP status codes in a log file
Stream the file so memory stays flat for GB-size logs. O(n).
```python
from collections import Counter

def status_counts(path):
    counts = Counter()
    with open(path) as f:
        for line in f:               # streams line by line
            parts = line.split()
            if parts:
                counts[parts[-1]] += 1   # assume status is the last field
    return counts
```

### 27. DevOps-flavored: find lines containing a pattern (e.g. "404")
```python
def grep(path, pattern):
    with open(path) as f:
        return [line.rstrip() for line in f if pattern in line]
```

---

## The REST request lifecycle question (asked in the Altimetrik interview)
> *"What's the lifecycle of a REST call from client request to response in Python (Flask)?"*

Request hits the WSGI server (gunicorn) → routed by the framework to the matching view based on the URL/verb
→ **middleware/decorators** run first (**authentication**, **token validation**, **rate limiting**) → the
view executes business logic (DB/cache calls) → returns a response object → serialized (JSON) back to the
client. **Auth is commonly done with a decorator** wrapping the endpoint.

---

## Quick recall — the instincts they test
- **Hash map (dict/set)** to turn O(n²) into O(n) — Two Sum, first-unique, anagrams.
- **Two pointers** for in-place array rearrangement/palindrome.
- **Stack** for matching/cancelling (parentheses, adjacent pairs).
- **`Counter` / `defaultdict`** for counting/grouping.
- **Generators / streaming** for large files.
- **Memoization** for expensive recursion.
- Always: **brute force → optimize → complexity**, narrated aloud.
