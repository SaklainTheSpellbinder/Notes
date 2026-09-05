     
Assume:

```python
import numpy as np

from transforms import FFTTransformer, next_power_of_two
from image_conv import transform_2d, inverse_2d

engine = FFTTransformer()
```

---

# Two helpers that solve half the questions

## Helper 1 — linear convolution using your FFT

```python
def fft_convolve(a, b, engine):
    a = np.asarray(a, dtype=complex)
    b = np.asarray(b, dtype=complex)

    L = len(a) + len(b) - 1
    N = next_power_of_two(L)

    pa = np.zeros(N, dtype=complex)
    pb = np.zeros(N, dtype=complex)

    pa[:len(a)] = a
    pb[:len(b)] = b

    A = engine.transform(pa)
    B = engine.transform(pb)

    c = engine.inverse(A * B)

    return c.real[:L]
```

Mental formula:

$$
\boxed{
a*b=IFFT(FFT(a)\cdot FFT(b))
}
$$

with enough padding.

---

## Helper 2 — detect circular shift

Convention:

```python
shifted = np.roll(reference, s)
```

Then this finds \(s\):

```python
def detect_shift(reference, shifted, engine):
    reference = np.asarray(reference, dtype=complex)
    shifted = np.asarray(shifted, dtype=complex)

    X = engine.transform(reference)
    Y = engine.transform(shifted)

    corr = engine.inverse(np.conj(X) * Y).real

    shift = int(np.argmax(corr))

    return shift
```

Undo it:

```python
corrected = np.roll(shifted, -shift)
```

This is your main cross-correlation template.

---

# PYQ 1 — Large integers already given as LSD-first digit arrays

The question gives something like:

```python
A = [3, 2, 1]      # 123
B = [5, 4]         # 45
```

and asks for LSD-first result. 

```python
def multiply_digit_arrays(A, B):
    A = np.asarray(A, dtype=np.int64)
    B = np.asarray(B, dtype=np.int64)

    # FFT convolution
    raw = fft_convolve(A, B, FFTTransformer())

    # Mathematical result should be integer
    coeff = np.rint(raw).astype(np.int64).tolist()

    # Carry propagation
    carry = 0

    for i in range(len(coeff)):
        total = coeff[i] + carry

        coeff[i] = total % 10
        carry = total // 10

    while carry > 0:
        coeff.append(carry % 10)
        carry //= 10

    # Remove useless highest zeros
    while len(coeff) > 1 and coeff[-1] == 0:
        coeff.pop()

    return np.array(coeff, dtype=np.int64)
```

Example concept:

```python
A = [3, 2, 1]
B = [5, 4]

result = multiply_digit_arrays(A, B)
```

Result:

```text
[5, 3, 5, 5]
```

representing \(5535\).

The source explicitly requires the convolution followed by base-10 carry propagation. 

---

# PYQ 2 — Weighted polynomial multiplication

Question:

$$
R[k]=\sum_i w_i p_iq_{k-i}.
$$

The key observation:

$$
a_i=w_ip_i
$$

so:

$$
\boxed{R=a*q}.
$$

The question gives polynomial coefficients in **descending powers**. 

```python
def weighted_polynomial(P, Q, W):
    P = np.asarray(P)
    Q = np.asarray(Q)
    W = np.asarray(W)

    # Input is descending powers.
    # Convert to ascending powers.
    p = P[::-1]
    q = Q[::-1]
    w = W[::-1]

    # wi * pi
    weighted_p = w * p

    # convolution
    result = fft_convolve(
        weighted_p,
        q,
        FFTTransformer()
    )

    result = np.rint(result).astype(np.int64)

    # Convert back to descending powers
    return result[::-1]
```

For the paper's:

```python
P = [1, 3, 2]
Q = [4, 1]
W = [3, 2, 1]
```

the output order is:

```text
[12, 27, 14, 2]
```

matching its descending-power convention. 

The one line to remember:

```python
weighted_p = P[::-1] * W[::-1]
```

then convolve with:

```python
Q[::-1]
```

---

# PYQ 3 — Huge numbers provided as normal decimal strings

