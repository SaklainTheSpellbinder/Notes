# Python + NumPy + Matplotlib: Problem-Solving Master Guide

> Goal: get to the point where Python syntax, containers, NumPy array manipulation, and plotting are **not** the bottleneck when solving competitive-programming, LeetCode, numerical/DSP, signal-processing, or lab problems.

This is written as a **working reference + learning guide**. Read it once in order, then practice the exercises at the end. After that, use the headings as a lookup sheet.

---

# Table of Contents

1. [The mental model: Python vs C++/Java vs NumPy](#1-the-mental-model-python-vs-cjava-vs-numpy)
2. [Python syntax you should be able to write without thinking](#2-python-syntax-you-should-be-able-to-write-without-thinking)
3. [Core data structures: list, tuple, set, dict](#3-core-data-structures-list-tuple-set-dict)
4. [Mutability, references, shallow copy, deep copy](#4-mutability-references-shallow-copy-deep-copy)
5. [Indexing and slicing: `a[start:stop:step]`](#5-indexing-and-slicing-astartstopstep)
6. [Looping through anything](#6-looping-through-anything)
7. [Sorting anything](#7-sorting-anything)
8. [Strings](#8-strings)
9. [Functions, lambdas, unpacking, scope](#9-functions-lambdas-unpacking-scope)
10. [Useful standard-library tools for CP/LeetCode](#10-useful-standard-library-tools-for-cpleetcode)
11. [Input/output patterns for contests](#11-inputoutput-patterns-for-contests)
12. [Python OOP](#12-python-oop)
13. [NumPy fundamentals](#13-numpy-fundamentals)
14. [NumPy indexing: 1D, 2D, 3D](#14-numpy-indexing-1d-2d-3d)
15. [Views vs copies in NumPy](#15-views-vs-copies-in-numpy)
16. [Shape manipulation](#16-shape-manipulation)
17. [The `axis` parameter](#17-the-axis-parameter)
18. [Broadcasting](#18-broadcasting)
19. [Boolean masks and conditional selection](#19-boolean-masks-and-conditional-selection)
20. [Sorting, searching, ranking in NumPy](#20-sorting-searching-ranking-in-numpy)
21. [Combining and splitting arrays](#21-combining-and-splitting-arrays)
22. [Math, statistics, complex numbers](#22-math-statistics-complex-numbers)
23. [Interpolation](#23-interpolation)
24. [Numerical integration](#24-numerical-integration)
25. [Numerical derivatives and gradients](#25-numerical-derivatives-and-gradients)
26. [Meshgrids and multidimensional coordinates](#26-meshgrids-and-multidimensional-coordinates)
27. [Linear algebra](#27-linear-algebra)
28. [FFT and DSP essentials](#28-fft-and-dsp-essentials)
29. [Convolution and correlation](#29-convolution-and-correlation)
30. [Common NumPy operations cheat sheet](#30-common-numpy-operations-cheat-sheet)
31. [Matplotlib fundamentals](#31-matplotlib-fundamentals)
32. [DSP plotting patterns](#32-dsp-plotting-patterns)
33. [Common mistakes and debugging](#33-common-mistakes-and-debugging)
34. [Python↔NumPy conversion guide](#34-pythonnumpy-conversion-guide)
35. [Patterns you should memorize](#35-patterns-you-should-memorize)
36. [Exercises](#36-exercises)
37. [Suggested practice order](#37-suggested-practice-order)

---

# 1. The mental model: Python vs C++/Java vs NumPy

Think of the three environments like this:

| Task | C++ | Java | Python |
|---|---|---|---|
| Dynamic array | `vector<int>` | `ArrayList<Integer>` | `list` |
| Fixed tuple-like record | `pair`, `tuple` | record/class | `tuple` |
| Hash map | `unordered_map` | `HashMap` | `dict` |
| Ordered map | `map` | `TreeMap` | usually `dict` + `sorted(...)` |
| Hash set | `unordered_set` | `HashSet` | `set` |
| Queue/deque | `deque` | `ArrayDeque` | `collections.deque` |
| Heap | `priority_queue` | `PriorityQueue` | `heapq` |
| Sort | `sort()` | `Collections.sort()` | `list.sort()` / `sorted()` |
| Numerical array | vector/manual | arrays/manual | `numpy.ndarray` |

The most important distinction:

```python
a = [1, 2, 3]          # Python list: general-purpose container
b = np.array([1, 2, 3]) # NumPy array: numeric tensor
```

Python lists:
- can contain mixed types;
- grow/shrink easily;
- are ideal for CP data structures;
- `+` concatenates;
- `*` repeats.

NumPy arrays:
- normally contain one dtype;
- have a fixed rectangular shape;
- support vectorized mathematical operations;
- `+`, `*`, `/`, comparisons are **elementwise**;
- have `shape`, `ndim`, `axis`, broadcasting, masks, FFT, linear algebra, etc.

Example:

```python
[1, 2, 3] * 2
# [1, 2, 3, 1, 2, 3]

np.array([1, 2, 3]) * 2
# array([2, 4, 6])
```

For LeetCode/CF: mostly use Python containers.

For DSP/numerical labs: once your data is numeric and rectangular, strongly prefer NumPy.

---

# 2. Python syntax you should be able to write without thinking

## Variables

```python
x = 10
name = "DSP"
pi = 3.14159
ok = True
nothing = None
```

Python is dynamically typed:

```python
x = 5
x = "hello"  # legal
```

## Multiple assignment

```python
a, b = 10, 20
a, b = b, a       # swap
```

## Comparisons

```python
x == y
x != y
x < y
x <= y
x > y
x >= y
```

Chained comparison:

```python
if 0 <= x < 10:
    ...
```

Equivalent to:

```python
if 0 <= x and x < 10:
    ...
```

## Boolean operators

```python
and
or
not
```

Do **not** write:

```python
if x == 1 or 2:   # WRONG
```

Write:

```python
if x == 1 or x == 2:
    ...
```

or:

```python
if x in (1, 2):
    ...
```

## `if / elif / else`

```python
if score >= 80:
    grade = "A"
elif score >= 70:
    grade = "B"
else:
    grade = "C"
```

Conditional expression:

```python
parity = "even" if x % 2 == 0 else "odd"
```

## `for`

```python
for i in range(5):
    print(i)
# 0 1 2 3 4
```

```python
for i in range(2, 8):
    ...
# 2,3,4,5,6,7
```

```python
for i in range(10, 0, -2):
    ...
# 10,8,6,4,2
```

## `while`

```python
while x > 0:
    x -= 1
```

## `break`, `continue`

```python
for x in a:
    if x < 0:
        continue
    if x == target:
        break
```

## `for ... else`

The `else` runs only if the loop was **not broken**:

```python
for x in nums:
    if x == target:
        print("found")
        break
else:
    print("not found")
```

---

# 3. Core data structures: list, tuple, set, dict

# 3.1 List

Closest idea: C++ `vector`, Java `ArrayList`.

```python
a = [10, 20, 30]
```

Properties:
- ordered;
- indexed;
- mutable;
- duplicates allowed.

## Access

```python
a[0]      # first
a[-1]     # last
a[-2]     # second-last
```

## Change

```python
a[1] = 999
```

## Append / remove

```python
a.append(40)
a.extend([50, 60])
a.insert(1, 15)

x = a.pop()       # remove and return last
x = a.pop(2)      # remove and return index 2

a.remove(999)     # remove first matching value
a.clear()
```

## Search

```python
if 30 in a:
    ...

idx = a.index(30)   # first occurrence; ValueError if absent
count = a.count(30)
```

## Length

```python
len(a)
```

## Concatenate

```python
c = a + b
```

## Repeat

```python
zeros = [0] * 5
# [0,0,0,0,0]
```

## List comprehension

```python
squares = [x*x for x in range(10)]
evens = [x for x in nums if x % 2 == 0]
```

Transformation + condition:

```python
result = [x*x for x in nums if x > 0]
```

Nested:

```python
matrix = [[i*j for j in range(4)] for i in range(3)]
```

### Very important 2D-list trap

WRONG:

```python
a = [[0] * 3] * 4
a[0][0] = 9
```

All rows refer to the same inner list.

Correct:

```python
a = [[0] * 3 for _ in range(4)]
```

---

# 3.2 Tuple

```python
p = (10, 20)
```

Properties:
- ordered;
- indexed;
- **immutable**;
- duplicates allowed;
- can be dictionary keys if all elements are hashable.

```python
x = p[0]
```

Cannot do:

```python
p[0] = 99   # TypeError
```

## Tuple unpacking

```python
x, y = p
```

```python
name, age, score = ("Alice", 20, 91)
```

## One-element tuple

```python
x = (5,)   # tuple
y = (5)    # int
```

## Why tuples matter in CP

Coordinates:

```python
pos = (r, c)
visited.add(pos)
```

Dictionary keys:

```python
dist[(r, c)] = 10
```

Sorting records:

```python
arr = [(3, "c"), (1, "a"), (2, "b")]
arr.sort()
# sorted lexicographically by tuple elements
```

---

# 3.3 Set

Closest idea: C++ `unordered_set`, Java `HashSet`.

```python
s = {1, 2, 3}
```

Empty set:

```python
s = set()
```

Not:

```python
s = {}   # this is an empty dict
```

## Add/remove

```python
s.add(4)
s.remove(2)       # KeyError if absent
s.discard(2)      # safe if absent
x = s.pop()       # arbitrary element
```

## Membership

```python
if x in s:
    ...
```

Average O(1).

## Set operations

```python
A | B      # union
A & B      # intersection
A - B      # difference
A ^ B      # symmetric difference
A <= B     # subset
A < B      # proper subset
```

Equivalent methods:

```python
A.union(B)
A.intersection(B)
A.difference(B)
```

## Remove duplicates

```python
unique = set(nums)
```

If you need sorted unique values:

```python
unique_sorted = sorted(set(nums))
```

If you need uniqueness while preserving insertion order:

```python
unique_ordered = list(dict.fromkeys(nums))
```

---

# 3.4 Dictionary

Closest idea: C++ `unordered_map`, Java `HashMap`.

```python
d = {
    "alice": 90,
    "bob": 82
}
```

## Access

```python
d["alice"]
```

If missing, this raises `KeyError`.

Safe:

```python
score = d.get("charlie")       # None
score = d.get("charlie", 0)    # 0
```

## Insert/change

```python
d["charlie"] = 88
d["alice"] = 95
```

## Delete

```python
del d["bob"]
x = d.pop("alice")
```

Safe pop:

```python
x = d.pop("missing", None)
```

## Membership

```python
if "alice" in d:
    ...
```

Checks **keys**, not values.

## Keys, values, items

```python
d.keys()
d.values()
d.items()
```

Convert to list:

```python
keys = list(d.keys())
values = list(d.values())
pairs = list(d.items())
```

Example:

```python
d = {"b": 2, "a": 5, "c": 1}

list(d.items())
# [('b', 2), ('a', 5), ('c', 1)]
```

## Loop dictionary

Keys:

```python
for k in d:
    print(k)
```

Keys + values:

```python
for k, v in d.items():
    print(k, v)
```

Values:

```python
for v in d.values():
    print(v)
```

## Get list associated with a key

```python
groups = {
    "red": [1, 4, 7],
    "blue": [2, 3]
}

red_items = groups["red"]
```

Safer:

```python
red_items = groups.get("red", [])
```

## Build dictionary of lists

Manual:

```python
groups = {}

for word in words:
    first = word[0]
    if first not in groups:
        groups[first] = []
    groups[first].append(word)
```

Better:

```python
from collections import defaultdict

groups = defaultdict(list)

for word in words:
    groups[word[0]].append(word)
```

## Frequency map

Manual:

```python
freq = {}

for x in nums:
    freq[x] = freq.get(x, 0) + 1
```

Better:

```python
from collections import Counter

freq = Counter(nums)
```

## Dictionary comprehension

```python
square = {x: x*x for x in range(5)}
```

Filter:

```python
big = {k: v for k, v in d.items() if v >= 50}
```

---

# 3.5 Container comparison

| Type | Ordered | Mutable | Duplicates | Indexed | Hashable itself |
|---|---:|---:|---:|---:|---:|
| `list` | yes | yes | yes | yes | no |
| `tuple` | yes | no | yes | yes | yes, if contents hashable |
| `set` | conceptually unordered | yes | no | no | no |
| `frozenset` | unordered | no | no | no | yes |
| `dict` | insertion ordered | yes | keys unique | by key | no |
| `str` | yes | no | yes | yes | yes |

---

# 4. Mutability, references, shallow copy, deep copy

This is one of the most important Python topics.

## Assignment does not copy

```python
a = [1, 2, 3]
b = a

b[0] = 99

print(a)
# [99, 2, 3]
```

`a` and `b` refer to the same list.

Check identity:

```python
a is b
# True
```

## Shallow copy

```python
b = a.copy()
```

or:

```python
b = list(a)
```

or:

```python
b = a[:]
```

For a flat list this is usually enough.

## Nested lists

```python
a = [[1, 2], [3, 4]]
b = a.copy()

b[0][0] = 99

print(a)
# [[99, 2], [3, 4]]
```

Why? Outer list was copied, inner lists were shared.

## Deep copy

```python
import copy

b = copy.deepcopy(a)
```

Now nested objects are copied recursively.

## Immutable objects

Ints, floats, strings, tuples are immutable.

```python
x = 5
y = x
y += 1
```

This does not mutate `x`; `y` is rebound to a new integer.

## `==` vs `is`

```python
a == b   # same value?
a is b   # same object?
```

Usually use `==`.

Use `is` mainly for singleton checks:

```python
if x is None:
    ...
```

---

# 5. Indexing and slicing: `a[start:stop:step]`

This syntax is:

```python
a[start : stop : step]
```

Important:
- `start` included;
- `stop` excluded;
- `step` controls jump;
- omitted values use defaults.

Suppose:

```python
a = [0, 1, 2, 3, 4, 5, 6, 7]
```

## Basic slices

```python
a[2:5]
# [2, 3, 4]
```

```python
a[:4]
# [0, 1, 2, 3]
```

```python
a[4:]
# [4, 5, 6, 7]
```

```python
a[:]
# full shallow copy
```

## Step

```python
a[::2]
# [0, 2, 4, 6]
```

```python
a[1::2]
# [1, 3, 5, 7]
```

## Reverse

```python
a[::-1]
# [7,6,5,4,3,2,1,0]
```

Every second element backwards:

```python
a[::-2]
```

## Negative indices

```python
a[-1]      # last
a[-3:]     # last 3
a[:-1]     # everything except last
```

## Replace slice

Lists allow slice assignment:

```python
a[2:5] = [100, 200]
```

The replacement does not need the same length.

Reverse list in place:

```python
a.reverse()
```

or:

```python
a[:] = a[::-1]
```

## Delete slice

```python
del a[2:5]
```

---

# 6. Looping through anything

## Values

```python
for x in nums:
    print(x)
```

## Index

```python
for i in range(len(nums)):
    print(i, nums[i])
```

Usually prefer `enumerate`.

## Index + value

```python
for i, x in enumerate(nums):
    print(i, x)
```

Start numbering at 1:

```python
for i, x in enumerate(nums, start=1):
    ...
```

## Parallel arrays

```python
names = ["a", "b", "c"]
scores = [90, 80, 70]

for name, score in zip(names, scores):
    print(name, score)
```

Three arrays:

```python
for a, b, c in zip(A, B, C):
    ...
```

## Reverse loop

```python
for x in reversed(nums):
    ...
```

Indices backwards:

```python
for i in range(len(nums) - 1, -1, -1):
    ...
```

## Dictionary

```python
for key in d:
    ...
```

```python
for key, value in d.items():
    ...
```

## Set

```python
for x in s:
    ...
```

Do not assume useful order.

## 2D list

```python
for row in matrix:
    for x in row:
        print(x)
```

With indices:

```python
for r, row in enumerate(matrix):
    for c, x in enumerate(row):
        print(r, c, x)
```

## Flatten nested Python list

```python
flat = [x for row in matrix for x in row]
```

---

# 7. Sorting anything

This is a major section. Memorize it.

# 7.1 `sorted()` vs `.sort()`

`sorted(iterable)`:
- returns a new list;
- works on lists, tuples, sets, dict keys, generators, etc.;
- does not modify original.

```python
b = sorted(a)
```

`list.sort()`:
- modifies the list;
- returns `None`.

```python
a.sort()
```

Do not write:

```python
b = a.sort()
# b is None
```

## Descending

```python
sorted(a, reverse=True)
```

or:

```python
a.sort(reverse=True)
```

## Sort strings

```python
names.sort()
```

Case-insensitive:

```python
names.sort(key=str.lower)
```

## Sort tuples

```python
pairs = [(2, 5), (1, 9), (2, 3)]
pairs.sort()
```

Result:

```text
(1,9), (2,3), (2,5)
```

Python sorts tuples lexicographically:
1. first element;
2. ties by second;
3. then third; etc.

---

# 7.2 Custom sort with `key=`

Sort by second element:

```python
pairs.sort(key=lambda p: p[1])
```

Descending by second:

```python
pairs.sort(key=lambda p: p[1], reverse=True)
```

Sort by second ascending, then first ascending:

```python
pairs.sort(key=lambda p: (p[1], p[0]))
```

Sort first ascending, second descending:

```python
pairs.sort(key=lambda p: (p[0], -p[1]))
```

Example:

```python
students = [
    ("Alice", 85, 20),
    ("Bob", 90, 22),
    ("Cara", 90, 19)
]

students.sort(key=lambda x: (-x[1], x[2]))
```

This means:
- score descending;
- age ascending.

## Sort dictionary entries by value

```python
d = {"a": 5, "b": 2, "c": 9}

items = sorted(d.items(), key=lambda kv: kv[1])
# [('b',2), ('a',5), ('c',9)]
```

Descending:

```python
items = sorted(d.items(), key=lambda kv: kv[1], reverse=True)
```

Extract sorted keys:

```python
keys = [k for k, v in sorted(d.items(), key=lambda kv: kv[1])]
```

Extract sorted values:

```python
values = [v for k, v in sorted(d.items(), key=lambda kv: kv[1])]
```

Create a new dictionary in sorted insertion order:

```python
sorted_d = dict(sorted(d.items(), key=lambda kv: kv[1]))
```

Important: this is still a normal dictionary. It preserves insertion order, but it is not a tree map that automatically stays sorted after future updates.

## Sort list of dictionaries

```python
people = [
    {"name": "A", "age": 20},
    {"name": "B", "age": 18},
]

people.sort(key=lambda p: p["age"])
```

## Sort objects

```python
students.sort(key=lambda s: s.score)
```

---

# 7.3 Custom comparator with `cmp_to_key`

Python normally uses `key`, not comparator functions.

If you truly need pairwise comparison:

```python
from functools import cmp_to_key

def cmp(a, b):
    if a + b > b + a:
        return -1
    if a + b < b + a:
        return 1
    return 0

nums = ["3", "30", "34", "5", "9"]
nums.sort(key=cmp_to_key(cmp))
```

This is useful for problems such as "largest concatenated number".

Prefer `key=` whenever possible.

---

# 7.4 Reverse

New reversed list:

```python
b = a[::-1]
```

Iterator:

```python
for x in reversed(a):
    ...
```

In-place:

```python
a.reverse()
```

---

# 8. Strings

Strings are immutable sequences of characters.

```python
s = "hello"
```

## Index/slice

```python
s[0]
s[-1]
s[1:4]
s[::-1]
```

Cannot do:

```python
s[0] = "H"
```

Instead:

```python
s = "H" + s[1:]
```

## Length

```python
len(s)
```

## Common methods

```python
s.lower()
s.upper()
s.capitalize()
s.title()

s.strip()
s.lstrip()
s.rstrip()

s.startswith("abc")
s.endswith(".txt")

s.find("cat")       # index or -1
s.index("cat")      # index or error

s.count("a")
s.replace("old", "new")
```

## Split

```python
line = "10 20 30"
parts = line.split()
# ['10','20','30']
```

Custom delimiter:

```python
"a,b,c".split(",")
```

## Join

```python
words = ["I", "love", "DSP"]
sentence = " ".join(words)
```

```python
",".join(words)
```

## Convert

```python
x = int("123")
y = float("3.14")
s = str(42)
```

## Character checks

```python
c.isdigit()
c.isalpha()
c.isalnum()
c.islower()
c.isupper()
c.isspace()
```

## `ord` and `chr`

```python
ord('a')   # 97
chr(97)    # 'a'
```

Alphabet index:

```python
idx = ord(c) - ord('a')
```

## Efficient string construction

Do not repeatedly concatenate huge strings in a loop if avoidable.

Better:

```python
parts = []
for x in nums:
    parts.append(str(x))

result = " ".join(parts)
```

## Formatted strings

```python
name = "Alice"
score = 91.23456

print(f"{name}: {score:.2f}")
```

Useful formatting:

```python
f"{x:.3f}"     # 3 digits after decimal
f"{x:08d}"     # integer padded to width 8
f"{x:b}"       # binary
f"{x:x}"       # hex
f"{x:e}"       # scientific notation
```

---

# 9. Functions, lambdas, unpacking, scope

## Normal function

```python
def add(a, b):
    return a + b
```

## Multiple return values

```python
def stats(a):
    return min(a), max(a), sum(a)

lo, hi, total = stats(nums)
```

Technically it returns a tuple.

## Default parameter

```python
def power(x, p=2):
    return x ** p
```

## Keyword arguments

```python
power(x=3, p=4)
```

## Variable positional arguments

```python
def total(*args):
    return sum(args)

total(1, 2, 3)
```

`args` is a tuple.

## Variable keyword arguments

```python
def show(**kwargs):
    print(kwargs)

show(name="A", age=20)
# {'name': 'A', 'age': 20}
```

## Unpack function arguments

```python
p = (3, 4)
print(pow(*p))
```

Dictionary keyword unpacking:

```python
options = {"sep": "-", "end": "!\n"}
print("a", "b", **options)
```

## Lambda

```python
f = lambda x: x*x
```

Most useful as a short callback:

```python
nums.sort(key=lambda x: abs(x))
```

## Scope

```python
x = 10

def f():
    x = 20  # local x
```

Modify global:

```python
global x
```

Nested function modifying enclosing variable:

```python
nonlocal count
```

In most CP problems, simply pass data as parameters and return results instead of relying heavily on globals.

## Mutable default argument trap

WRONG:

```python
def f(a=[]):
    a.append(1)
    return a
```

The same list persists across calls.

Correct:

```python
def f(a=None):
    if a is None:
        a = []
```

---

# 10. Useful standard-library tools for CP/LeetCode

# 10.1 `collections.deque`

Efficient queue / double-ended queue.

```python
from collections import deque

q = deque()

q.append(x)
q.appendleft(x)

q.pop()
q.popleft()
```

BFS:

```python
q = deque([start])

while q:
    u = q.popleft()
    ...
```

---

# 10.2 `defaultdict`

```python
from collections import defaultdict

graph = defaultdict(list)

for u, v in edges:
    graph[u].append(v)
```

Frequency:

```python
freq = defaultdict(int)

for x in nums:
    freq[x] += 1
```

Sets:

```python
groups = defaultdict(set)
```

---

# 10.3 `Counter`

```python
from collections import Counter

cnt = Counter("banana")
```

```python
cnt["a"]       # 3
cnt.most_common(2)
```

Counter subtraction/addition also exists:

```python
c1 + c2
c1 - c2
```

---

# 10.4 `heapq`

Python `heapq` is a **min-heap**.

```python
import heapq

heap = []

heapq.heappush(heap, 5)
heapq.heappush(heap, 2)

x = heapq.heappop(heap)
# 2
```

Build heap:

```python
heapq.heapify(nums)
```

Max-heap classic technique:

```python
heapq.heappush(heap, -x)
largest = -heapq.heappop(heap)
```

Heap of tuples:

```python
heapq.heappush(heap, (distance, node))
```

Useful for Dijkstra.

Useful helpers:

```python
heapq.nsmallest(k, nums)
heapq.nlargest(k, nums)
```

---

# 10.5 `bisect`

Binary search in sorted lists.

```python
from bisect import bisect_left, bisect_right, insort
```

```python
a = [1, 2, 2, 2, 5]

bisect_left(a, 2)
# 1

bisect_right(a, 2)
# 4
```

Number of `x`:

```python
count = bisect_right(a, x) - bisect_left(a, x)
```

Insert while keeping sorted:

```python
insort(a, x)
```

---

# 10.6 `itertools`

```python
from itertools import permutations, combinations, product, accumulate
```

Permutations:

```python
list(permutations([1,2,3], 2))
```

Combinations:

```python
list(combinations([1,2,3,4], 2))
```

Cartesian product:

```python
list(product([0,1], repeat=3))
```

Prefix sums:

```python
prefix = list(accumulate(nums))
```

Group consecutive equal keys:

```python
from itertools import groupby

for key, group in groupby(sorted(nums)):
    vals = list(group)
```

---

# 10.7 `math`

```python
import math

math.gcd(a, b)
math.lcm(a, b)
math.sqrt(x)
math.isqrt(x)
math.ceil(x)
math.floor(x)
math.factorial(n)
math.log(x)
math.log2(x)
math.sin(x)
math.cos(x)
math.pi
math.inf
```

---

# 10.8 Memoization

```python
from functools import lru_cache

@lru_cache(None)
def dp(i, state):
    ...
```

or modern:

```python
from functools import cache

@cache
def dp(i):
    ...
```

---

# 10.9 Min/max with keys

```python
longest = max(words, key=len)
```

```python
best = max(students, key=lambda x: x[1])
```

Index of largest Python-list value:

```python
idx = max(range(len(a)), key=a.__getitem__)
```

NumPy has `np.argmax`, which is easier for arrays.

---

# 11. Input/output patterns for contests

## One integer

```python
n = int(input())
```

## Two integers

```python
a, b = map(int, input().split())
```

## Integer array

```python
nums = list(map(int, input().split()))
```

## Multiple rows

```python
matrix = [list(map(int, input().split())) for _ in range(n)]
```

## Faster input

```python
import sys

input = sys.stdin.readline
```

Remember `readline()` includes newline, but `.split()` handles it.

## Read all tokens

Very useful:

```python
import sys

data = sys.stdin.buffer.read().split()
it = iter(data)

n = int(next(it))
a = [int(next(it)) for _ in range(n)]
```

## Print list

```python
print(*nums)
```

Separator:

```python
print(*nums, sep=",")
```

Build many output lines:

```python
out = []

for ...:
    out.append(str(answer))

print("\n".join(out))
```

---

# 12. Python OOP

Python OOP is conceptually similar to Java/C++, but syntax is smaller.

# 12.1 Class

```python
class Student:
    def __init__(self, name, score):
        self.name = name
        self.score = score

    def passed(self):
        return self.score >= 50
```

Usage:

```python
s = Student("Alice", 90)
print(s.name)
print(s.passed())
```

`self` is the current object.

---

# 12.2 Instance vs class variables

```python
class Student:
    school = "BUET"           # class variable

    def __init__(self, name):
        self.name = name      # instance variable
```

---

# 12.3 String representation

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __repr__(self):
        return f"Point({self.x}, {self.y})"
```

---

# 12.4 Inheritance

```python
class Animal:
    def speak(self):
        raise NotImplementedError

class Dog(Animal):
    def speak(self):
        return "woof"
```

## `super()`

```python
class Student(Person):
    def __init__(self, name, score):
        super().__init__(name)
        self.score = score
```

---

# 12.5 Abstract classes

```python
from abc import ABC, abstractmethod

class Shape(ABC):
    @abstractmethod
    def area(self):
        pass
```

---

# 12.6 Static/class methods

```python
class MathUtil:
    @staticmethod
    def square(x):
        return x*x
```

```python
class Person:
    count = 0

    @classmethod
    def how_many(cls):
        return cls.count
```

---

# 12.7 Properties

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @radius.setter
    def radius(self, value):
        if value < 0:
            raise ValueError("negative radius")
        self._radius = value
```

---

# 12.8 Dataclass

Very useful for lightweight records:

```python
from dataclasses import dataclass

@dataclass
class Edge:
    u: int
    v: int
    weight: int
```

Automatically creates initializer and representation.

---

# 12.9 Operator overloading

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y

    def __add__(self, other):
        return Point(self.x + other.x, self.y + other.y)
```

Other methods include:
- `__eq__`
- `__lt__`
- `__len__`
- `__getitem__`
- `__iter__`

---

# 13. NumPy fundamentals

```python
import numpy as np
```

## Create arrays

```python
a = np.array([1, 2, 3])
```

2D:

```python
A = np.array([
    [1, 2, 3],
    [4, 5, 6]
])
```

3D:

```python
X = np.array([
    [[1,2], [3,4]],
    [[5,6], [7,8]]
])
```

## Shape information

```python
A.shape
# (2, 3)

A.ndim
# 2

A.size
# 6

A.dtype
```

Interpretation:

```python
shape = (rows, columns)
```

For 3D:

```python
shape = (depth_or_batch, rows, columns)
```

The exact semantic meaning of dimensions is your choice.

## Create common arrays

```python
np.zeros(5)
np.ones(5)
np.full(5, 7)
np.empty(5)
```

2D:

```python
np.zeros((3, 4))
np.ones((2, 5))
```

Identity:

```python
np.eye(4)
```

Range:

```python
np.arange(0, 10, 2)
# [0,2,4,6,8]
```

Evenly spaced by number of points:

```python
np.linspace(0, 1, 5)
# [0, .25, .5, .75, 1]
```

DSP time vectors often use:

```python
t = np.linspace(0, T, N, endpoint=False)
```

Why `endpoint=False`? If representing one periodic interval, you often do not want both `t=0` and `t=T`, because they describe the same periodic point.

## `arange` vs `linspace`

Use `arange` when step size is conceptually exact/integer-like.

```python
np.arange(0, 10, 1)
```

Use `linspace` when you know start, end, number of samples.

```python
np.linspace(0, 1, 1000)
```

For floating steps, `linspace` is often safer.

---

# 13.1 Dtype

```python
a = np.array([1,2,3], dtype=np.float64)
```

Common dtypes:

```python
np.int32
np.int64
np.float32
np.float64
np.complex64
np.complex128
np.bool_
```

Convert:

```python
b = a.astype(np.float64)
```

Important: `astype` normally creates a new array.

---

# 13.2 Elementwise arithmetic

```python
a + b
a - b
a * b
a / b
a ** 2
```

All elementwise.

```python
np.sqrt(a)
np.exp(a)
np.log(a)
np.sin(a)
np.cos(a)
np.abs(a)
```

Comparisons also elementwise:

```python
a > 3
```

returns a Boolean array.

---

# 13.3 Matrix multiplication

Elementwise multiply:

```python
A * B
```

Matrix multiply:

```python
A @ B
```

or:

```python
np.matmul(A, B)
```

Dot product 1D:

```python
a @ b
```

or:

```python
np.dot(a, b)
```

---

# 14. NumPy indexing: 1D, 2D, 3D

# 14.1 1D

```python
a = np.array([10,20,30,40,50])
```

```python
a[0]
a[-1]
a[1:4]
a[::2]
a[::-1]
```

Same basic slicing idea as Python lists.

---

# 14.2 2D

```python
A = np.array([
    [10, 11, 12, 13],
    [20, 21, 22, 23],
    [30, 31, 32, 33]
])
```

Shape:

```python
A.shape
# (3, 4)
```

Element at row 1, column 2:

```python
A[1, 2]
# 22
```

Also works:

```python
A[1][2]
```

But `A[1, 2]` is preferred.

## Entire row

```python
A[1, :]
```

or simply:

```python
A[1]
```

Output shape:

```text
(4,)
```

## Entire column

```python
A[:, 2]
```

Output shape:

```text
(3,)
```

## Row but keep it 2D

```python
A[1:2, :]
```

Shape:

```text
(1, 4)
```

## Column but keep it 2D

```python
A[:, 2:3]
```

Shape:

```text
(3, 1)
```

This distinction matters for broadcasting and matrix multiplication.

## Submatrix

Rows 0-1, columns 1-3:

```python
A[0:2, 1:4]
```

## Every other row/column

```python
A[::2, ::2]
```

## Reverse rows

```python
A[::-1, :]
```

## Reverse columns

```python
A[:, ::-1]
```

## Reverse both

```python
A[::-1, ::-1]
```

---

# 14.3 3D

Suppose:

```python
X.shape == (2, 3, 4)
```

Interpret dimensions as:

```text
axis 0: blocks
axis 1: rows
axis 2: columns
```

Access:

```python
X[block, row, col]
```

Examples:

```python
X[0]          # first 2D block, shape (3,4)
X[:, 1, :]    # row 1 from every block, shape (2,4)
X[:, :, 2]    # column 2 from every block, shape (2,3)
X[1, :, :]    # block 1
X[1, 0, :]    # row 0 from block 1
```

Last column of every row of every block:

```python
X[:, :, -1]
```

---

# 14.4 Fancy indexing

```python
a = np.array([10,20,30,40,50])

a[[0, 3, 4]]
# [10,40,50]
```

2D selected elements:

```python
A[[0,2], [1,3]]
```

This selects:
- `A[0,1]`
- `A[2,3]`

It does **not** select the Cartesian submatrix.

For a Cartesian selection:

```python
A[np.ix_([0,2], [1,3])]
```

---

# 15. Views vs copies in NumPy

Extremely important.

## Basic slicing usually gives a view

```python
a = np.array([1,2,3,4,5])
b = a[1:4]

b[0] = 99
print(a)
# [1,99,3,4,5]
```

`b` shares memory with `a`.

If you need independent data:

```python
b = a[1:4].copy()
```

## Fancy indexing usually gives a copy

```python
b = a[[1,2,3]]
```

Changing `b` normally does not change `a`.

## Boolean indexing also gives a copy

```python
b = a[a > 0]
```

## `reshape`

Often returns a view when possible:

```python
B = A.reshape(2, 3)
```

Do not rely blindly on whether memory is shared; if independence matters, use `.copy()`.

## Check memory sharing

```python
np.shares_memory(a, b)
```

---

# 16. Shape manipulation

Suppose:

```python
a = np.arange(12)
```

## Reshape

```python
A = a.reshape(3, 4)
```

Use `-1` to infer one dimension:

```python
A = a.reshape(3, -1)
```

Flatten:

```python
flat = A.reshape(-1)
```

or:

```python
flat = A.ravel()
```

`ravel()` tries to return a view when possible.

Guaranteed copy:

```python
flat = A.flatten()
```

## Transpose

2D:

```python
A.T
```

For 1D array:

```python
a.T
```

does nothing to shape because `(N,)` has only one axis.

To make row/column vectors:

```python
row = a.reshape(1, -1)  # (1,N)
col = a.reshape(-1, 1)  # (N,1)
```

or:

```python
row = a[None, :]
col = a[:, None]
```

## Add axis

```python
a[:, None]
a[None, :]
```

`None` is equivalent to `np.newaxis`.

Example:

```python
a.shape
# (5,)

a[:, None].shape
# (5,1)

a[None, :].shape
# (1,5)
```

## Remove length-1 axes

```python
A.squeeze()
```

Careful: it may remove multiple singleton dimensions.

Specific axis:

```python
A.squeeze(axis=1)
```

## Move/swap axes

```python
np.swapaxes(A, 0, 1)
np.moveaxis(X, 0, -1)
```

## General transpose

```python
X.transpose(2, 0, 1)
```

reorders axes.

---

# 17. The `axis` parameter

This confuses almost everyone initially.

Think:

> `axis=k` means "perform the operation by collapsing / working along dimension k."

For:

```python
A = np.array([
    [1,2,3],
    [4,5,6]
])
# shape (2,3)
```

## `axis=0`

Collapse rows. Values travel vertically down each column.

```python
A.sum(axis=0)
# [5,7,9]
```

Output has one value per **column**.

## `axis=1`

Collapse columns. Values travel horizontally across each row.

```python
A.sum(axis=1)
# [6,15]
```

Output has one value per **row**.

Memory trick for 2D:

```text
axis=0 -> remove row dimension -> result indexed by columns
axis=1 -> remove column dimension -> result indexed by rows
```

## 3D example

If:

```python
X.shape == (2,3,4)
```

then:

```python
X.sum(axis=0).shape  # (3,4)
X.sum(axis=1).shape  # (2,4)
X.sum(axis=2).shape  # (2,3)
```

The selected axis disappears unless `keepdims=True`.

```python
X.sum(axis=1, keepdims=True).shape
# (2,1,4)
```

`keepdims=True` is extremely useful for broadcasting later.

## Multiple axes

```python
X.sum(axis=(1,2))
```

Collapses axes 1 and 2.

---

# 18. Broadcasting

Broadcasting lets NumPy combine arrays with compatible shapes without manually copying values.

Rules compare dimensions from **right to left**.

Dimensions are compatible if:
1. equal, or
2. one of them is `1`.

Example:

```text
A: (3, 4)
b:    (4,)
```

Treat `b` as `(1,4)`:

```text
A: (3,4)
b: (1,4)
```

Compatible -> `b` is broadcast across all rows.

```python
A + b
```

## Add one value per column

```python
A.shape == (3,4)
b.shape == (4,)

A + b
```

## Add one value per row

Need shape `(3,1)`:

```python
r = np.array([10,20,30])
A + r[:, None]
```

because:

```text
A: (3,4)
r: (3,1)
```

The `1` expands to 4 columns.

---

# 18.1 Outer operations

```python
x = np.array([1,2,3])       # (3,)
y = np.array([10,20])       # (2,)
```

Want every `x` with every `y`:

```python
x[:, None] + y[None, :]
```

Shapes:

```text
x[:,None]: (3,1)
y[None,:]: (1,2)
result:    (3,2)
```

Example output:

```text
[[11,21],
 [12,22],
 [13,23]]
```

Outer product:

```python
x[:, None] * y[None, :]
```

or:

```python
np.outer(x, y)
```

---

# 18.2 DSP broadcasting example

Suppose you need:

```text
e^(j*n*w0*t)
```

for:
- many harmonics `n`;
- many time samples `t`.

```python
n = np.arange(-N, N+1)       # shape (H,)
t = np.linspace(0, T, M)     # shape (M,)

E = np.exp(1j * n[:, None] * w0 * t[None, :])
```

Shapes:

```text
n[:,None]  -> (H,1)
t[None,:]  -> (1,M)
E          -> (H,M)
```

Each row corresponds to one harmonic across all time samples.

This is the core idea behind vectorized Fourier-series calculations.

---

# 18.3 Broadcasting checklist

Before writing a vectorized expression:
1. write every array's shape;
2. align shapes from the right;
3. decide which dimensions should vary independently;
4. insert singleton dimensions with `None` / `np.newaxis`.

Example:

```python
print(a.shape)
print(b.shape)
```

Use this constantly while learning.

---

# 19. Boolean masks and conditional selection

```python
a = np.array([1,5,2,9,3])
mask = a > 3
```

`mask`:

```text
[False, True, False, True, False]
```

Select:

```python
a[mask]
# [5,9]
```

Direct:

```python
a[a > 3]
```

## Multiple conditions

Use:
- `&` for elementwise AND;
- `|` for elementwise OR;
- `~` for elementwise NOT.

```python
a[(a >= 2) & (a <= 5)]
```

Do not write:

```python
a[a >= 2 and a <= 5]   # WRONG for arrays
```

Use parentheses around each comparison.

## Modify selected values

```python
a[a < 0] = 0
```

## `np.where`

Two-form conditional:

```python
out = np.where(a >= 0, a, -a)
```

Equivalent idea:

```text
if condition true: choose a
else: choose -a
```

Get indices:

```python
idx = np.where(a > 3)
```

For 1D:

```python
idx = np.where(a > 3)[0]
```

## `np.clip`

```python
np.clip(a, 0, 1)
```

Clamps to `[0,1]`.

---

# 20. Sorting, searching, ranking in NumPy

## Sort

```python
np.sort(a)
```

returns new array.

In-place:

```python
a.sort()
```

## Sort descending

```python
np.sort(a)[::-1]
```

## Argsort

Returns indices that would sort the array.

```python
a = np.array([50, 10, 30])

idx = np.argsort(a)
# [1,2,0]

a[idx]
# [10,30,50]
```

Descending:

```python
idx = np.argsort(a)[::-1]
```

## Sort 2D by axis

```python
np.sort(A, axis=0)
```

Sorts each column independently.

```python
np.sort(A, axis=1)
```

Sorts each row independently.

Flatten then sort:

```python
np.sort(A, axis=None)
```

## Sort rows by one column

```python
A[A[:, 1].argsort()]
```

Sort descending by column 1:

```python
A[A[:, 1].argsort()[::-1]]
```

## Multiple keys with `lexsort`

Suppose columns:
- col 0 = primary;
- col 1 = secondary.

`np.lexsort` uses the **last key as primary**.

```python
idx = np.lexsort((A[:,1], A[:,0]))
sorted_A = A[idx]
```

Here `A[:,0]` is primary.

## `argmin` / `argmax`

```python
np.argmax(a)
np.argmin(a)
```

For 2D:

```python
A.argmax(axis=0)
A.argmax(axis=1)
```

Global flat index:

```python
idx = np.argmax(A)
```

Convert to coordinates:

```python
r, c = np.unravel_index(idx, A.shape)
```

## Top-k

Simple:

```python
idx = np.argsort(a)[-k:][::-1]
```

Faster for large arrays:

```python
idx = np.argpartition(a, -k)[-k:]
idx = idx[np.argsort(a[idx])[::-1]]
```

## Search sorted

```python
np.searchsorted(a, x)
```

Equivalent idea to `bisect_left`.

```python
np.searchsorted(a, x, side="right")
```

Equivalent to `bisect_right`.

---

# 21. Combining and splitting arrays

Suppose:

```python
a = np.array([1,2,3])
b = np.array([4,5,6])
```

## Concatenate

```python
np.concatenate([a, b])
# [1,2,3,4,5,6]
```

2D:

```python
np.concatenate([A, B], axis=0)
```

Stacks rows if column counts match.

```python
np.concatenate([A, B], axis=1)
```

Stacks columns if row counts match.

## `stack`

Creates a **new axis**.

```python
np.stack([a,b], axis=0)
```

shape `(2,3)`.

```python
np.stack([a,b], axis=1)
```

shape `(3,2)`.

Difference:

- `concatenate`: joins along an existing axis.
- `stack`: creates a new axis.

## Convenience functions

```python
np.vstack([a,b])
np.hstack([a,b])
np.column_stack([a,b])
```

For 1D arrays:

```python
np.column_stack([a,b])
```

is very handy for making `(N,2)` data.

## Split

```python
np.split(a, 3)
```

Requires equal division.

Unequal allowed:

```python
np.array_split(a, 3)
```

2D convenience:

```python
np.vsplit(A, 2)
np.hsplit(A, 2)
```

---

# 22. Math, statistics, complex numbers

## Reductions

```python
np.sum(a)
np.mean(a)
np.min(a)
np.max(a)
np.std(a)
np.var(a)
np.median(a)
np.prod(a)
```

Axis:

```python
A.mean(axis=0)
A.mean(axis=1)
```

## Cumulative

```python
np.cumsum(a)
np.cumprod(a)
```

Differences:

```python
np.diff(a)
```

For:

```python
a = [1,4,9,16]
```

result:

```text
[3,5,7]
```

Second difference:

```python
np.diff(a, n=2)
```

## Unique

```python
np.unique(a)
```

Counts:

```python
values, counts = np.unique(a, return_counts=True)
```

Inverse mapping:

```python
values, inverse = np.unique(a, return_inverse=True)
```

## Bincount

For nonnegative integers:

```python
np.bincount(a)
```

Often faster than dictionary frequency counts.

## NaN-aware versions

```python
np.nanmean(a)
np.nansum(a)
np.nanmax(a)
```

---

# 22.1 Complex numbers

Python:

```python
z = 3 + 4j
```

NumPy:

```python
z = np.array([1+2j, 3-4j])
```

Real/imaginary parts:

```python
z.real
z.imag
```

Conjugate:

```python
np.conj(z)
z.conj()
```

Magnitude:

```python
np.abs(z)
```

Phase:

```python
np.angle(z)
```

Phase in degrees:

```python
np.angle(z, deg=True)
```

Complex exponential:

```python
np.exp(1j * theta)
```

Euler:

```text
e^(jθ) = cos θ + j sin θ
```

Construct from magnitude and phase:

```python
z = magnitude * np.exp(1j * phase)
```

Unwrap phase:

```python
phase_unwrapped = np.unwrap(np.angle(z))
```

---

# 23. Interpolation

One-dimensional linear interpolation:

```python
np.interp(x_new, x_known, y_known)
```

Example:

```python
x = np.array([0, 1, 2, 3])
y = np.array([0, 10, 20, 30])

np.interp(1.5, x, y)
# 15.0
```

Multiple points:

```python
x_new = np.array([0.5, 1.5, 2.2])
y_new = np.interp(x_new, x, y)
```

Important assumptions:
- `x_known` should normally be increasing;
- interpolation is piecewise linear;
- outside the range, NumPy by default uses endpoint values.

Specify left/right:

```python
np.interp(x_new, x, y, left=-1, right=999)
```

## Resampling example

```python
t_old = np.linspace(0, 1, 100)
x_old = np.sin(2*np.pi*5*t_old)

t_new = np.linspace(0, 1, 1000)
x_new = np.interp(t_new, t_old, x_old)
```

This gives linearly interpolated samples. It is not equivalent to ideal bandlimited resampling.

---

# 24. Numerical integration

For sampled data, the trapezoidal rule is fundamental.

Modern NumPy:

```python
area = np.trapezoid(y, x)
```

If samples have constant spacing `dx`:

```python
area = np.trapezoid(y, dx=dt)
```

Older material may use:

```python
np.trapz(y, x)
```

Same basic trapezoidal idea.

## Example

Approximate:

```text
∫₀^π sin(x) dx
```

```python
x = np.linspace(0, np.pi, 1000)
y = np.sin(x)

area = np.trapezoid(y, x)
# approximately 2
```

## Integrate rows or columns

For:

```python
Z.shape == (rows, cols)
```

Integrate along columns:

```python
np.trapezoid(Z, x=x, axis=1)
```

Integrate along rows:

```python
np.trapezoid(Z, x=y, axis=0)
```

## Double integration

For sampled `f(y,x)`:

```python
temp = np.trapezoid(F, x=x, axis=1)
result = np.trapezoid(temp, x=y, axis=0)
```

Or choose axes according to your array layout.

Always know which axis corresponds to which physical variable.

---

# 25. Numerical derivatives and gradients

## `np.diff`

Simple adjacent differences:

```python
dy = np.diff(y)
dx = np.diff(x)

slope = dy / dx
```

Result has length `N-1`.

## `np.gradient`

Estimates derivative while keeping same length.

```python
dy_dx = np.gradient(y, x)
```

Example:

```python
x = np.linspace(0, 2*np.pi, 1000)
y = np.sin(x)

dy = np.gradient(y, x)
# approximately cos(x)
```

## 2D gradient

```python
gy, gx = np.gradient(F, dy, dx)
```

For image-like array `F[y, x]`:
- first returned gradient corresponds to axis 0 / rows / y;
- second corresponds to axis 1 / columns / x.

If coordinates vary:

```python
gy, gx = np.gradient(F, y, x)
```

---

# 26. Meshgrids and multidimensional coordinates

Suppose:

```python
x = np.array([0,1,2])
y = np.array([10,20])
```

```python
X, Y = np.meshgrid(x, y)
```

Then:

```python
X.shape == (2,3)
Y.shape == (2,3)
```

Conceptually:

```text
X =
[[0,1,2],
 [0,1,2]]

Y =
[[10,10,10],
 [20,20,20]]
```

Now evaluate:

```python
Z = X**2 + Y
```

## DSP/image frequency grid

```python
u = np.linspace(-Umax, Umax, Nu)
v = np.linspace(-Vmax, Vmax, Nv)

U, V = np.meshgrid(u, v)
R = np.sqrt(U**2 + V**2)
```

Circular mask:

```python
mask = R >= cutoff
```

## `indexing="ij"`

```python
X, Y = np.meshgrid(x, y, indexing="ij")
```

Then dimension ordering follows input arrays more directly.

Default is `indexing="xy"`, which is convenient for Cartesian plots.

---

# 27. Linear algebra

```python
import numpy as np
```

## Dot product

```python
a @ b
```

## Matrix multiplication

```python
A @ B
```

## Transpose

```python
A.T
```

## Solve linear system

Instead of computing inverse manually:

```python
x = np.linalg.solve(A, b)
```

for:

```text
Ax = b
```

## Inverse

```python
np.linalg.inv(A)
```

Usually avoid inverse when `solve` directly solves your problem.

## Determinant

```python
np.linalg.det(A)
```

## Norm

```python
np.linalg.norm(a)
```

Euclidean norm.

## Eigenvalues

```python
vals, vecs = np.linalg.eig(A)
```

## SVD

```python
U, S, Vh = np.linalg.svd(A)
```

---

# 28. FFT and DSP essentials

# 28.1 FFT

```python
X = np.fft.fft(x)
```

Inverse:

```python
x_rec = np.fft.ifft(X)
```

For real signals, reconstructed output can contain tiny imaginary numerical error:

```python
x_rec = np.fft.ifft(X).real
```

## Frequency bins

```python
freq = np.fft.fftfreq(N, d=dt)
```

where:
- `N` = number of samples;
- `dt` = sample spacing;
- sampling frequency `fs = 1/dt`.

Equivalent:

```python
freq = np.fft.fftfreq(N, d=1/fs)
```

## Center zero frequency

```python
Xc = np.fft.fftshift(X)
fc = np.fft.fftshift(freq)
```

Undo:

```python
np.fft.ifftshift(...)
```

## Magnitude and phase

```python
mag = np.abs(X)
phase = np.angle(X)
```

Log magnitude:

```python
mag_db = 20 * np.log10(np.abs(X) + 1e-12)
```

or visual-only compression:

```python
logmag = np.log1p(np.abs(X))
```

`1e-12` avoids `log10(0)`.

---

# 28.2 FFT normalization

Raw FFT:

```python
X = np.fft.fft(x)
```

A common amplitude-normalized version:

```python
Xn = X / N
```

For a single-sided spectrum of a real signal:

```python
X = np.fft.rfft(x)
f = np.fft.rfftfreq(N, d=1/fs)

mag = np.abs(X) / N
```

Depending on your amplitude convention, interior non-DC/non-Nyquist bins may be doubled for a single-sided amplitude spectrum.

Always follow the convention expected by the lab/course.

---

# 28.3 2D FFT

```python
F = np.fft.fft2(image)
Fc = np.fft.fftshift(F)
```

Magnitude:

```python
spectrum = np.log1p(np.abs(Fc))
```

Inverse:

```python
F_unshifted = np.fft.ifftshift(Fc)
img_rec = np.fft.ifft2(F_unshifted).real
```

## Frequency coordinates

```python
fy = np.fft.fftfreq(rows, d=dy)
fx = np.fft.fftfreq(cols, d=dx)

fy = np.fft.fftshift(fy)
fx = np.fft.fftshift(fx)

FX, FY = np.meshgrid(fx, fy)
```

Then:

```python
R = np.sqrt(FX**2 + FY**2)
```

High-pass:

```python
mask = R >= cutoff
F_filtered = Fc * mask
```

Low-pass:

```python
mask = R <= cutoff
```

---

# 28.4 Fourier-series vectorization pattern

Suppose coefficients:

```python
c.shape == (H,)
n.shape == (H,)
t.shape == (M,)
```

Then:

```python
E = np.exp(1j * n[:, None] * w0 * t[None, :])
x_hat = np.sum(c[:, None] * E, axis=0)
```

Shapes:

```text
c[:,None] -> (H,1)
E         -> (H,M)
product   -> (H,M)
sum axis0 -> (M,)
```

This one pattern is worth memorizing.

---

# 29. Convolution and correlation

## 1D convolution

```python
y = np.convolve(x, h, mode="full")
```

Modes:
- `"full"`: all overlap positions;
- `"same"`: output size based on input;
- `"valid"`: only complete overlap.

## Correlation

```python
r = np.correlate(x, y, mode="full")
```

For complex-signal theoretical correlation, be aware of conjugation conventions; inspect course definitions.

## Moving average

```python
kernel = np.ones(M) / M
smooth = np.convolve(x, kernel, mode="same")
```

---

# 30. Common NumPy operations cheat sheet

## Creation

```python
np.array(...)
np.zeros(shape)
np.ones(shape)
np.full(shape, value)
np.arange(start, stop, step)
np.linspace(start, stop, num)
np.eye(n)
```

## Inspection

```python
a.shape
a.ndim
a.size
a.dtype
```

## Conversion

```python
a.astype(float)
a.tolist()
np.asarray(x)
```

## Indexing

```python
a[i]
a[start:stop:step]
A[r, c]
A[r, :]
A[:, c]
A[r1:r2, c1:c2]
```

## Reshape

```python
a.reshape(...)
a.ravel()
a.flatten()
A.T
a[:, None]
a[None, :]
np.squeeze(a)
```

## Combine

```python
np.concatenate(...)
np.stack(...)
np.vstack(...)
np.hstack(...)
np.column_stack(...)
```

## Math

```python
np.abs
np.sqrt
np.exp
np.log
np.log10
np.sin
np.cos
np.angle
np.conj
```

## Reduction

```python
np.sum
np.mean
np.min
np.max
np.std
np.var
np.argmin
np.argmax
```

## Selection

```python
a[mask]
np.where(...)
np.clip(...)
```

## Sorting/search

```python
np.sort
np.argsort
np.lexsort
np.searchsorted
np.unique
```

## Difference/calculus

```python
np.diff
np.gradient
np.trapezoid
```

## DSP

```python
np.interp
np.convolve
np.correlate
np.fft.fft
np.fft.ifft
np.fft.fft2
np.fft.ifft2
np.fft.fftfreq
np.fft.fftshift
np.fft.ifftshift
```

---

# 31. Matplotlib fundamentals

```python
import matplotlib.pyplot as plt
```

## Basic line plot

```python
x = np.linspace(0, 2*np.pi, 1000)
y = np.sin(x)

plt.plot(x, y)
plt.xlabel("Time")
plt.ylabel("Amplitude")
plt.title("Sine wave")
plt.grid(True)
plt.show()
```

## Multiple curves

```python
plt.plot(x, np.sin(x), label="sin")
plt.plot(x, np.cos(x), label="cos")
plt.legend()
plt.show()
```

## Line style / marker

```python
plt.plot(x, y, "--")
plt.plot(x, y, marker="o")
```

## Scatter

```python
plt.scatter(x, y)
```

## Stem plot

Very useful for discrete-time signals / spectra:

```python
plt.stem(n, x)
```

## Bar chart

```python
plt.bar(labels, values)
```

## Histogram

```python
plt.hist(data, bins=30)
```

## Axis limits

```python
plt.xlim(-5, 5)
plt.ylim(-1.2, 1.2)
```

## Log scales

```python
plt.semilogy(x, y)
plt.semilogx(x, y)
plt.loglog(x, y)
```

or:

```python
plt.yscale("log")
```

## Save

```python
plt.savefig("figure.png", dpi=300, bbox_inches="tight")
```

## New figure

```python
plt.figure(figsize=(8, 5))
```

## Close

```python
plt.close()
```

Useful in loops.

---

# 31.1 Subplots

```python
fig, ax = plt.subplots(2, 1)

ax[0].plot(t, x)
ax[0].set_title("Signal")

ax[1].plot(f, mag)
ax[1].set_title("Spectrum")

plt.tight_layout()
plt.show()
```

For a 2D grid:

```python
fig, ax = plt.subplots(2, 2)
ax[0,0].plot(...)
```

---

# 31.2 Object-oriented Matplotlib style

Recommended for nontrivial figures:

```python
fig, ax = plt.subplots()

ax.plot(x, y)
ax.set_xlabel("x")
ax.set_ylabel("y")
ax.set_title("Title")
ax.grid(True)

plt.show()
```

This scales better than relying entirely on global `plt.*`.

---

# 31.3 Images

```python
plt.imshow(image, cmap="gray")
plt.colorbar()
plt.show()
```

For mathematical coordinates:

```python
plt.imshow(
    F,
    extent=[x_min, x_max, y_min, y_max],
    origin="lower",
    aspect="auto"
)
```

Important image parameters:

- `cmap="gray"`: grayscale;
- `origin="lower"`: first row shown at bottom;
- `extent=[xmin,xmax,ymin,ymax]`: physical coordinate labeling;
- `aspect="auto"`: allow axes to stretch;
- `vmin`, `vmax`: control value range.

---

# 31.4 3D plotting basics

```python
fig = plt.figure()
ax = fig.add_subplot(111, projection="3d")

ax.plot_surface(X, Y, Z)
plt.show()
```

Wireframe:

```python
ax.plot_wireframe(X, Y, Z)
```

---

# 32. DSP plotting patterns

## Continuous-looking signal

```python
t = np.linspace(0, 1, 2000)
x = np.sin(2*np.pi*5*t)

plt.plot(t, x)
plt.xlabel("t")
plt.ylabel("x(t)")
plt.show()
```

## Discrete signal

```python
n = np.arange(20)
x = np.cos(0.2*np.pi*n)

plt.stem(n, x)
plt.xlabel("n")
plt.ylabel("x[n]")
plt.show()
```

## Magnitude spectrum

```python
X = np.fft.fft(x)
f = np.fft.fftfreq(len(x), d=1/fs)

X = np.fft.fftshift(X)
f = np.fft.fftshift(f)

plt.plot(f, np.abs(X))
plt.xlabel("Frequency (Hz)")
plt.ylabel("|X(f)|")
plt.show()
```

## Phase spectrum

```python
plt.plot(f, np.angle(X))
```

Unwrapped:

```python
plt.plot(f, np.unwrap(np.angle(X)))
```

## 2D spectrum

```python
F = np.fft.fftshift(np.fft.fft2(image))

plt.imshow(np.log1p(np.abs(F)), cmap="gray", origin="lower")
plt.colorbar()
plt.show()
```

## Complex trajectory

```python
z = x + 1j*y

plt.plot(z.real, z.imag)
plt.axis("equal")
plt.xlabel("Real")
plt.ylabel("Imag")
plt.show()
```

---

# 33. Common mistakes and debugging

# 33.1 `list.sort()` returns `None`

Wrong:

```python
b = a.sort()
```

Correct:

```python
a.sort()
```

or:

```python
b = sorted(a)
```

---

# 33.2 Confusing list and NumPy multiplication

```python
[1,2,3] * 2
# repetition
```

```python
np.array([1,2,3]) * 2
# arithmetic
```

---

# 33.3 Using `and/or` with NumPy arrays

Wrong:

```python
mask = (a > 0) and (a < 10)
```

Correct:

```python
mask = (a > 0) & (a < 10)
```

---

# 33.4 Forgetting parentheses with masks

Wrong:

```python
a > 0 & a < 10
```

Correct:

```python
(a > 0) & (a < 10)
```

---

# 33.5 Shape mismatch

Print shapes:

```python
print(a.shape, b.shape)
```

Then reason through broadcasting.

---

# 33.6 `A[:, 0]` vs `A[:, 0:1]`

```python
A[:, 0].shape
# (N,)
```

```python
A[:, 0:1].shape
# (N,1)
```

Use the second when you need a 2D column vector.

---

# 33.7 1D transpose does nothing

```python
a.shape == (N,)
a.T.shape == (N,)
```

Use:

```python
a[:, None]
```

---

# 33.8 Accidental NumPy view mutation

```python
b = a[2:5]
b[:] = 0
```

May change `a`.

Use:

```python
b = a[2:5].copy()
```

---

# 33.9 Python nested list aliasing

Wrong:

```python
grid = [[0]*m]*n
```

Correct:

```python
grid = [[0]*m for _ in range(n)]
```

---

# 33.10 Floating-point equality

Risky:

```python
if x == 0.3:
```

Better:

```python
np.isclose(x, 0.3)
```

Arrays:

```python
np.allclose(a, b)
```

---

# 33.11 Integer dtype truncation

```python
a = np.array([1,2,3])
a /= 2
```

Depending on operation/dtype, dtype rules matter.

If you need floating-point math:

```python
a = np.array([1,2,3], dtype=float)
```

---

# 33.12 Debugging NumPy systematically

Print:

```python
print("shape:", a.shape)
print("dtype:", a.dtype)
print("ndim:", a.ndim)
print(a)
```

For several intermediate arrays:

```python
for name, arr in [
    ("n", n),
    ("t", t),
    ("E", E),
    ("coeff", coeff)
]:
    print(name, arr.shape, arr.dtype)
```

Check finite values:

```python
np.all(np.isfinite(a))
```

Check NaN:

```python
np.any(np.isnan(a))
```

Check Inf:

```python
np.any(np.isinf(a))
```

---

# 34. Python↔NumPy conversion guide

## List -> array

```python
a = np.array(my_list)
```

## Array -> list

```python
lst = a.tolist()
```

## Tuple -> array

```python
a = np.array(my_tuple)
```

## Array -> tuple

```python
t = tuple(a)
```

For 2D, this produces a tuple of NumPy row arrays. If you want nested Python tuples:

```python
t = tuple(map(tuple, A))
```

## Dict values -> list

```python
vals = list(d.values())
```

## Dict values -> NumPy array

```python
vals = np.array(list(d.values()))
```

## Dict items -> array-like pairs

```python
pairs = list(d.items())
```

If numeric:

```python
pairs = np.array(list(d.items()))
```

Be careful if keys and values have incompatible types; NumPy may choose string/object dtype.

## Enumerate -> list

```python
pairs = list(enumerate(nums))
```

## Zip -> list

```python
pairs = list(zip(A, B))
```

In Python 3, `zip` itself is an iterator.

---

# 35. Patterns you should memorize

These are the "I should never need Google for this" patterns.

## Read integer list

```python
a = list(map(int, input().split()))
```

## Frequency map

```python
from collections import Counter
cnt = Counter(a)
```

## Dictionary of lists

```python
from collections import defaultdict
g = defaultdict(list)
```

## Sort by second, then first

```python
a.sort(key=lambda x: (x[1], x[0]))
```

## Sort descending score, ascending name

```python
a.sort(key=lambda x: (-x[1], x[0]))
```

## Unique sorted

```python
a = sorted(set(a))
```

## Loop index + value

```python
for i, x in enumerate(a):
    ...
```

## Parallel loop

```python
for x, y in zip(a, b):
    ...
```

## Reverse

```python
a[::-1]
```

## Queue

```python
from collections import deque
q = deque([start])
u = q.popleft()
```

## Min heap

```python
import heapq
heapq.heappush(h, x)
x = heapq.heappop(h)
```

## Binary search insertion index

```python
from bisect import bisect_left
i = bisect_left(a, x)
```

## 2D grid

```python
grid = [[0]*m for _ in range(n)]
```

## NumPy row

```python
A[r, :]
```

## NumPy column

```python
A[:, c]
```

## Keep column dimension

```python
A[:, c:c+1]
```

## Boolean mask

```python
A[A > threshold]
```

## Conditional replace

```python
A[A < 0] = 0
```

## Sort NumPy array and get order

```python
idx = np.argsort(a)
```

## Sort 2D rows by column `c`

```python
A = A[A[:, c].argsort()]
```

## Reverse an axis

```python
A[::-1, :]
A[:, ::-1]
```

## Add dimension

```python
a[:, None]
a[None, :]
```

## Harmonic × time grid

```python
phase = n[:, None] * w0 * t[None, :]
```

## Fourier synthesis

```python
x = np.sum(c[:, None] * np.exp(1j*phase), axis=0)
```

## FFT frequency axis

```python
f = np.fft.fftfreq(N, d=1/fs)
```

## Center FFT

```python
X = np.fft.fftshift(np.fft.fft(x))
f = np.fft.fftshift(np.fft.fftfreq(N, d=1/fs))
```

## Interpolate

```python
y_new = np.interp(x_new, x, y)
```

## Integrate sampled data

```python
area = np.trapezoid(y, x)
```

## Differentiate sampled data

```python
dy = np.gradient(y, x)
```

## 2D coordinate grid

```python
X, Y = np.meshgrid(x, y)
```

## Clip

```python
a = np.clip(a, lo, hi)
```

## Convert list ↔ NumPy

```python
arr = np.array(lst)
lst = arr.tolist()
```

---


# 35A. Advanced toolbox that prevents documentation lookups

These are not the first functions to memorize, but they are very useful once the core material above is comfortable.

## Python arithmetic and bit operations

```python
x + y
x - y
x * y
x / y       # true division -> float
x // y      # floor division
x % y       # remainder/modulo
x ** y      # power
```

Bit operations:

```python
x & y       # bitwise AND
x | y       # bitwise OR
x ^ y       # XOR
~x          # bitwise NOT
x << k      # left shift
x >> k      # right shift
```

Check bit `k`:

```python
if x & (1 << k):
    ...
```

Set bit:

```python
x |= 1 << k
```

Clear bit:

```python
x &= ~(1 << k)
```

Toggle bit:

```python
x ^= 1 << k
```

Binary representation:

```python
bin(x)
f"{x:08b}"
```

## `any` and `all`

Python:

```python
any([False, False, True])   # True
all([True, True, False])    # False
```

Very useful with generators:

```python
if any(x < 0 for x in nums):
    ...

if all(x >= 0 for x in nums):
    ...
```

NumPy:

```python
np.any(A > 0)
np.all(A >= 0)
np.any(A > 0, axis=1)
np.all(A > 0, axis=0)
```

## `nonzero`, `argwhere`, `flatnonzero`

Indices where condition is true:

```python
np.nonzero(A > 5)
```

2D coordinate rows:

```python
coords = np.argwhere(A > 5)
# each row is [row, col]
```

Flattened positions:

```python
idx = np.flatnonzero(a > 5)
```

## `take`, `take_along_axis`

Select indices along one axis:

```python
np.take(A, [0, 2], axis=0)
np.take(A, [1, 3], axis=1)
```

Useful when each row has a different chosen column:

```python
idx = np.array([[2], [0], [1]])
chosen = np.take_along_axis(A, idx, axis=1)
```

## `repeat` vs `tile`

Repeat individual elements:

```python
np.repeat([1,2,3], 2)
# [1,1,2,2,3,3]
```

Repeat an entire pattern:

```python
np.tile([1,2,3], 2)
# [1,2,3,1,2,3]
```

2D examples:

```python
np.repeat(A, 2, axis=0)   # duplicate every row
np.repeat(A, 2, axis=1)   # duplicate every column
```

Use broadcasting instead of `tile` when you only need arithmetic; broadcasting avoids materializing unnecessary copies.

## `roll`

Circular shift:

```python
np.roll(a, 2)
np.roll(A, 1, axis=0)
np.roll(A, -1, axis=1)
```

Useful for periodic signals and circular operations.

## `pad`

Zero padding:

```python
np.pad(a, (2, 3))
```

For 2D:

```python
np.pad(A, ((1,1), (2,2)))
```

Other modes:

```python
np.pad(a, (2,2), mode="edge")
np.pad(a, (2,2), mode="reflect")
np.pad(a, (2,2), mode="constant", constant_values=5)
```

DSP FFT zero-padding example:

```python
x_pad = np.pad(x, (0, Nfft - len(x)))
X = np.fft.fft(x_pad)
```

Zero-padding increases frequency-grid density; it does not create new physical information.

## `delete`, `insert`, `append`

NumPy has:

```python
np.delete(a, 3)
np.insert(a, 2, 99)
np.append(a, [7,8])
```

These return new arrays. NumPy arrays are not dynamic vectors, so repeated append/insert in loops is inefficient. Build a Python list first, then convert:

```python
values = []
for ...:
    values.append(x)

arr = np.array(values)
```

## Diagonal and trace

```python
np.diag(A)
np.trace(A)
```

Create diagonal matrix:

```python
np.diag([1,2,3])
```

## Min/max positions in multidimensional arrays

```python
flat_idx = np.argmax(A)
coord = np.unravel_index(flat_idx, A.shape)
```

Example:

```python
r, c = np.unravel_index(np.argmax(A), A.shape)
```

## Stable sorting

Python's built-in sort is stable.

```python
items.sort(key=lambda x: x[1])
```

Equal-key items retain their earlier relative order.

This permits multi-pass sorting:

```python
items.sort(key=lambda x: x.name)               # secondary
items.sort(key=lambda x: x.score, reverse=True) # primary
```

Usually a tuple key is clearer:

```python
items.sort(key=lambda x: (-x.score, x.name))
```

NumPy sorting also allows a stable algorithm where needed:

```python
np.argsort(a, kind="stable")
```

## `partition` / `argpartition`

When you need only the k smallest/largest values, full sorting may be unnecessary.

```python
smallest_k = np.partition(a, k)[:k]
```

Indices:

```python
idx = np.argpartition(a, k)[:k]
```

The selected section is not guaranteed to be sorted. Sort that small section afterward if needed.

## Random-number generation

Modern NumPy style:

```python
rng = np.random.default_rng(42)
```

Examples:

```python
rng.random(5)                 # uniform [0,1)
rng.integers(0, 10, size=5)  # integers
rng.normal(0, 1, size=1000)   # Gaussian
rng.choice([10,20,30], size=5)
rng.permutation(10)
```

Add Gaussian noise to a signal:

```python
noise = rng.normal(0, sigma, size=x.shape)
y = x + noise
```

Using a seed makes experiments reproducible.

## Saving/loading NumPy arrays

Single array:

```python
np.save("signal.npy", x)
x2 = np.load("signal.npy")
```

Multiple arrays:

```python
np.savez("data.npz", t=t, x=x, fs=fs)

data = np.load("data.npz")
t = data["t"]
x = data["x"]
```

Text:

```python
np.savetxt("data.txt", A)
A = np.loadtxt("data.txt")
```

CSV-like:

```python
A = np.loadtxt("data.csv", delimiter=",")
```

## `einsum` — optional advanced tool

You do not need this for most lab tests, but it can express sums/products over axes compactly.

Dot product:

```python
np.einsum("i,i->", a, b)
```

Matrix multiply:

```python
np.einsum("ik,kj->ij", A, B)
```

Batch dot products:

```python
np.einsum("ij,ij->i", A, B)
```

Learn normal broadcasting and `@` first; treat `einsum` as an advanced convenience.

## `vectorize` warning

```python
f_vec = np.vectorize(f)
```

This is convenient syntax but usually does **not** provide true NumPy-level speed. It often behaves like a Python loop internally.

Prefer genuine array expressions when possible:

```python
# better
out = np.sin(x) + x**2
```

rather than wrapping a scalar Python function unnecessarily.

## `apply_along_axis` warning

```python
np.apply_along_axis(func, axis, A)
```

Useful occasionally, but it is not a replacement for vectorization. If a direct NumPy axis operation exists, prefer it.

## Broadcasting assignment

You can assign one vector across many rows/columns:

```python
A[:, 0] = 0
A[2, :] = 5
A[:, :] = np.arange(A.shape[1])
```

Example last line: the row-shaped vector is broadcast to every row.

## In-place arithmetic

```python
A += 1
A *= 2
A /= 3
```

This mutates `A` when dtype rules permit it.

Compare:

```python
B = A + 1   # new result array
A += 1      # mutates A
```

Be especially conscious of this if another variable/view shares the same data.

## Python unpacking patterns

First + rest:

```python
first, *rest = [10,20,30,40]
```

First/middle/last:

```python
first, *middle, last = [1,2,3,4,5]
```

Ignore a value:

```python
x, _, y = point
```

Unpack dictionary pairs:

```python
for key, value in d.items():
    ...
```

Transpose Python pairs using `zip`:

```python
pairs = [(1,10), (2,20), (3,30)]
xs, ys = zip(*pairs)
```

`xs` and `ys` are tuples. Convert if needed:

```python
xs = list(xs)
```

## `map`, `filter`, and comprehensions

```python
squares = list(map(lambda x: x*x, nums))
positive = list(filter(lambda x: x > 0, nums))
```

Usually comprehensions are more readable:

```python
squares = [x*x for x in nums]
positive = [x for x in nums if x > 0]
```

## Iterators vs materialized collections

These return lazy iterators:

```python
map(...)
filter(...)
zip(...)
reversed(...)
```

To inspect/store all results:

```python
list(zip(a,b))
```

This is why printing `zip(a,b)` itself does not print all pairs.

## Dictionary merge/update

Mutate existing dictionary:

```python
d.update(other)
```

New merged dictionary:

```python
merged = d1 | d2
```

If the same key occurs, the right-hand dictionary wins.

## Set comprehension

```python
squares = {x*x for x in nums}
```

## Generator expressions

```python
total = sum(x*x for x in nums)
```

This avoids constructing a whole temporary list.

## Exception pattern

```python
try:
    x = int(text)
except ValueError:
    print("not an integer")
```

For CP, exceptions are not usually part of the algorithm, but understanding the syntax helps with normal Python programs.

---

# 36. Exercises

Do these **without looking at the solutions file first**.

---

## Part A — Python containers and syntax

### Exercise 1 — List operations

Given:

```python
a = [8, 3, 7, 3, 2, 9, 1]
```

Produce:
1. the first 4 values;
2. the last 3 values;
3. all values at even indices;
4. the reversed list;
5. a sorted copy without changing `a`;
6. a list containing only values greater than 3.

---

### Exercise 2 — Frequency map

Given:

```python
words = ["red", "blue", "red", "green", "blue", "red"]
```

Build a dictionary containing the frequency of every word.

Then print the words sorted:
1. by frequency descending;
2. ties alphabetically ascending.

Expected order begins with `red`.

---

### Exercise 3 — Dictionary of lists

Given:

```python
students = [
    ("CSE", "A"),
    ("EEE", "B"),
    ("CSE", "C"),
    ("ME", "D"),
    ("EEE", "E")
]
```

Build:

```python
{
    "CSE": ["A", "C"],
    "EEE": ["B", "E"],
    "ME": ["D"]
}
```

Then obtain the CSE list safely.

---

### Exercise 4 — Custom tuple sort

Given:

```python
players = [
    ("alice", 120, 5),
    ("bob", 150, 8),
    ("cara", 150, 3),
    ("dan", 120, 2)
]
```

Sort by:
1. score descending;
2. penalty ascending;
3. name alphabetically.

---

### Exercise 5 — Unique while preserving order

Given:

```python
a = [4, 1, 4, 2, 1, 3, 2]
```

Produce:

```python
[4, 1, 2, 3]
```

---

### Exercise 6 — String processing

Given:

```python
s = "  Python,NumPy,Matplotlib  "
```

Produce the list:

```python
["python", "numpy", "matplotlib"]
```

---

### Exercise 7 — Character frequency

Given a lowercase string, find the most frequent character. If tied, choose alphabetically smaller.

---

### Exercise 8 — Matrix with Python lists

Create a `4 x 5` matrix initialized to zero using Python lists. Set:
- row 1, col 2 to 7;
- all values in last row to 9.

Print row 1 and column 2.

---

## Part B — CP/LeetCode patterns

### Exercise 9 — Two Sum

Given `nums` and `target`, return the two indices using a dictionary in O(n).

---

### Exercise 10 — Group Anagrams

Group strings that are anagrams.

Example:

```python
["eat","tea","tan","ate","nat","bat"]
```

---

### Exercise 11 — BFS shortest path

Given an unweighted graph as edge pairs, calculate shortest distance from node `0` to every reachable node.

---

### Exercise 12 — Top K frequent

Given a list of integers and `k`, return the `k` most frequent values.

Try:
1. `Counter + sorted`;
2. `heapq`.

---

### Exercise 13 — Binary-search count

Given a sorted list and number `x`, count how many times `x` occurs using `bisect_left` and `bisect_right`.

---

### Exercise 14 — Prefix sums

Given `a`, build prefix sums so that range sum `[l, r]` can be answered in O(1).

---

### Exercise 15 — Custom object sort

Create a `Student` class with:
- `name`;
- `score`;
- `age`.

Sort a list of students by score descending and age ascending.

---

## Part C — NumPy fundamentals

### Exercise 16 — Array creation and slicing

Create:

```python
a = np.arange(1, 21)
```

Extract:
1. first 5;
2. last 5;
3. every third;
4. reversed;
5. values from index 4 through 12 inclusive.

---

### Exercise 17 — 2D slicing

Create:

```python
A = np.arange(1, 21).reshape(4, 5)
```

Extract:
1. row 2;
2. column 3;
3. first two rows;
4. last two columns;
5. submatrix rows 1..3 and cols 2..4;
6. all rows reversed;
7. all columns reversed.

---

### Exercise 18 — 3D slicing

Create:

```python
X = np.arange(2*3*4).reshape(2,3,4)
```

Extract:
1. first block;
2. row 1 from every block;
3. column 2 from every block;
4. last column from the second block.

Write down the shape of every result before running it.

---

### Exercise 19 — Axis reductions

Given:

```python
A = np.array([
    [1,2,3],
    [4,5,6],
    [7,8,9]
])
```

Calculate:
1. column sums;
2. row sums;
3. column means;
4. index of maximum in each row.

Predict output shapes first.

---

### Exercise 20 — Normalize rows

Given a 2D array `A`, subtract each row's mean from its own row using broadcasting.

No Python loops.

---

### Exercise 21 — Normalize columns

Given a 2D array `A`, subtract each column's mean from its own column.

No Python loops.

---

### Exercise 22 — Pairwise difference matrix

Given:

```python
x = np.array([2,5,9,10])
```

Construct a matrix `D` such that:

```text
D[i,j] = x[i] - x[j]
```

using broadcasting.

---

### Exercise 23 — Boolean masking

Given an array of samples:
1. keep only values in `[-1, 1]`;
2. replace negative values with zero;
3. clip all values into `[0, 1]`.

Do all three separately.

---

### Exercise 24 — Sort 2D rows

Given:

```python
A = np.array([
    [101, 90],
    [102, 75],
    [103, 95],
    [104, 80]
])
```

Sort rows by the second column descending.

---

### Exercise 25 — Top 3 indices

Given a 1D NumPy array, return indices of its three largest values from largest to smallest.

---

### Exercise 26 — Stack and concatenate

Given:

```python
a = np.array([1,2,3])
b = np.array([4,5,6])
```

Find the outputs and shapes of:

```python
np.concatenate([a,b])
np.stack([a,b], axis=0)
np.stack([a,b], axis=1)
np.column_stack([a,b])
```

---

## Part D — Numerical / DSP exercises

### Exercise 27 — Interpolation

Sample:

```python
x = [0, 1, 2, 3, 4]
y = [0, 1, 4, 9, 16]
```

Estimate `y` at:

```python
x_new = [0.5, 1.5, 2.5, 3.5]
```

using `np.interp`.

---

### Exercise 28 — Numerical integration

Approximate:

```text
∫₀^π sin(x) dx
```

with 1000 samples.

Compare against exact result `2`.

---

### Exercise 29 — Numerical derivative

Approximate derivative of:

```text
y = sin(x)
```

and calculate maximum absolute error against `cos(x)`.

Ignore a few endpoint samples when evaluating error if needed.

---

### Exercise 30 — Generate a sampled sinusoid

Create a 2-second signal sampled at 1000 Hz:

```text
x(t) = 2 cos(2π·50t + π/4)
```

Make the time vector with the correct number of samples.

---

### Exercise 31 — FFT peak

For the signal in Exercise 30, use an FFT and identify the positive frequency bin with maximum magnitude.

You should obtain approximately 50 Hz.

---

### Exercise 32 — Two-tone FFT

Generate:

```text
x(t) = 2sin(2π·50t) + 0.5cos(2π·120t)
```

at 1000 Hz for 1 second.

Find the two strongest positive-frequency components.

---

### Exercise 33 — Fourier basis broadcasting

Let:

```python
n = np.arange(-3, 4)
t = np.linspace(0, 1, 1000, endpoint=False)
w0 = 2*np.pi
```

Construct:

```text
E[n_index, t_index] = exp(j n w0 t)
```

without loops.

State its shape.

---

### Exercise 34 — Fourier synthesis

Using Exercise 33, define coefficients:

```python
c = np.array([0,0,0,1,0.5,0,0], dtype=complex)
```

Compute:

```text
x(t) = Σ c_n e^(jnw0t)
```

using broadcasting + `sum(axis=...)`.

---

### Exercise 35 — Moving average

Given a noisy 1D signal `x`, calculate a length-5 moving average using convolution.

---

### Exercise 36 — 2D frequency mask

Create a 2D frequency grid for a `256 x 256` image. Build a Boolean circular low-pass mask with radius 30 frequency-index units.

---

### Exercise 37 — 2D FFT filter

Given a grayscale 2D image array:
1. FFT2;
2. shift;
3. create a circular high-pass mask;
4. multiply;
5. inverse shift;
6. inverse FFT;
7. take real part.

---

### Exercise 38 — Gradient of a 2D function

Let:

```text
f(x,y) = x² + 3y²
```

Create a grid and approximate both partial derivatives numerically.

Compare against:

```text
df/dx = 2x
df/dy = 6y
```

---

## Part E — Copying, views, shape reasoning

### Exercise 39 — Python reference test

Predict the output:

```python
a = [1,2,3]
b = a
c = a.copy()

b[0] = 9
c[1] = 8

print(a)
print(b)
print(c)
```

---

### Exercise 40 — Nested shallow copy

Predict:

```python
a = [[1,2],[3,4]]
b = a.copy()
b[0][0] = 99

print(a)
```

Then fix it using deep copy.

---

### Exercise 41 — NumPy view test

Predict:

```python
a = np.arange(10)
b = a[2:6]
b[:] = -1

print(a)
```

Then modify the code so `a` remains unchanged.

---

### Exercise 42 — Broadcasting shapes

For every pair, decide whether broadcasting works and state output shape:

```text
(3,4) + (4,)
(3,4) + (3,)
(3,4) + (3,1)
(5,1,7) + (1,3,1)
(2,3,4) + (3,4)
(2,3,4) + (2,4)
```

---

### Exercise 43 — Axis prediction

If:

```python
X.shape == (4,5,6)
```

predict shapes of:

```python
X.sum(axis=0)
X.sum(axis=1)
X.sum(axis=2)
X.mean(axis=(1,2))
X.max(axis=1, keepdims=True)
```

---

## Part F — Matplotlib

### Exercise 44 — Signal plot

Plot a 5 Hz sine wave over one second with:
- x label;
- y label;
- title;
- grid.

---

### Exercise 45 — Signal + spectrum

Make one figure containing:
- time-domain signal;
- magnitude spectrum.

Use labeled axes.

---

### Exercise 46 — Image spectrum

For a grayscale image:
- display image;
- display log-magnitude centered 2D FFT;
- include colorbar.

---

# 37. Suggested practice order

Do not try to memorize every function in one sitting.

## Pass 1 — Python survival

Master:
- lists;
- tuples;
- sets;
- dictionaries;
- loops;
- comprehensions;
- sorting;
- strings;
- functions.

Do Exercises 1-8.

## Pass 2 — CP fluency

Master:
- `Counter`;
- `defaultdict`;
- `deque`;
- `heapq`;
- `bisect`;
- custom sorting;
- prefix sums;
- hash-map patterns.

Do Exercises 9-15.

## Pass 3 — NumPy indexing/shape fluency

Master:
- `shape`;
- slices;
- rows/columns;
- 3D indexing;
- `reshape`;
- `axis`;
- broadcasting;
- masks;
- `argsort`;
- stack/concatenate.

Do Exercises 16-26.

## Pass 4 — DSP fluency

Master:
- `linspace`;
- `interp`;
- `trapezoid`;
- `gradient`;
- complex arrays;
- FFT;
- frequency bins;
- `fftshift`;
- meshgrid;
- convolution.

Do Exercises 27-38.

## Pass 5 — eliminate dangerous bugs

Master:
- references;
- shallow/deep copy;
- NumPy views;
- singleton dimensions;
- broadcasting shape reasoning.

Do Exercises 39-43.

## Pass 6 — plotting

Do Exercises 44-46.

---

# Final compact checklist

Before a Python/NumPy lab or contest, you should be able to write these without searching:

```python
# Python
list(...)
set(...)
dict(...)
tuple(...)

a.append(x)
a.pop()
a.sort(key=lambda x: ...)
sorted(a, key=lambda x: ..., reverse=True)

for i, x in enumerate(a):
    ...

for k, v in d.items():
    ...

from collections import Counter, defaultdict, deque
import heapq
from bisect import bisect_left, bisect_right

# NumPy
import numpy as np

a = np.array(...)
A = np.arange(...).reshape(...)
t = np.linspace(...)

A[r, :]
A[:, c]
A[r1:r2, c1:c2]
A[::-1]
A[:, ::-1]

A.mean(axis=0)
A.mean(axis=1)

a[:, None]
a[None, :]

A[A > 0]
np.where(...)
np.argsort(...)
np.concatenate(...)
np.stack(...)

np.interp(...)
np.trapezoid(...)
np.gradient(...)

np.exp(1j * theta)
np.abs(z)
np.angle(z)

np.fft.fft(...)
np.fft.fftfreq(...)
np.fft.fftshift(...)
np.fft.fft2(...)

# Matplotlib
import matplotlib.pyplot as plt

plt.plot(...)
plt.stem(...)
plt.imshow(...)
plt.xlabel(...)
plt.ylabel(...)
plt.title(...)
plt.grid(True)
plt.legend()
plt.show()
```

If these patterns become automatic, you can spend your lab/contest time on the actual algorithm, mathematics, or DSP logic instead of syntax.
