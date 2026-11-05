<!-- #genai: Python conceptual interview Q&A, compiled from chanduthedev Medium series + high-frequency additions, for Docusign R1 coding round -->

# Python Interview Questions — Conceptual (Part 1 of 2)

> Compiled from the chanduthedev Medium series (linked below) plus commonly-asked additions. Pairs with
> [05-python-programming-interview-questions.md](./05-python-programming-interview-questions.md) (coding
> problems). For a DevOps role they test **fundamentals + clear reasoning**, not deep CPython internals.
>
> Sources: [Topics to prepare](https://chanduthedev.medium.com/topics-to-prepare-for-python-developer-interviews-18f0dc82f29b) ·
> [Beginner](https://chanduthedev.medium.com/python-interview-questions-for-beginners-cd05ddda40af) ·
> [Intermediate](https://chanduthedev.medium.com/python-interview-questions-intermediate-level-c7ce66342e2d) ·
> [Advanced](https://chanduthedev.medium.com/python-interview-questions-advanced-2dabff728e54)

---

## A. Data types & basics

### 1. List vs Set vs Tuple vs Dict
| Type | Ordered | Mutable | Duplicates | Syntax | Use |
|---|---|---|---|---|---|
| **List** | yes | yes | yes | `[1, 2]` | ordered collection you modify |
| **Set** | no* | yes | **no** | `{1, 2}` | uniqueness, membership tests (O(1)) |
| **Tuple** | yes | **no** | yes | `(1, 2)` | fixed record; hashable → dict key |
| **Dict** | yes (3.7+) | yes | keys unique | `{"k": 1}` | key→value, O(1) lookup |

\*sets are unordered. Only **tuple** (and str/numbers) are immutable among these.

### 2. Mutable vs immutable objects
- **Immutable:** `str`, `tuple`, `int`/`float`, `frozenset`, `bytes` — can't change in place.
- **Mutable:** `list`, `set`, `dict`, most objects.
- **Why it matters:** immutables are safe to share/pass and are **hashable** (usable as dict keys / set members). Mutable default arguments are a classic bug (see #12).

### 3. Slicing — `s[start:stop:step]`
- `"chanduthedev"[1:8:2]` → start at 1, step 2, up to (not incl.) 8.
- **`s[::-1]`** reverses. **`s[-1]`** = last element; **`s[:-1]`** = everything *except* the last.

### 4. List/dict/set comprehensions
Concise way to build a collection: `[x*x for x in nums if x % 2 == 0]`. Dict: `{k: v for k, v in pairs}`.
Set: `{x for x in items}`. More readable and often faster than manual loops.

### 5. `enumerate()` and `zip()`
- **`enumerate(seq)`** → `(index, value)` pairs; prefer over `range(len(seq))`.
- **`zip(a, b)`** → pairs positional elements; stops at the shortest (use `itertools.zip_longest` to pad).

### 6. `is` vs `==`
- **`==`** compares **values**. **`is`** compares **identity** (same object in memory).
- Use `is` only for `None`/singletons: `if x is None`.

### 7. `*args` and `**kwargs`
- `*args` = variable positional args (a tuple). `**kwargs` = variable keyword args (a dict). Used for
  flexible signatures and to forward args (`func(*args, **kwargs)`).

---

## B. Functions & functional features

### 8. Lambda functions
Anonymous, single-expression functions: `square = lambda x: x*x`. Common as a `key=` in `sorted`/`max`.
Can't contain statements or multiple lines.

### 9. Generators & `yield`
A function using **`yield`** returns an **iterator** that produces values lazily, one at a time, **pausing
and resuming** while keeping state. Great for **infinite sequences** and **reading large files** without
loading everything into memory.

### 10. Function vs generator
- Function `return`s once with the full result; doesn't retain state.
- Generator `yield`s repeatedly, **saves state** between calls, is **memory-efficient**, and is iterated in a
  loop. Use a generator when you want items one-by-one / streaming; a function when you want the whole result
  at once.

### 11. Decorators
A **decorator** wraps a function to add behavior **without changing** its code — a form of the decorator
design pattern. Applied with `@decorator` above the function. Ubiquitous for logging, timing, auth,
caching, retries.
```python
def timer(func):
    def wrapper(*args, **kwargs):
        import time; t = time.time()
        r = func(*args, **kwargs)
        print(time.time() - t)
        return r
    return wrapper
```

### 12. Default arguments & the mutable-default pitfall
Default args are evaluated **once at definition time**. A mutable default (`def f(x, acc=[])`) is **shared
across calls** → surprising accumulation. Fix with the sentinel pattern:
```python
def f(x, acc=None):
    if acc is None:
        acc = []
    acc.append(x)
    return acc
```

### 13. Context managers (`with`)
Manage resources with guaranteed setup/cleanup via `__enter__`/`__exit__`. Removes boilerplate — e.g.
`with open("f") as fh:` auto-closes the file even on exceptions.

---

## C. OOP

### 14. Constructor
`__init__` initializes a new object (Python's constructor). `__new__` actually creates it (rarely
overridden).

### 15. class method vs instance (object) method vs static method
- **Instance method** — default; first arg `self`; reads/writes instance data.
- **Class method** — `@classmethod`; first arg `cls`; operates on the class (e.g. alternative constructors).
- **Static method** — `@staticmethod`; no `self`/`cls`; a plain function grouped in the class.

### 16. Abstract classes
Via the `abc` module + `@abstractmethod`. Defines an interface a subclass **must** implement; the abstract
class itself can't be instantiated.

### 17. Multiple inheritance & MRO
Python allows multiple inheritance. Conflicts resolved by **MRO (Method Resolution Order)** — subclasses
before parents, left-to-right (C3 linearization). Inspect with `ClassName.__mro__`.

### 18. Magic / dunder methods
Methods with `__name__` (double underscores). Customize behavior: `__init__`, `__str__`, `__repr__`,
`__len__`, `__eq__`, `__enter__/__exit__`, `__iter__/__next__`. `dir(obj)` lists them.

### 19. `__str__` vs `__repr__`
`__str__` = human-readable (for `print`); `__repr__` = unambiguous, for developers/debugging (ideally
recreates the object). If only one is defined, define `__repr__`.

### 20. `@property`
Turns a method into a managed attribute (getter/setter) — access like `obj.value` while keeping validation
logic. Good for computed or protected fields.

### 21. Singleton
A design pattern allowing **one instance** app-wide (logging, config, connection pool). Implement via a
class variable + a `get_instance()` that creates it once, or a metaclass/module-level object.

### 22. OOP principles (be ready to name)
**Encapsulation** (bundle data+behavior, hide internals), **Abstraction** (expose *what*, hide *how*),
**Inheritance** (reuse via subclassing), **Polymorphism** (same interface, different behavior).

---

## D. Advanced / internals

### 23. GIL (Global Interpreter Lock)
A mutex that lets **only one thread execute Python bytecode at a time**, making memory management
thread-safe. Consequence: threads don't give true CPU parallelism for CPU-bound work.

### 24. Multithreading vs multiprocessing
- **Threads** — share memory; good for **I/O-bound** work; limited by the GIL for CPU work (concurrency, not
  parallelism).
- **Processes** (`multiprocessing`) — separate memory, bypass the GIL → true parallelism for **CPU-bound**
  work; heavier to create and communicate.
- **asyncio** — single-threaded cooperative concurrency for high-I/O (`async`/`await`).

### 25. Memory management & garbage collection
CPython uses **reference counting** (object freed when refcount hits 0) plus a **cyclic garbage collector**
for reference cycles. Memory is managed in private heaps by the interpreter.

### 26. Shallow vs deep copy
- **Assignment (`=`)** just makes another reference to the *same* object.
- **Shallow copy** (`copy.copy`) — new outer object, but nested objects still **shared**.
- **Deep copy** (`copy.deepcopy`) — fully independent recursive copy.

### 27. Iterables vs iterators
- **Iterable** — has `__iter__` (list, str, dict) — can be looped.
- **Iterator** — has `__next__` (and `__iter__`) — produces values one at a time; exhausts. Generators are
  iterators.

### 28. Memoization
Caching results of expensive/repeated calls (esp. recursion) to save recomputation. Use a dict or
`functools.lru_cache`.

### 29. Exception handling
`try` / `except` / `else` (runs if no exception) / `finally` (always runs). Catch specific exceptions;
raise custom ones by subclassing `Exception`. **NameError** = undefined variable; **AttributeError** =
attribute/method missing on the object.

### 30. Scope & namespaces (LEGB)
Name lookup order: **Local → Enclosing → Global → Built-in**. `global` and `nonlocal` let you rebind names
in outer scopes.

### 31. `collections` module (very handy in coding rounds)
`Counter` (counts), `defaultdict` (default values), `namedtuple` (lightweight record), `deque` (fast
ends/queue), `OrderedDict`.

### 32. Type hints & f-strings
Type hints (`def f(x: int) -> str:`) aid readability/tooling (not enforced at runtime). f-strings
(`f"{name=}"`) are the modern formatting.

---

## E. Web / tooling (name-level for DevOps)

### 33. Flask / WSGI
Flask is a micro web framework. **WSGI** is the standard interface between Python web apps and servers
(gunicorn/uWSGI). Know: routing, **request context**, **extensions**, and **caching** (e.g. Flask-Caching).

### 34. Unit testing
`unittest` (stdlib, class-based) and `pytest` (concise, fixtures, popular). Know assertions, fixtures,
mocking (`unittest.mock`), and why tests matter in CI.

### 35. Design patterns to name
Singleton, Decorator, Factory, Observer, Strategy. For DevOps, **Singleton** and **Decorator** come up most.

### 36. Virtual environments & packaging
`venv`/`virtualenv` isolate dependencies; `pip` + `requirements.txt` (or `pyproject.toml`) manage them.
Relevant for reproducible builds in CI.

---

## Rapid-fire one-liners (memorize)
- **`is` vs `==`** → identity vs value.
- **Mutable default** → shared across calls; use `None` sentinel.
- **Generator** → lazy, stateful, memory-efficient iterator via `yield`.
- **Decorator** → wrap to add behavior without editing the function.
- **GIL** → one thread runs Python bytecode at a time; use multiprocessing for CPU-bound.
- **Deepcopy** → fully independent; **shallow** → nested still shared.
- **`__str__`** readable vs **`__repr__`** unambiguous.
