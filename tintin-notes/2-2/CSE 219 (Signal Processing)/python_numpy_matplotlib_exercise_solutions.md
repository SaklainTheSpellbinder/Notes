# Solutions — Python + NumPy + Matplotlib Master Guide

Use this file only after attempting the exercises.

---

# Exercise 1 — List operations

```python
a = [8, 3, 7, 3, 2, 9, 1]

print(a[:4])
print(a[-3:])
print(a[::2])
print(a[::-1])
print(sorted(a))
print([x for x in a if x > 3])

print("original:", a)
```

---

# Exercise 2 — Frequency map

```python
from collections import Counter

words = ["red", "blue", "red", "green", "blue", "red"]

freq = Counter(words)

ordered = sorted(freq.items(), key=lambda kv: (-kv[1], kv[0]))

print(freq)
print(ordered)
```

Manual version:

```python
freq = {}

for word in words:
    freq[word] = freq.get(word, 0) + 1
```

---

# Exercise 3 — Dictionary of lists

```python
from collections import defaultdict

students = [
    ("CSE", "A"),
    ("EEE", "B"),
    ("CSE", "C"),
    ("ME", "D"),
    ("EEE", "E")
]

groups = defaultdict(list)

for department, name in students:
    groups[department].append(name)

print(dict(groups))

cse = groups.get("CSE", [])
print(cse)
```

---

# Exercise 4 — Custom tuple sort

```python
players = [
    ("alice", 120, 5),
    ("bob", 150, 8),
    ("cara", 150, 3),
    ("dan", 120, 2)
]

players.sort(key=lambda p: (-p[1], p[2], p[0]))

print(players)
```

---

# Exercise 5 — Unique while preserving order

```python
a = [4, 1, 4, 2, 1, 3, 2]

result = list(dict.fromkeys(a))
print(result)
```

Alternative:

```python
seen = set()
result = []

for x in a:
    if x not in seen:
        seen.add(x)
        result.append(x)
```

---

# Exercise 6 — String processing

```python
s = "  Python,NumPy,Matplotlib  "

parts = [x.lower() for x in s.strip().split(",")]

print(parts)
```

---

# Exercise 7 — Character frequency

```python
from collections import Counter

s = "abracadabra"

cnt = Counter(s)

answer = min(cnt.items(), key=lambda kv: (-kv[1], kv[0]))[0]

print(answer)
```

Another readable version:

```python
items = sorted(cnt.items(), key=lambda kv: (-kv[1], kv[0]))
answer = items[0][0]
```

---

# Exercise 8 — Matrix with Python lists

```python
n, m = 4, 5

A = [[0] * m for _ in range(n)]

A[1][2] = 7
A[-1] = [9] * m

print("row 1:", A[1])
print("column 2:", [row[2] for row in A])
```

---

# Exercise 9 — Two Sum

```python
def two_sum(nums, target):
    seen = {}

    for i, x in enumerate(nums):
        need = target - x

        if need in seen:
            return seen[need], i

        seen[x] = i

    return None
```

---

# Exercise 10 — Group Anagrams

```python
from collections import defaultdict

def group_anagrams(words):
    groups = defaultdict(list)

    for word in words:
        key = "".join(sorted(word))
        groups[key].append(word)

    return list(groups.values())


words = ["eat","tea","tan","ate","nat","bat"]
print(group_anagrams(words))
```

For lowercase English letters, a frequency-tuple key avoids sorting each word:

```python
def group_anagrams(words):
    groups = defaultdict(list)

    for word in words:
        cnt = [0] * 26

        for c in word:
            cnt[ord(c) - ord('a')] += 1

        groups[tuple(cnt)].append(word)

    return list(groups.values())
```

---

# Exercise 11 — BFS shortest path