This is the older alien multiplication problem. It asks you to convert digits, FFT-convolve, carry, and rebuild the number. 

```python
def multiply_strings(A, B):
    # "123" -> [3, 2, 1]
    a = np.array(
        [int(ch) for ch in A[::-1]],
        dtype=np.int64
    )

    b = np.array(
        [int(ch) for ch in B[::-1]],
        dtype=np.int64
    )

    digits = multiply_digit_arrays(a, b)

    # LSD-first -> normal decimal order
    answer = ''.join(
        str(d) for d in digits[::-1]
    )

    return answer
```

Example:

```python
multiply_strings("123", "456")
```

Conceptually:

```text
"123"
   ↓
[3,2,1]

"456"
   ↓
[6,5,4]

FFT convolution
   ↓
carry
   ↓
[8,8,0,6,5]
   ↓
"56088"
```

The older paper specifically says the IFFT result should be rounded to the nearest integer before carrying. 

---

# PYQ 4 — Rolling shutter: every image row shifted differently

You have:

```text
original image
shifted image
```

Every row has its own horizontal circular shift. No vertical shift. 

Assume grayscale first:

```python
def fix_rolling_shutter(original, shifted):
    original = np.asarray(original)
    shifted = np.asarray(shifted)

    H, W = original.shape

    engine = FFTTransformer()

    corrected = np.zeros_like(shifted)
    shifts = np.zeros(H, dtype=int)

    for r in range(H):

        ref = original[r, :]
        cur = shifted[r, :]

        shift = detect_shift(ref, cur, engine)

        shifts[r] = shift

        corrected[r, :] = np.roll(
            cur,
            -shift
        )

    return corrected, shifts
```

The entire algorithm is just:

```python
for every row:
    correlation
    argmax
    roll backwards
```

---

## RGB variation

Detect using one channel:

```python
def fix_rolling_shutter_rgb(original, shifted):
    H, W, C = original.shape

    engine = FFTTransformer()

    corrected = np.zeros_like(shifted)
    shifts = np.zeros(H, dtype=int)

    for r in range(H):

        # Use red channel to detect shift
        ref = original[r, :, 0]
        cur = shifted[r, :, 0]

        s = detect_shift(ref, cur, engine)

        shifts[r] = s

        # shifted[r,:,:] shape = (W, 3)
        corrected[r, :, :] = np.roll(
            shifted[r, :, :],
            -s,
            axis=0
        )

    return corrected, shifts
```

Why `axis=0`?

Because:

```python
shifted[r, :, :]
```

has shape:

$$
(W,3).
$$

So axis 0 is horizontal pixel position.

---

# PYQ 5 — BUET logo encryption by circular convolution with a key row

Encryption:

$$
encrypted_r
=
original_r\circledast key.
$$

Therefore:

$$
E_r=X_rK.
$$

So:

$$
\boxed{
X_r=\frac{E_r}{K}
}
$$

and then IFFT.

The paper says the key row itself is left untouched and can be identified because its values are much smaller. 

```python
def decrypt_image(encrypted):
    encrypted = np.asarray(encrypted, dtype=float)

    H, W = encrypted.shape

    engine = FFTTransformer()

    # Paper hint: find smallest entry of a column
    key_index = int(
        np.argmin(encrypted[:, 0])
    )

    key = encrypted[key_index, :]

    # Compute key FFT ONCE
    K = engine.transform(key)

    restored = np.zeros_like(
        encrypted,
        dtype=float
    )

    for r in range(H):

        # key row wasn't encrypted
        if r == key_index:
            restored[r, :] = key
            continue

        E = engine.transform(
            encrypted[r, :]
        )

        # Deconvolution
        X = E / K

        restored[r, :] = engine.inverse(X).real

    return restored, key_index
```

The paper's hint is literally:

$$
DFT(x\circledast key)=X\cdot KEY.
$$



---

## If they make this problem harder

If some \(K[k]\) are near zero, direct division is dangerous.

Use:

```python
eps = 1e-10

X = (
    E * np.conj(K)
    / (np.abs(K)**2 + eps)
)
```

This is a regularized inverse.

For the exact PYQ, plain:

```python
E / K
```

