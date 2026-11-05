# Docusign R1 — Coding Practice Workbook

Fill in each answer in the empty Python block. For **every** problem also write the **time/space complexity** and a one-line **talking track** (brute force -> optimization). Do them out loud.

Workflow per problem: restate -> brute force -> optimize (can a dict/set kill the nested loop?) -> code -> test 1 normal + 1 edge case -> state Big-O.

---

## Part A — Warm-up drills (write from scratch, no lookups)

No platform needed — just write and run these locally.

### A1. Count frequencies

Given a list of items, return a mapping of item -> count.

```python
# your answer
# l = list
d={}
for i in l:
    if i in d:
        d[i]+=1
    else:
        d[i]=1

return d

```

Complexity / notes:

# TC: O(n); M: O(n)

---

### A2. Reverse a string

Return the reversed string (know both the slice trick and a manual two-pointer loop).

```python
# your answer
# s-string
# brute force

c=list(s)
for i in range(n//2):
    x=c[i]
    c[i]=c[n-i-1]
    c[n-i-1]=x
return c.join('')

# TC: O(n/2); M: O

s=s[::-1]
#  no idea



```

Complexity / notes:

---

### A3. Merge two dicts

Merge `a` and `b`; on key conflict, `b` wins. Show the `{**a, **b}` way and a loop way.

```python
# your answer
c={}
for i in a:
    c[i]=a[i]

for i in b:
    c[i]=b[i]

time = na+nb
mem = na+nb

IDK about {**a, **b}

```

Complexity / notes:

---

### A4. Group anagrams

Given `["eat","tea","tan","ate","nat","bat"]`, group words that are anagrams of each other.

```python
# your answer
n=len(l)
ans=[]

for i in range(n):
    x=[l[i]]
    for j in range(i+1, n):
        if (len(l[i]) == len(l[j])) and (l[i] in l[j]):
            x.append(l[j])
            del l[j]
    
    ans.append(x)


```

## Complexity / notes:

TC: O(n^2); MC: N

### A5. Read JSON and extract a field

Given a JSON string (or file), parse it and pull out a nested field. Bonus: handle a missing key gracefully.

```python
# your answer
import json
def df(file):
    data=json.load(open("file.json"))

    req_key=data.get(key,{})
    if(req_key=={}):
        return "key doesn't exists"
    else:
        req_key

```

Complexity / notes:

---

## Part B — Core problems (LeetCode)

Do the first five first — they carry the "hash map kills the nested loop" instinct they want to see.

### B1. Two Sum