```python
from collections import defaultdict, deque

def shortest_distances(n, edges):
    graph = defaultdict(list)

    for u, v in edges:
        graph[u].append(v)
        graph[v].append(u)

    dist = [-1] * n
    dist[0] = 0

    q = deque([0])

    while q:
        u = q.popleft()

        for v in graph[u]:
            if dist[v] == -1:
                dist[v] = dist[u] + 1
                q.append(v)

    return dist
```

---

# Exercise 12 — Top K frequent

## Counter + sorted

```python
from collections import Counter

def top_k(nums, k):
    cnt = Counter(nums)

    items = sorted(
        cnt.items(),
        key=lambda kv: (-kv[1], kv[0])
    )

    return [value for value, freq in items[:k]]
```

## Heap

```python
import heapq
from collections import Counter

def top_k_heap(nums, k):
    cnt = Counter(nums)
    heap = []

    for value, freq in cnt.items():
        heapq.heappush(heap, (-freq, value))

    return [heapq.heappop(heap)[1] for _ in range(k)]
```

---

# Exercise 13 — Binary-search count

```python
from bisect import bisect_left, bisect_right

def count_x(a, x):
    return bisect_right(a, x) - bisect_left(a, x)
```

---

# Exercise 14 — Prefix sums

```python
def build_prefix(a):
    prefix = [0]

    for x in a:
        prefix.append(prefix[-1] + x)

    return prefix

def range_sum(prefix, l, r):
    return prefix[r + 1] - prefix[l]
```

Example:

```python
a = [5,2,7,1]
p = build_prefix(a)

print(range_sum(p, 1, 3))
```

---

# Exercise 15 — Custom object sort

```python
class Student:
    def __init__(self, name, score, age):
        self.name = name
        self.score = score
        self.age = age

    def __repr__(self):
        return f"Student({self.name!r}, {self.score}, {self.age})"


students = [
    Student("Alice", 90, 20),
    Student("Bob", 95, 22),
    Student("Cara", 95, 19),
]

students.sort(key=lambda s: (-s.score, s.age, s.name))

print(students)
```

---

# Exercise 16 — Array creation and slicing

```python
import numpy as np

a = np.arange(1, 21)

print(a[:5])
print(a[-5:])
print(a[::3])
print(a[::-1])
print(a[4:13])
```

---

# Exercise 17 — 2D slicing

```python
import numpy as np

A = np.arange(1, 21).reshape(4, 5)

print("row 2:", A[2, :])
print("column 3:", A[:, 3])
print("first two rows:\n", A[:2, :])
print("last two columns:\n", A[:, -2:])
print("submatrix:\n", A[1:4, 2:5])
print("rows reversed:\n", A[::-1, :])
print("columns reversed:\n", A[:, ::-1])
```

---

# Exercise 18 — 3D slicing

```python
import numpy as np

X = np.arange(2*3*4).reshape(2,3,4)

a = X[0]
b = X[:, 1, :]
c = X[:, :, 2]
d = X[1, :, -1]

print(a, a.shape)
print(b, b.shape)
print(c, c.shape)
print(d, d.shape)
```

Shapes:

```text
X[0]       -> (3,4)
X[:,1,:]   -> (2,4)
X[:,:,2]   -> (2,3)
X[1,:,-1]  -> (3,)
```

---

# Exercise 19 — Axis reductions

```python
import numpy as np

A = np.array([
    [1,2,3],
    [4,5,6],
    [7,8,9]
])

column_sums = A.sum(axis=0)
row_sums = A.sum(axis=1)
column_means = A.mean(axis=0)
row_argmax = A.argmax(axis=1)

print(column_sums)
print(row_sums)
print(column_means)
print(row_argmax)
```

Shapes:

```text
(3,)
(3,)
(3,)
(3,)
```

---

# Exercise 20 — Normalize rows

```python
row_means = A.mean(axis=1, keepdims=True)
B = A - row_means
```

Equivalent:

```python
B = A - A.mean(axis=1)[:, None]
```

---

# Exercise 21 — Normalize columns

```python
column_means = A.mean(axis=0, keepdims=True)
B = A - column_means
```