is the intended idea.

---

# PYQ 6 — Entire image shifted horizontally + vertically

The older Section C says one complete image is circularly shifted horizontally and vertically. Detect both shifts and reverse them. 

Since you now have `transform_2d`, the clean general version is:

```python
def align_2d(original, shifted):
    original = np.asarray(original, dtype=float)
    shifted = np.asarray(shifted, dtype=float)

    engine = FFTTransformer()

    F1 = transform_2d(original, engine)
    F2 = transform_2d(shifted, engine)

    corr = inverse_2d(
        np.conj(F1) * F2,
        engine
    ).real

    # Location of strongest 2D correlation
    flat_index = np.argmax(corr)

    dy, dx = np.unravel_index(
        flat_index,
        corr.shape
    )

    H, W = original.shape

    # Convert circular indices to signed shifts
    if dy > H // 2:
        dy -= H

    if dx > W // 2:
        dx -= W

    corrected = np.roll(
        shifted,
        shift=(-dy, -dx),
        axis=(0, 1)
    )

    return corrected, dy, dx
```

This is the most general solution.

---

# What is `np.unravel_index`?

Suppose:

```python
A.shape == (4, 5)
```

and:

```python
np.argmax(A)
```

returns:

```text
13
```

That's the index if the matrix were flattened:

```text
0 1 2 3 4
5 6 7 8 9
10 11 12 13 14
...
```

To convert 13 back into row/column:

```python
r, c = np.unravel_index(
    13,
    A.shape
)
```

gives:

```text
r = 2
c = 3
```

Very useful for 2D correlation.

---

# If examiner specifically asks you to use 1D correlations only

A neat solution is to form row and column signatures:

```python
def align_using_1d(original, shifted):
    engine = FFTTransformer()

    # One value per row
    orig_rows = original.sum(axis=1)
    shift_rows = shifted.sum(axis=1)

    dy = detect_shift(
        orig_rows,
        shift_rows,
        engine
    )

    # One value per column
    orig_cols = original.sum(axis=0)
    shift_cols = shifted.sum(axis=0)

    dx = detect_shift(
        orig_cols,
        shift_cols,
        engine
    )

    corrected = np.roll(
        shifted,
        shift=(-dy, -dx),
        axis=(0, 1)
    )

    return corrected, dy, dx
```

Why this works:

horizontal shifting does not change the total of each row, while vertical shifting merely moves those row totals around. Similarly for column totals.

The exact paper recommends deriving horizontal and vertical shifts from carefully chosen rows and columns; this projection version is a robust extension of that same 1D-correlation idea. 

---

# Your six-question cheat sheet

| Problem                    | Core code                                            |
| -------------------------- | ---------------------------------------------------- |
| Large integer digit arrays | `fft_convolve → rint → %10 //10`                     |
| Weighted polynomial        | `(P[::-1] * W[::-1])` convolve `Q[::-1]`             |
| Huge number strings        | string → reversed digits → previous solution         |
| Different shift per row    | `IFFT(conj(X)*Y) → argmax → roll` per row            |
| Convolution-encrypted rows | `FFT(row) / FFT(key) → IFFT`                         |
| Whole-image shift          | 2D correlation → `unravel_index(argmax)` → `np.roll` |

And the few lines I would absolutely memorize are:

```python
# convolution
c = engine.inverse(
    engine.transform(a) *
    engine.transform(b)
).real
```

```python
# correlation
corr = engine.inverse(
    np.conj(engine.transform(a)) *
    engine.transform(b)
).real

shift = np.argmax(corr)
```

```python
# undo shift
fixed = np.roll(shifted, -shift)
```

```python
# deconvolution
X = engine.transform(encrypted) / engine.transform(key)
original = engine.inverse(X).real
```

```python
# reverse
x[::-1]
```

```python
# largest / smallest index
np.argmax(x)
np.argmin(x)
```

```python
# row / column
A[r, :]
A[:, c]
```

```python
# 2D peak
r, c = np.unravel_index(
    np.argmax(A),
    A.shape
)
```

If those patterns are automatic, the scenario wording can change substantially and you'll still have a clear path to the solution.
