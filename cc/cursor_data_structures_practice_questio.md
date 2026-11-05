# Data structures practice questions
_Exported on 25/05/2026 at 20:04:59 GMT+5:30 from Cursor (2.5.25)_

---

**User**

Hi. So I, four years back, I used to be really good at data structures and algorithm because I spent two years of my life doing that and I had a hundred day streak at LeetCode, so I was really good at it somehow I didn't get a very good organization like Phuong or anything, but now I want to get back at data structures. I tried solving questions, but I'm unable to. I am not good anymore, and I have I'm thinking of starting in Python. So I would like to solve around small list of set of questions which have all the patterns but not in very advanced or hard level some within similar, simpler level and you know, something after solving, I can get some confidence, but I also learn about different, different patterns which I have, I have forgotten now. So I want you to Find me if there's a list online which Is having all this type of questions which is not extremely advanced, but the small set of questions and also covers almost everything if there is a list, well and good, if not can you make one list? Well, that'd be possible if yes, then let's go ahead.

---

**Cursor**

Good news — yes, several well-known curated lists exist online for exactly this purpose, and I can also tailor a smaller, confidence-building one for you. Let me orient you quickly before asking 1–2 questions to nail the scope.

## Existing curated lists worth knowing

- **Blind 75** (by Yangshun, ex-Meta) — 75 problems covering all core patterns. Mostly Medium, a few Easy/Hard. Great breadth but can feel heavy when rusty.
- **Grind 75** (same author, more flexible) — lets you pick number of problems and weeks; ordered by importance. Best for ramping up.
- **NeetCode 150 / NeetCode Roadmap** (neetcode.io) — visual pattern roadmap (Arrays → Two Pointers → Sliding Window → … → DP). Has free video walkthroughs in Python. Excellent for relearning.
- **Sean Prashad's LeetCode Patterns** — pattern-tagged list, can filter Easy.
- **Striver's SDE Sheet / A2Z Sheet** — popular in India, very comprehensive but larger.

Given you want a **small set, not advanced, covers all patterns, rebuild confidence**, my recommendation is to build you a **custom 25–35 problem list** based on the NeetCode pattern roadmap, biased toward Easy and the gentlest Mediums, in Python — rather than handing you Blind 75 directly.

The patterns I'd cover (one or two problems each):
Arrays/Hashing, Two Pointers, Sliding Window, Stack, Binary Search, Linked List, Trees (BFS/DFS), Tries, Heap, Backtracking, Graphs (BFS/DFS), 1-D DP, Greedy, Intervals, Bit Manipulation, Math.

Before I draft the plan, two quick questions:

I have everything I need. Here's the plan I'm proposing.

---

**User**

Implement the plan as specified, it is attached for your reference. Do NOT edit the plan file itself.

To-do's from the plan have already been created. Do not create them again. Mark them as in_progress as you work, starting with the first one. Don't stop until you have completed all the to-dos.

---

**Cursor**

Now let me verify the file: check the header, count problems, and confirm all 15 patterns are present.

All checks pass:

- Line 1 is `# genai` (matching the convention used in `16-idp-fundamentals.md`, `23-mareana-cheat-sheet.md`, etc.)
- Exactly **30** problem rows with LeetCode links and difficulty (Easy/Medium only — no Hards)
- All **15 patterns** present with correct counts: Arrays&Hashing(4), Two Pointers(3), Sliding Window(2), Stack(2), Binary Search(2), Linked List(3), Trees(3), Tries(1), Heap(1), Backtracking(2), Graphs(2), 1-D DP(2), Greedy(1), Intervals(1), Bit Manipulation(1) = 30
- 4-week schedule with day-by-day breakdown
- Pattern templates appendix with Python skeletons for sliding window, binary search, linked list reversal, tree BFS, grid DFS, backtracking, heap top-k, and 1-D DP

Done. Your curated DSA relearn list is ready at [`Learn/24-dsa-relearn-python.md`](/Users/anshag-blrm24/Desktop/Learn/24-dsa-relearn-python.md).

## What's inside