Since `A.mean(axis=0)` has shape `(cols,)`, this also works:

```python
B = A - A.mean(axis=0)
```

---

# Exercise 22 — Pairwise difference matrix

```python
import numpy as np

x = np.array([2,5,9,10])

D = x[:, None] - x[None, :]

print(D)
```

---

# Exercise 23 — Boolean masking

```python
import numpy as np

a = np.array([-2.0, -0.5, 0.2, 0.9, 1.5, 3.0])

inside = a[(a >= -1) & (a <= 1)]

nonnegative = a.copy()
nonnegative[nonnegative < 0] = 0

clipped = np.clip(a, 0, 1)

print(inside)
print(nonnegative)
print(clipped)
```

---

# Exercise 24 — Sort 2D rows

```python
import numpy as np

A = np.array([
    [101, 90],
    [102, 75],
    [103, 95],
    [104, 80]
])

idx = np.argsort(A[:, 1])[::-1]
A_sorted = A[idx]

print(A_sorted)
```

---

# Exercise 25 — Top 3 indices

```python
idx = np.argsort(a)[-3:][::-1]
```

Faster large-array version:

```python
idx = np.argpartition(a, -3)[-3:]
idx = idx[np.argsort(a[idx])[::-1]]
```

---

# Exercise 26 — Stack and concatenate

```python
import numpy as np

a = np.array([1,2,3])
b = np.array([4,5,6])

x1 = np.concatenate([a,b])
x2 = np.stack([a,b], axis=0)
x3 = np.stack([a,b], axis=1)
x4 = np.column_stack([a,b])

print(x1, x1.shape)
print(x2, x2.shape)
print(x3, x3.shape)
print(x4, x4.shape)
```

Shapes:

```text
concatenate -> (6,)
stack axis0 -> (2,3)
stack axis1 -> (3,2)
column_stack -> (3,2)
```

---

# Exercise 27 — Interpolation

```python
import numpy as np

x = np.array([0,1,2,3,4], dtype=float)
y = np.array([0,1,4,9,16], dtype=float)

x_new = np.array([0.5,1.5,2.5,3.5])

y_new = np.interp(x_new, x, y)

print(y_new)
```

Note that this is piecewise-linear interpolation, not exact evaluation of `x²`.

---

# Exercise 28 — Numerical integration

```python
import numpy as np

x = np.linspace(0, np.pi, 1000)
y = np.sin(x)

area = np.trapezoid(y, x)

print("approx:", area)
print("exact:", 2.0)
print("error:", abs(area - 2.0))
```

---

# Exercise 29 — Numerical derivative

```python
import numpy as np

x = np.linspace(0, 2*np.pi, 1000)
y = np.sin(x)

dy = np.gradient(y, x)
exact = np.cos(x)

error = np.max(np.abs(dy[2:-2] - exact[2:-2]))

print(error)
```

---

# Exercise 30 — Generate a sampled sinusoid

```python
import numpy as np

fs = 1000
duration = 2.0

N = int(fs * duration)

t = np.arange(N) / fs

x = 2 * np.cos(2*np.pi*50*t + np.pi/4)

print(t.shape)
print(x.shape)
```

Equivalent:

```python
t = np.linspace(0, duration, N, endpoint=False)
```

---

# Exercise 31 — FFT peak

```python
import numpy as np

fs = 1000
duration = 2
N = int(fs * duration)

t = np.arange(N) / fs
x = 2*np.cos(2*np.pi*50*t + np.pi/4)

X = np.fft.rfft(x)
f = np.fft.rfftfreq(N, d=1/fs)

idx = np.argmax(np.abs(X))

print("peak frequency:", f[idx])
```

---

# Exercise 32 — Two-tone FFT