[https://leetcode.com/problems/two-sum/](https://leetcode.com/problems/two-sum/)
Return indices of the two numbers that add up to `target`. Example: `nums=[2,7,11,15], target=9 -> [0,1]`.
Hint: store `value -> index` in a dict; look for `target - n`.

```python
def two_sum(nums, target):
    # your answer
    pass
```

Complexity / talking track:

---

### B2. Valid Anagram

[https://leetcode.com/problems/valid-anagram/](https://leetcode.com/problems/valid-anagram/)
Return True if `t` is an anagram of `s`. Example: `s="anagram", t="nagaram" -> True`.
Hint: compare `Counter(s) == Counter(t)`, or sort both.

```python
from collections import Counter

def is_anagram(s, t):
    if len(s) != len(t):        # quick reject; also lets you short-circuit
        return False
    return Counter(s) == Counter(t)
    # alt one-liner: return sorted(s) == sorted(t)
```

Complexity / talking track:

- Counter: **O(n) time, O(1) space** (bounded alphabet) — one pass to count, then compare maps.
- sorted() version: O(n log n) time. Talking track: "Anagrams have identical character multisets, so I compare frequency maps — O(n) — instead of sorting, which is O(n log n)."

---

### B3. Contains Duplicate

[https://leetcode.com/problems/contains-duplicate/](https://leetcode.com/problems/contains-duplicate/)
Return True if any value appears at least twice. Example: `[1,2,3,1] -> True`.
Hint: a set; compare `len(set(nums)) != len(nums)`.

```python
def contains_duplicate(nums):
    # your answer
    pass
```

Complexity / talking track:

---

### B4. First Unique Character in a String

[https://leetcode.com/problems/first-unique-character-in-a-string/](https://leetcode.com/problems/first-unique-character-in-a-string/)
Return the index of the first non-repeating character, or -1. Example: `"leetcode" -> 0`, `"loveleetcode" -> 2`.
Hint: count chars, then scan for the first with count 1.

```python
def first_uniq_char(s):
    # your answer
    pass
```

Complexity / talking track:

---

### B5. Valid Parentheses

[https://leetcode.com/problems/valid-parentheses/](https://leetcode.com/problems/valid-parentheses/)
Given a string of `()[]{}`, return True if brackets are balanced/correctly nested. Example: `"()[]{}" -> True`, `"(]" -> False`.
Hint: stack; push openers, pop and match on closers.

```python
def is_valid(s):
    # your answer
    pass
```

Complexity / talking track:

---

### B6. Best Time to Buy and Sell Stock

[https://leetcode.com/problems/best-time-to-buy-and-sell-stock/](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/)
One buy + one later sell; return max profit (0 if none). Example: `[7,1,5,3,6,4] -> 5`.
Hint: track running min price and best profit in one pass.

```python
def max_profit(prices):
    # your answer
    pass
```

Complexity / talking track:

---

### B7. Merge Sorted Array

[https://leetcode.com/problems/merge-sorted-array/](https://leetcode.com/problems/merge-sorted-array/)
Merge `nums2` into `nums1` in-place; `nums1` has trailing zeros as space. Example: `nums1=[1,2,3,0,0,0], m=3, nums2=[2,5,6], n=3 -> [1,2,2,3,5,6]`.
Hint: fill from the back with two pointers.

```python
def merge(nums1, m, nums2, n):
    # your answer (in-place)
    pass
```

Complexity / talking track:

---

### B8. Reverse Linked List

[https://leetcode.com/problems/reverse-linked-list/](https://leetcode.com/problems/reverse-linked-list/)
Reverse a singly linked list; return new head. Example: `1->2->3 => 3->2->1`.
Hint: iterate with `prev`, `curr`, flipping `next`.

```python
# class ListNode: def __init__(self, val=0, next=None): ...
def reverse_list(head):
    # your answer
    pass
```

Complexity / talking track:

---

### B9. Middle of the Linked List

[https://leetcode.com/problems/middle-of-the-linked-list/](https://leetcode.com/problems/middle-of-the-linked-list/)
Return the middle node (second middle if even length). Example: `1->2->3->4->5 -> node 3`.
Hint: slow/fast pointers (fast moves 2x).

```python
def middle_node(head):
    # your answer
    pass
```

Complexity / talking track:

---

### B10. Binary Search

[https://leetcode.com/problems/binary-search/](https://leetcode.com/problems/binary-search/)
Return the index of `target` in a sorted array, else -1. Example: `nums=[-1,0,3,5,9,12], target=9 -> 4`.
Hint: `lo/hi` pointers; be careful with `mid` and loop bounds.

```python
def search(nums, target):
    # your answer
    pass
```

Complexity / talking track:

---

## Part C — DevOps-flavored variants (most likely style for this role)

No single LeetCode link — practice with a sample file. Create a small `app.log` to test against.

### C1. Count HTTP status codes in a log file

Read a (potentially huge) log file and count how many times each HTTP status code appears.
Hint: stream with `with open(path) as f:` so memory stays flat; assume status is a known field per line.

```python
from collections import Counter

def status_counts(path):
    counts = Counter()
    with open(path) as f:                 # stream line-by-line; memory stays flat
        for line in f:
            parts = line.split()
            if parts:
                counts[parts[-1]] += 1     # assume status code is the last field
    return counts
```

Complexity / talking track:

- **O(n) time** over lines, **O(k) space** where k = number of distinct codes.
- Talking track: "I stream the file with `with open(...)` so a multi-GB log never loads fully into memory; a `Counter` gives O(1) increments. If the status is at a fixed column, I'd index it directly; otherwise a small regex."

---

### C2. Find matching / duplicate lines

Return all lines containing a pattern (e.g. `"404"`), or find lines that appear more than once.

```python
from collections import Counter

# (a) all lines containing a pattern
def find_lines(path, pattern):
    with open(path) as f:
        return [line.rstrip("\n") for line in f if pattern in line]

# (b) lines that appear more than once
def duplicate_lines(path):
    counts = Counter()
    with open(path) as f:
        for line in f:
            counts[line.rstrip("\n")] += 1
    return [line for line, c in counts.items() if c > 1]
```

Complexity / talking track:

- (a) **O(n) time, O(m) space** (m = matches); streamed, so working memory is flat.
- (b) **O(n) time, O(u) space** (u = unique lines). Talking track: "For 'appears more than once' I count with a dict/Counter in one pass, then filter for count > 1."

---

### C3. Read a CSV / JSON and extract fields

Parse a CSV or JSON file and pull out specific fields (e.g. all user emails, or sum a numeric column).

```python
import csv, json

# CSV: collect one column (e.g. "email")
def emails_from_csv(path):
    with open(path, newline="") as f:
        reader = csv.DictReader(f)                     # streams row by row
        return [row["email"] for row in reader if row.get("email")]

# CSV: sum a numeric column
def sum_column_csv(path, column):
    total = 0.0
    with open(path, newline="") as f:
        for row in csv.DictReader(f):
            total += float(row[column])
    return total

# JSON: extract a field from a list of records
def field_from_json(path, field):
    with open(path) as f:
        data = json.load(f)                            # loads whole file
    return [rec.get(field) for rec in data]
```

Complexity / talking track:

- **O(n)** over rows/records. `csv.DictReader` streams (flat memory); `json.load` reads the whole file (use `ijson` for huge JSON).
- Talking track: "`DictReader` gives me dict-per-row so I reference columns by name; `.get` keeps it safe if a field is missing."

---

### C4. Call an API and parse the response

Hit an endpoint (`requests.get`), check status, parse JSON, and extract a field. Handle non-200 gracefully.

```python
import requests

def fetch_and_parse(url):
    resp = requests.get(url, timeout=10)     # always set a timeout
    resp.raise_for_status()                  # raises on non-2xx
    data = resp.json()
    return data.get("some_field")

# Error-tolerant variant (returns value, error) instead of raising
def fetch_safe(url):
    try:
        resp = requests.get(url, timeout=10)
        resp.raise_for_status()
    except requests.RequestException as e:
        return None, str(e)
    return resp.json(), None
```

Complexity / talking track:

- O(1) logic; cost is the network call.
- Talking track: "Set a timeout so a hung endpoint can't block forever, check the status (`raise_for_status`), parse JSON, and `.get` the field. In real tooling I'd add retries with backoff for flaky/rate-limited APIs."

---

## Extra practice (optional platforms)

- LeetCode "Top Interview 150" / Easy list: [https://leetcode.com/studyplan/top-interview-150/](https://leetcode.com/studyplan/top-interview-150/)
- NeetCode 150 (grouped by pattern, great explanations): [https://neetcode.io/practice](https://neetcode.io/practice)
- HackerRank Python track: [https://www.hackerrank.com/domains/python](https://www.hackerrank.com/domains/python)

---

*After you fill answers in, tell me and I'll review them: correctness, edge cases, complexity, and how you'd narrate each in the interview.*