- **30 problems** in NeetCode-roadmap order, all linked to LeetCode, ~22 Easy + ~8 gentle Medium, no Hards.
- **15 patterns covered** — Arrays & Hashing, Two Pointers, Sliding Window, Stack, Binary Search, Linked List, Trees, Tries, Heap, Backtracking, Graphs, 1-D DP, Greedy, Intervals, Bit Manipulation.
- **One-line hint per problem** so you can peek if stuck without spoiling the solution.
- **How to use** rules baked in: 30-min attempt rule, 3-day & 10-day spaced redo, Python idioms to keep loaded.
- **4-week day-by-day schedule** (7–8 problems/week, Easy-heavy in week 1).
- **Pattern templates appendix** — small Python skeletons for sliding window, binary search, linked-list reversal, tree BFS, grid DFS, backtracking, top-k heap, and 1-D DP rolling variables.
- **"After this list" section** pointing at next steps (NeetCode 150 → Blind 75 mediums → harder patterns) so the path beyond the 30 is clear.

Suggested first move: open the file, do problem #1 (Contains Duplicate) and #2 (Valid Anagram) today. The first two are deliberately the easiest in the list — pure confidence wins.

---

**User**

Add two more row to that table People kind of thing that you have created in all the topics. one is revision three day or just three day or just 3D or s and the second will be 10 day or 10D s-something So that I can track I can comment, feel that, okay, I came and I did revise those after few days.

---

**Cursor**

Got it — you want two extra columns in every problem table for tracking your spaced-redo (3-day and 10-day revisions). Let me add `3D` and `10D` columns to all 15 tables and update the "How to use" section to explain them.