```python
import numpy as np

fs = 1000
duration = 1
N = int(fs * duration)

t = np.arange(N) / fs

x = (
    2*np.sin(2*np.pi*50*t)
    + 0.5*np.cos(2*np.pi*120*t)
)

X = np.fft.rfft(x)
f = np.fft.rfftfreq(N, d=1/fs)

mag = np.abs(X)

idx = np.argsort(mag[1:])[-2:] + 1
idx = idx[np.argsort(mag[idx])[::-1]]

print(f[idx])
print(mag[idx])
```

Expected frequencies approximately:

```text
50 Hz
120 Hz
```

---

# Exercise 33 — Fourier basis broadcasting

```python
import numpy as np

n = np.arange(-3, 4)
t = np.linspace(0, 1, 1000, endpoint=False)
w0 = 2*np.pi

E = np.exp(1j * n[:, None] * w0 * t[None, :])

print(E.shape)
```

Shape:

```text
(7, 1000)
```

---

# Exercise 34 — Fourier synthesis

```python
import numpy as np

n = np.arange(-3, 4)
t = np.linspace(0, 1, 1000, endpoint=False)
w0 = 2*np.pi

c = np.array([0,0,0,1,0.5,0,0], dtype=complex)

E = np.exp(1j * n[:, None] * w0 * t[None, :])

x = np.sum(c[:, None] * E, axis=0)

print(x.shape)
```

---

# Exercise 35 — Moving average

```python
import numpy as np

M = 5
kernel = np.ones(M) / M

smooth = np.convolve(x, kernel, mode="same")
```

---

# Exercise 36 — 2D frequency mask

Index-unit version:

```python
import numpy as np

N = 256

u = np.arange(-N//2, N//2)
v = np.arange(-N//2, N//2)

U, V = np.meshgrid(u, v)

R = np.sqrt(U**2 + V**2)

mask = R <= 30

print(mask.shape)
```

---

# Exercise 37 — 2D FFT filter

```python
import numpy as np

def high_pass_filter(image, cutoff):
    rows, cols = image.shape

    F = np.fft.fft2(image)
    Fc = np.fft.fftshift(F)

    u = np.arange(-cols//2, cols - cols//2)
    v = np.arange(-rows//2, rows - rows//2)

    U, V = np.meshgrid(u, v)
    R = np.sqrt(U**2 + V**2)

    mask = R >= cutoff

    F_filtered = Fc * mask

    F_unshifted = np.fft.ifftshift(F_filtered)
    out = np.fft.ifft2(F_unshifted).real

    return out
```

---

# Exercise 38 — Gradient of a 2D function

```python
import numpy as np

x = np.linspace(-2, 2, 200)
y = np.linspace(-2, 2, 150)

X, Y = np.meshgrid(x, y)

F = X**2 + 3*Y**2

dF_dy, dF_dx = np.gradient(F, y, x)

exact_dx = 2*X
exact_dy = 6*Y

err_x = np.max(np.abs(dF_dx[2:-2, 2:-2] - exact_dx[2:-2, 2:-2]))
err_y = np.max(np.abs(dF_dy[2:-2, 2:-2] - exact_dy[2:-2, 2:-2]))

print(err_x, err_y)
```

---

# Exercise 39 — Python reference test

Code:

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

Output:

```text
[9, 2, 3]
[9, 2, 3]
[1, 8, 3]
```

Why:
- `b is a`;
- `c` is a different outer list.

---

# Exercise 40 — Nested shallow copy

```python
a = [[1,2],[3,4]]
b = a.copy()
b[0][0] = 99

print(a)
```

Output:

```text
[[99, 2], [3, 4]]
```

Deep-copy fix:

```python
import copy

a = [[1,2],[3,4]]
b = copy.deepcopy(a)

b[0][0] = 99

print(a)
# [[1,2],[3,4]]
```

---

# Exercise 41 — NumPy view test

```python
import numpy as np

a = np.arange(10)
b = a[2:6]

b[:] = -1

print(a)
```

Output:

```text
[0, 1, -1, -1, -1, -1, 6, 7, 8, 9]
```

Independent copy:

```python
a = np.arange(10)
b = a[2:6].copy()

b[:] = -1

print(a)
```