Now update all 15 table headers and separators in one shot using replace_all (they're identical across tables).

Let me check which table didn't match — there should be 15.

Looks like you manually edited the first Arrays & Hashing table to add a `Revise` column already. Let me check that table to align with your intent.

You manually set up a single `Revise` column there, but your request is for two separate columns (`3D` and `10D`). Let me align that first table with the rest, preserving the `Y` you marked on Contains Duplicate by moving it into the `3D` column.

Now I'll append the empty `3D` and `10D` cells to all 26 remaining rows in parallel.

Now let me verify all 30 rows have exactly 6 pipe-separated columns and all 15 headers are consistent.

Let me also verify the `Y` mark on row 1 is preserved and row alignment looks right by sampling.

Done. Tracking columns are now in every problem table.

## What changed in [`Learn/24-dsa-relearn-python.md`](/Users/anshag-blrm24/Desktop/Learn/24-dsa-relearn-python.md)

- **All 15 problem tables** now have two extra columns: `3D` and `10D` — for your 3-day and 10-day spaced revisions.
- **All 30 rows** got empty tracking cells you can fill with `Y`, `x`, or a date (e.g. `Jun 12`).
- **The `Y` you put on Contains Duplicate is preserved** — moved into its `3D` column.
- **`How to use` rule #4 was updated** to explain how to use these columns:
  > Use the **`3D`** and **`10D`** columns in each table to mark your revisions: drop an `x` or a date so you can see at a glance what you've revisited and what's still pending.

Verification confirmed: 15 headers in the new format, 30 rows with 6 columns each.

So your workflow now is: solve a problem → leave the row alone if you nailed it, or come back after 3 days, redo it, and put `x` (or the date) under `3D`. Same again at 10 days under `10D`. One glance at the file tells you exactly which problems still need a redo pass.

---

**User**

ok
don't take any action 
just need suggestions and input
I am sharing a code I just wrote and I know a few issues in this
which are not going away and it's been months

class Solution:
    def isPalindrome(self, s: str) -> bool:
        i=0
        j=s.len()-1
        while(i<j):
            if (isAlphanum(s[i]) == False):
                i+=1
                continue
            if (isAlphanum(s[j]) == False):
                j-=1
                continue
            if lower(s[i]) == lower(s[j]):
                i+=1
                j-=1
                continue
            return False
        
        return True
            

---

**Cursor**

Good news first: **your algorithm is 100% correct**. Two-pointer skip-non-alnum, lowercase compare, return False on mismatch — that's the textbook approach. The bugs are all the same flavor: you're writing Python like it's Java/C++.

## The 3 actual bugs (and they're the same root cause)

| Line | What you wrote | What Python expects | Why |
|---|---|---|---|
| `j = s.len() - 1` | `s.len()` | `len(s)` | `len` is a **built-in function** in Python, not a method on objects. So you call `len(s)`, not `s.len()`. |
| `isAlphanum(s[i])` | `isAlphanum(...)` as a free function | `s[i].isalnum()` | This is a **method on the string**, not a free function. Also: spelled `isalnum` (all lowercase, no capital A/N). |
| `lower(s[i])` | `lower(...)` as a free function | `s[i].lower()` | Same again — `.lower()` is a method on strings. |

See the pattern? **All three bugs are you reaching for `function(thing)` when Python wants `thing.method()`.** Java/C# muscle memory. Once you internalise this, all three go away forever.

The rule of thumb in Python:

- **Built-ins like `len`, `sum`, `max`, `min`, `abs`, `sorted`, `reversed`** → free functions: `len(s)`, `max(arr)`.
- **Anything that "asks the object about itself" — `.lower()`, `.upper()`, `.isalnum()`, `.isdigit()`, `.strip()`, `.split()`, `.startswith()`** → methods: `s.lower()`.

(There's no perfect rule for *which* is which, but `len`/`sum`/`max` being free functions is mostly historical. Memorise that small set; everything else on strings is a method.)

## The Pythonic version

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        i, j = 0, len(s) - 1
        while i < j:
            if not s[i].isalnum():
                i += 1
                continue
            if not s[j].isalnum():
                j -= 1
                continue
            if s[i].lower() != s[j].lower():
                return False
            i += 1
            j -= 1
        return True
```

Style nits I changed (none of these were bugs, just Pythonic polish):

1. **`if not x:`** instead of `if x == False:`. Never compare booleans with `==`. PEP 8 explicitly says "don't compare boolean values to True or False using `==`."
2. **No parentheses** around `while i < j` or `if not ...`. Python isn't C — those parens are noise.
3. **Flipped the final compare to `!=` and `return False` early**, so the success path falls through naturally. Removes one `continue`.
4. **`i, j = 0, len(s) - 1`** — Python's tuple unpacking is one of its best features; use it for paired init.
5. **Spaces around `=`** (`i = 0`, not `i=0`). PEP 8 again.

## One-liner version (for fun, not for interviews)

Once you trust the pattern:

```python
class Solution:
    def isPalindrome(self, s: str) -> bool:
        cleaned = [c.lower() for c in s if c.isalnum()]
        return cleaned == cleaned[::-1]
```

`cleaned[::-1]` is the Python idiom for "reversed list/string" — slice with step `-1`. Worth memorising. But in an interview, **stick with your two-pointer version** — it's O(1) extra space, the one-liner is O(n) extra space.

## The meta-lesson for the next 29 problems

The fact that the same conceptual error showed up three times in 12 lines tells me what to watch for as you go through the list:

- Every time you reach for a "function," ask: *is this Python's job or the object's job?*
- Keep this cheat-list on a sticky note while doing the first 5–6 problems:

```
len(x)        sum(x)         max(x)        min(x)         abs(x)
sorted(x)     reversed(x)    enumerate(x)  zip(a, b)      range(n)

x.lower()     x.upper()      x.isalnum()   x.isdigit()    x.strip()
x.split()     x.join(items)  x.startswith() x.endswith()  x.replace(a,b)
x.append(v)   x.pop()        x.sort()      d.get(k)       d.items()
```

Top row: free functions. Bottom rows: methods. After ~10 problems this becomes automatic and you'll never write `s.len()` again.

Now go fill in the `3D` column for problem #5 — you've earned it. Move on to #6 (Move Zeroes).

---

**User**

Can you Add this sticky note on top of this twenty-four DSL on file.

---

**Cursor**

Done. The Python sticky note is now pinned at the top of [`Learn/24-dsa-relearn-python.md`](/Users/anshag-blrm24/Desktop/Learn/24-dsa-relearn-python.md), placed between the intro blurb and the "How to use" section so it's the first reference card you see when you open the file.

## What was added

A new section **"Python sticky note — function vs method"** with:

- **Free functions** block — `len`, `sum`, `max`, `min`, `abs`, `sorted`, `reversed`, `enumerate`, `zip`, `range`, `any`, `all`, `map`, `filter`.
- **Methods** block, grouped by data type:
  - **Strings** — `lower`, `upper`, `isalnum`, `strip`, `split`, `replace`, `startswith`, `join`, etc.
  - **Lists** — `append`, `pop`, `insert`, `extend`, `sort`, `reverse`, `index`, `remove`.
  - **Dicts** — `get`, `keys`, `values`, `items`, `pop`, `setdefault`, `update`.
  - **Sets** — `add`, `remove`, `discard`, `union`, `intersection`.
- **Two boolean traps** — never `== False`, and what counts as falsy in Python.
- **Reverse idiom** — `x[::-1]`, the one-liner that comes up constantly.

I also slightly expanded it beyond what I gave you in the chat — added `any`, `all`, `map`, `filter` (you'll hit these on the Sliding Window and Two Pointers problems), and the dict/set methods (you'll need them for Contains Duplicate, Two Sum, Group Anagrams, Implement Trie).

Open the file once before each session for the first week — you'll stop reaching for `s.len()` within 3–4 problems.

---

**User**

ok 
again one suggestion what to do when I don't remember about a topic?

---

**Cursor**

Great question — this is the single biggest difference between "grinding LeetCode" and "actually relearning." Here's the rule that works:

## The one suggestion: triage first, then choose your fix

Don't apply the 30-minute struggle rule to a topic you've **forgotten**. The 30-min rule is for problems where you *know* the pattern but are stuck on a specific twist. When you don't remember the pattern at all, struggling for 30 minutes is just frustration. Instead, **diagnose what kind of "forgot" it is in 60 seconds**, then pick the right depth of refresher:

| What you forgot | Right move | Time budget |
|---|---|---|
| **Python syntax** (e.g. how to make a heap) | Glance at the sticky note + 1 quick Google ("python heap min") | 2 min |
| **The data structure itself** (e.g. what a Trie even is) | Watch a 5-min explainer (NeetCode / Abdul Bari on YouTube) | 5–10 min |
| **The pattern** (e.g. you forgot what "sliding window" means) | Read the template in the **Pattern templates appendix** of your file + 1 NeetCode pattern video | 10–15 min |
| **The specific problem** (you remember the pattern but are stuck) | This is where the 30-min struggle rule applies | 30 min |

## The flow when you've forgotten a topic

```
1. Open NeetCode video for the pattern (NOT for the specific problem) → 5–10 min
2. Read the matching template from your file's appendix → 2 min
3. CLOSE everything. Phone face-down. No tabs.
4. Re-implement from scratch in your editor, no peeking.
5. If you get stuck for 15 min, peek at ONE thing (the hint in your table).
6. Mark in the 3D column. Redo in 3 days from cold memory.
```

## The anti-pattern to avoid (this is what kills relearning)

Reading the solution and thinking *"ah yes, I remember now."* That feeling is **recognition**, not **recall** — and it's a lie your brain tells you. You'll be just as stuck tomorrow.

The fix: **always close the editorial and re-implement from scratch the same day.** If you can't, you didn't learn it; you just read it.

## Concrete go-to resources

- **NeetCode YouTube** ([neetcode.io](https://neetcode.io)) — every problem in your 30 has a free 8–12 min video, *and* he explains the pattern before the code. This is the single best resource for your situation.
- **visualgo.net** — animated visualisations for trees, heaps, graphs, sorting. Use when you've forgotten a *data structure*.
- **Python REPL in your terminal** — for "I forgot the syntax" moments. Type `dir(list)` to see all list methods, or `help(heapq)`. Faster than Google for stdlib.
- **Your own file's "Pattern templates" appendix** (lines ~250-340 of `24-dsa-relearn-python.md`). Treat it as a memory anchor — re-read the template *before* attempting any problem in a new pattern.

## Bonus rule: the 5-minute pre-read

For each new pattern (you'll hit a new one every 2–3 problems), spend **5 minutes reading the template + watching the NeetCode pattern intro** *before* you even open the problem. This isn't cheating — it's loading the right context into working memory. The pattern is the part you want to *learn*; the problem is just the gym equipment for practising it.

---

Want me to add a condensed version of this ("When you've forgotten a topic — what to do") as a section in the file, near the top alongside the Python sticky note? It's exactly the kind of thing you'll want to re-read every time you hit a new pattern. Just say the word.

---

**User**

Hi
Can we create a folder in anshul-dir folder called plans
and create context about the chat and actions we do here
so that in case I lose the chat window or has to reinstall cursor we still can access them again without going through learning curve again!!!