---

# Exercise 42 — Broadcasting shapes

```text
(3,4) + (4,)      -> works -> (3,4)

(3,4) + (3,)      -> fails
rightmost dimensions 4 and 3 conflict

(3,4) + (3,1)     -> works -> (3,4)

(5,1,7) + (1,3,1) -> works -> (5,3,7)

(2,3,4) + (3,4)   -> works -> (2,3,4)

(2,3,4) + (2,4)   -> fails
align right:
(2,3,4)
  (2,4)
middle comparison is 3 vs 2
```

---

# Exercise 43 — Axis prediction

If:

```python
X.shape == (4,5,6)
```

then:

```python
X.sum(axis=0).shape
# (5,6)

X.sum(axis=1).shape
# (4,6)

X.sum(axis=2).shape
# (4,5)

X.mean(axis=(1,2)).shape
# (4,)

X.max(axis=1, keepdims=True).shape
# (4,1,6)
```

---

# Exercise 44 — Signal plot

```python
import numpy as np
import matplotlib.pyplot as plt

t = np.linspace(0, 1, 1000, endpoint=False)
x = np.sin(2*np.pi*5*t)

plt.plot(t, x)
plt.xlabel("Time (s)")
plt.ylabel("Amplitude")
plt.title("5 Hz sine wave")
plt.grid(True)
plt.show()
```

---

# Exercise 45 — Signal + spectrum

```python
import numpy as np
import matplotlib.pyplot as plt

fs = 1000
N = 1000

t = np.arange(N) / fs
x = np.sin(2*np.pi*50*t) + 0.5*np.sin(2*np.pi*120*t)

X = np.fft.rfft(x)
f = np.fft.rfftfreq(N, d=1/fs)

fig, ax = plt.subplots(2, 1)

ax[0].plot(t, x)
ax[0].set_xlabel("Time (s)")
ax[0].set_ylabel("Amplitude")
ax[0].set_title("Signal")
ax[0].grid(True)

ax[1].plot(f, np.abs(X))
ax[1].set_xlabel("Frequency (Hz)")
ax[1].set_ylabel("Magnitude")
ax[1].set_title("Magnitude spectrum")
ax[1].grid(True)

plt.tight_layout()
plt.show()
```

---

# Exercise 46 — Image spectrum

```python
import numpy as np
import matplotlib.pyplot as plt

# image: 2D grayscale NumPy array

F = np.fft.fftshift(np.fft.fft2(image))
S = np.log1p(np.abs(F))

fig, ax = plt.subplots(1, 2)

ax[0].imshow(image, cmap="gray")
ax[0].set_title("Image")

im = ax[1].imshow(S, cmap="gray", origin="lower")
ax[1].set_title("Log-magnitude spectrum")

fig.colorbar(im, ax=ax[1])

plt.tight_layout()
plt.show()
```

---

# What to do after finishing these exercises

Repeat the exercises later without the guide and see whether you can write the important patterns from memory:

```python
# dictionary frequency
freq[x] = freq.get(x, 0) + 1

# custom sort
items.sort(key=lambda x: (-x[1], x[0]))

# enumerate
for i, x in enumerate(a):
    ...

# dictionary iteration
for k, v in d.items():
    ...

# NumPy row / column
A[r, :]
A[:, c]

# broadcast outer operation
a[:, None] * b[None, :]

# row normalization
A - A.mean(axis=1, keepdims=True)

# masks
A[(A >= lo) & (A <= hi)]

# sort indices
idx = np.argsort(a)

# interpolation
np.interp(x_new, x, y)

# integration
np.trapezoid(y, x)

# derivative
np.gradient(y, x)

# FFT
X = np.fft.fft(x)
f = np.fft.fftfreq(len(x), d=1/fs)

# Fourier basis
E = np.exp(1j*n[:,None]*w0*t[None,:])
```

When these become automatic, Python/NumPy syntax stops being the limiting factor.
