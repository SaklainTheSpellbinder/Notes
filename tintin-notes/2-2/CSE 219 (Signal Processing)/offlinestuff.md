Yes. For the online, you do **not** need to memorize every assignment function equally. Some are core reusable tools; some are application-specific; some are basically useless during an online.

The most important distinction is:

$$
\boxed{\text{transform functions}} \rightarrow \text{your general Fourier toolbox}
$$

while `bigmul.py` and `image_conv.py` functions are higher-level shortcuts built on top of that.

## 1. `transforms.py` — these are your most important online tools

Your template defines `next_power_of_two`, `DFTAnalyzer`, `FFTTransformer`, and the optional arbitrary-length FFT.  

| Function                  | What it does               | When to use in online                                      |
| ------------------------- | -------------------------- | ---------------------------------------------------------- |
| `next_power_of_two(n)`    | Smallest \(2^k\ge n\)      | Whenever **linear convolution** needs FFT padding          |
| `FFTTransformer()`        | Creates your FFT engine    | Almost every online requiring FFT                          |
| `engine.transform(x)`     | 1D FFT                     | Convolution, correlation, deconvolution, spectrum analysis |
| `engine.inverse(X)`       | 1D IFFT                    | Return to time/space domain                                |
| `DFTAnalyzer()`           | Naive \(O(N^2)\) DFT       | Only if question explicitly asks DFT/direct comparison     |
| `DFTAnalyzer.transform()` | Direct DFT                 | Small signals / comparison                                 |
| `DFTAnalyzer.inverse()`   | Direct IDFT                | Small signals / comparison                                 |
| `ArbitraryLengthFFT`      | Bonus arbitrary-length FFT | Ignore unless you implemented bonus                        |

The online default should be:

```python
engine = FFTTransformer()
```

Then almost everything starts from:

```python
X = engine.transform(x)
```

and ends with:

```python
x = engine.inverse(X)
```

---

# 2. `next_power_of_two()` — when exactly do I need it?

Use it when the desired operation is **linear convolution**.

Suppose:

```python
a = [ ... ]     # length n
b = [ ... ]     # length m
```

Linear convolution needs:

$$
L=n+m-1.
$$

Your radix-2 FFT needs a power-of-two length:

```python
L = len(a) + len(b) - 1
N = next_power_of_two(L)
```

Then:

```python
pa = np.zeros(N, dtype=complex)
pb = np.zeros(N, dtype=complex)

pa[:len(a)] = a
pb[:len(b)] = b
```

### Use it for

* integer multiplication;
* polynomial multiplication;
* weighted polynomial multiplication;
* ordinary linear signal convolution;
* image linear convolution.

### Do NOT automatically use it for

**circular convolution** when the desired period is already \(N\).

For example, if a question says:

> signal `y` was circularly convolved with a key of the same length

then you generally want the same length, not padding intended to remove circular behavior.

---

# 3. `FFTTransformer.transform(x)`

This is probably the single most useful function.

```python
X = engine.transform(x)
```

means:

$$
x[n]\rightarrow X[k].
$$

Use it whenever you see:

### Convolution

```python
A = engine.transform(a)
B = engine.transform(b)

C = A * B
```

because:

$$
a\circledast b
\leftrightarrow AB.
$$

### Cross-correlation

```python
A = engine.transform(reference)
B = engine.transform(shifted)

R = np.conj(A) * B
```

### Deconvolution

```python
Y = engine.transform(encrypted)
H = engine.transform(key)

X = Y / H
```

### Frequency analysis

```python
X = engine.transform(signal)

mag = np.abs(X)
phase = np.angle(X)
```

Find strongest frequency:

```python
k = np.argmax(np.abs(X))
```

So mentally:

> **If I need to manipulate something using a DFT property, first call `.transform()`.**

---

# 4. `FFTTransformer.inverse(X)`

Use this after you have done whatever operation you needed in frequency domain.

### Convolution

```python
c = engine.inverse(A * B).real
```

### Correlation

```python
corr = engine.inverse(
    np.conj(A) * B
).real
```

### Deconvolution

```python
restored = engine.inverse(
    Y / H
).real
```

### Spectrum editing/filtering

```python
X[some_bins] = 0

filtered = engine.inverse(X).real
```

The important distinction:

```python
engine.inverse(...)
```

returns complex values.

Only use:

```python
.real
```

when mathematically you know your final signal/image should be real.

---

# 5. `multiply_transform(a, b, engine)`

This one from Task A is **very useful if they let you reuse assignment functions**.

It already does:

$$
\boxed{
\text{padding}
\rightarrow FFT
\rightarrow multiplication
\rightarrow IFFT
\rightarrow rounding
}
$$

for two 1D coefficient arrays. 

So if an online asks:

> Multiply these two polynomials using FFT.

and the coefficient arrays are already in **ascending powers**, you can potentially do:

```python
coeffs, N = multiply_transform(
    a,
    b,
    FFTTransformer()
)
```

Instead of rewriting the convolution machinery.

### Very useful for

* polynomial multiplication;
* weighted polynomial multiplication;
* digit-array integer multiplication;
* any 1D **linear convolution of integer coefficient arrays**.

### Example: weighted polynomial

If:

$$
R[k]=\sum_i w_ip_iq_{k-i},
$$

then:

```python
a = p * w

result, N = multiply_transform(
    a,
    q,
    FFTTransformer()
)
```

That's an excellent shortcut.

---

# 6. But be careful with `multiply_transform`

Your Task A function is designed specifically for polynomial/integer coefficients.

It:

* rounds the inverse result;
* converts to `int64`.

So do **not** use it for arbitrary floating-point signal convolution where you want fractions.

For something like:

```python
x = [0.1, 0.7, 0.2]
h = [0.25, 0.5, 0.25]
```

write the FFT convolution normally instead of using a function that rounds coefficients.

---

# 7. `to_limbs(text)`

Your Task A function converts a huge decimal string into little-endian base-\(10^4\) coefficients. 

Example:

```python
sign, limbs = to_limbs("123456789")
```

gives conceptually:

```text
sign = 1
limbs = [6789, 2345, 1]
```

Use it only if the online gives:

```text
"65767879797907"
```

as a huge decimal **string** and asks for large integer multiplication.

Then:

```python
sign_a, a = to_limbs(text_a)
sign_b, b = to_limbs(text_b)
```

follow with:

```python
coeffs, N = multiply_transform(
    a, b, FFTTransformer()
)
```

then:

```python
answer = from_limbs(
    sign_a * sign_b,
    coeffs
)
```

---

# 8. `from_limbs(sign, limbs)`

This does the opposite:

```text
raw convolution coefficients
        ↓
carry propagation
        ↓
decimal string
```

It is designed for your base-\(10^4\) Task A representation. 

Use it when the online asks for multiplication of actual huge decimal strings and allows your assignment's limb machinery.

Do **not** use it if the online explicitly says:

> digits are base-10 and output should be LSD-first digit array.

For that PYQ, carry manually using:

```python
digit = total % 10
carry = total // 10
```

because their required representation is different.

---

# 9. `multiply(text_a, text_b, method)`

This is essentially the complete Task A operation:

```python
result, N, limbs_a, limbs_b = multiply(
    A,
    B,
    "fft"
)
```

It:

* parses decimal strings;
* converts them into limbs;
* performs FFT multiplication;
* propagates carries;
* reconstructs the string. 

### When it would be useful

If an online literally says:

> Here are two huge decimal numbers. Use your assignment implementation to multiply them.

Then this could almost directly solve the mathematical part:

```python
product, N, _, _ = multiply(
    a,
    b,
    "fft"
)
```

### When NOT to use

If they are testing whether you know how to manipulate the convolution coefficients or want a different representation.

---

# 10. `multiply_schoolbook()`

Optional \(O(N^2)\) baseline. 

Almost never useful for the online if they explicitly require FFT.

Could be useful if they ask:

> Compare FFT multiplication with direct multiplication.

Otherwise ignore it.

---

# 11. `transform_2d(plane, engine)`

This is your main image-frequency function.

It transforms every row and then every column. 

Usage:

```python
F = transform_2d(
    image,
    FFTTransformer()
)
```

Use it for:

* 2D frequency analysis;
* image convolution;
* 2D cross-correlation;
* image shift detection;
* 2D filtering;
* image deconvolution.

For example, 2D correlation:

```python
F1 = transform_2d(original, engine)
F2 = transform_2d(shifted, engine)

corr = inverse_2d(
    np.conj(F1) * F2,
    engine
).real
```

Then:

```python
dy, dx = np.unravel_index(
    np.argmax(corr),
    corr.shape
)
```

This is extremely useful if a harder online asks for full-image shift detection instead of separate rows/columns.

---

# 12. `inverse_2d(spectrum, engine)`

Exactly the reverse:

```python
image = inverse_2d(F, engine)
```

Use after frequency-domain image operations.

Example:

```python
result = inverse_2d(
    F_image * F_kernel,
    engine
).real
```

or:

```python
corr = inverse_2d(
    np.conj(F1) * F2,
    engine
).real
```

Your implementation preserves shape. 

---

# 13. `convolve_plane(plane, kernel, engine, circular=False)`

This is an extremely convenient online function for image convolution.

It already handles:

* zero-padding;
* transform sizing;
* 2D FFT;
* kernel FFT;
* multiplication;
* IFFT;
* cropping.



### Correct linear image convolution

```python
blurred = convolve_plane(
    image,
    kernel,
    FFTTransformer(),
    circular=False
)
```

Use for:

* blur;
* filter kernel;
* any 2D linear convolution.

### Circular convolution

```python
result = convolve_plane(
    image,
    kernel,
    FFTTransformer(),
    circular=True
)
```

Use if question explicitly says:

> circular convolution / periodic boundary / wrap-around.

---

# 14. `convolve_image(image, kernel, engine, circular=False)`

Difference from `convolve_plane`:

`convolve_plane` handles one 2D array:

$$
(H,W).
$$

`convolve_image` also handles RGB:

$$
(H,W,3),
$$

by convolving every color channel separately. 

So if they give you a color image:

```python
result = convolve_image(
    image,
    kernel,
    FFTTransformer()
)
```

is more convenient.

If they explicitly say grayscale or one row/plane, `convolve_plane` is enough.

---

# 15. `convolve_plane_direct()`

This computes the literal four-loop spatial convolution. 

Use it mainly for:

### Verification

```python
fast = convolve_plane(
    x, h, engine
)

slow = convolve_plane_direct(
    x, h
)

error = np.max(
    np.abs(fast - slow)
)
```

### Complexity comparison

If question asks:

> Compare spatial convolution with FFT convolution.

Otherwise it is usually too slow and not the algorithm you want.

---

# 16. `load_image()`

Provided helper:

```python
image = load_image(
    path,
    as_gray=True
)
```

returns grayscale:

$$
(H,W).
$$

Or:

```python
as_gray=False
```

returns:

$$
(H,W,3).
$$



Use if the online setup gives an image filepath and lets you import the helper.

---

# 17. `make_kernel()`

Useful only for filtering/image convolution questions.

Examples:

```python
kernel = make_kernel(
    "box",
    size=9
)
```

```python
kernel = make_kernel(
    "gaussian",
    size=15
)
```

```python
kernel = make_kernel(
    "bokeh",
    radius=9
)
```

```python
kernel = make_kernel(
    "motion",
    length=15,
    angle=30
)
```

The helper normalizes its kernel to sum to 1. 

Probably not relevant unless online involves image filtering.

---

# 18. Functions you can mostly ignore during the online

These are mainly assignment plumbing:

```python
run_single(...)
run_benchmark(...)
save_image(...)
save_comparison(...)
save_kernel_preview(...)
write_report(...)
```

You don't normally need them to solve the underlying DFT problem.

Also:

```python
ArbitraryLengthFFT
```

is optional bonus. Don't make your life harder unless specifically necessary.

---

# The online decision map

This is the part I would actually memorize.

### “Multiply two arrays/polynomials”

```python
multiply_transform(a, b, FFTTransformer())
```

or manually:

```python
FFT
*
FFT
IFFT
```

---

### “Multiply two huge decimal strings”

```python
multiply(a, b, "fft")
```

if allowed.

Or:

```text
to_limbs
→ multiply_transform
→ from_limbs
```

---

### “Linear convolution of 1D signals”

Use:

```text
next_power_of_two
→ transform
→ multiply
→ inverse
```

Don't necessarily use `multiply_transform` if values are floating point because that function rounds.

---

### “Circular convolution”

```python
engine.inverse(
    engine.transform(a)
    * engine.transform(b)
)
```

at the required circular length.

No linear-convolution padding.

---

### “Find shift”

There is **no assignment function that directly does cross-correlation**.

Build it from your FFT:

```python
A = engine.transform(reference)
B = engine.transform(shifted)

corr = engine.inverse(
    np.conj(A) * B
).real

shift = np.argmax(corr)
```

This is a major one to memorize.

---

### “Undo detected shift”

Not Fourier:

```python
fixed = np.roll(
    shifted,
    -shift
)
```

---

### “Encrypted by convolution; recover”

Again no dedicated assignment wrapper.

Use FFT directly:

```python
Y = engine.transform(encrypted)
H = engine.transform(key)

X = Y / H

restored = engine.inverse(X).real
```

---

### “Image convolution/filtering”

```python
convolve_plane(...)
```

for grayscale/one plane.

```python
convolve_image(...)
```

for complete grayscale/RGB image.

---

### “2D image shift/correlation”

```python
transform_2d(...)
inverse_2d(...)
```

and:

```python
corr = inverse_2d(
    np.conj(F1) * F2,
    engine
).real
```

---

## The smallest cheat sheet

If you're allowed to import your assignment code, these are the functions I would remember first:

```python
FFTTransformer()
engine.transform(x)
engine.inverse(X)

next_power_of_two(n)

multiply_transform(a, b, engine)

transform_2d(A, engine)
inverse_2d(F, engine)

convolve_plane(A, kernel, engine)
convolve_image(image, kernel, engine)
```

Then remember these **NumPy functions**, because your assignment does *not* wrap them for you:

```python
np.conj(X)             # correlation / deconvolution
np.argmax(x)           # strongest correlation peak
np.argmin(x)           # smallest row/value
np.argsort(x)          # rank candidates
np.roll(x, shift)      # circular shifting
np.where(...)          # conditional indices
np.abs(X)              # magnitude
np.angle(X)            # phase
np.rint(x)             # round FFT numerical error
np.unravel_index(...)  # 2D argmax -> row,column
x[::-1]                # reverse order
```

The most important exam distinction is:

$$
\boxed{\text{convolution}:\quad FFT(a)\,FFT(b)}
$$

$$
\boxed{\text{correlation}:\quad \overline{FFT(a)}\,FFT(b)}
$$

$$
\boxed{\text{deconvolution}:\quad FFT(y)/FFT(h)}
$$

Once you identify which of those three the scenario describes, choosing the assignment functions becomes fairly mechanical.



Yes — **if you know the blur kernel, you can try to undo the blur using deconvolution**.

The core idea is almost embarrassingly simple.

In Task B you blurred using

$$
Y(u,v)=X(u,v)\,H(u,v)
$$

where

* \(X\) = original image spectrum,
* \(H\) = blur-kernel spectrum,
* \(Y\) = blurred image spectrum.

Your code literally did this multiplication before the inverse 2D transform. 

So to restore:

$$
\boxed{
X(u,v)=\frac{Y(u,v)}{H(u,v)}
}
$$

and then

$$
\boxed{
x=\operatorname{IFFT2}(X)
}
$$

So:

```text
original
   ↓ FFT2
X
   × H
   ↓
Y
   ↓ IFFT2
blurred
```

To reverse:

```text
blurred
   ↓ FFT2
Y
   ÷ H
   ↓
X
   ↓ IFFT2
restored
```

## The simplest case: circular blur

Circular convolution is easiest to invert because both original and blurred images have exactly the same dimensions.

Suppose:

```python
blurred = convolve_plane(
    image,
    kernel,
    engine,
    circular=True
)
```

You could conceptually restore it like this:

```python
def deblur_circular(blurred, kernel, engine):
    H, W = blurred.shape
    kh, kw = kernel.shape

    # Put kernel in H x W array
    kernel_full = np.zeros((H, W))

    kernel_full[:kh, :kw] = kernel

    # Same origin placement used during circular convolution
    kernel_full = np.roll(
        kernel_full,
        shift=(-(kh // 2), -(kw // 2)),
        axis=(0, 1)
    )

    Y = transform_2d(blurred, engine)
    K = transform_2d(kernel_full, engine)

    X = Y / K

    restored = inverse_2d(X, engine).real

    return restored
```

Notice how similar this is to the BUET-logo decryption PYQ:

$$
encrypted=original\circledast key
$$

so:

$$
E=XK
$$

and therefore:

$$
X=E/K.
$$

Image deblurring is mathematically the same idea.

---

# But there is a major problem

Suppose some kernel frequency is:

$$
H[u,v]=0.
$$

Then you try:

$$
\frac{Y[u,v]}{0}.
$$

Impossible.

Even worse, suppose:

$$
H[u,v]=0.000001.
$$

Then some tiny error such as:

$$
Y[u,v]=0.000003+\text{noise}
$$

gets divided by \(10^{-6}\), making the noise enormous.

This is why blur is generally **much easier to apply than to undo**.

---

# Why does blur cause this problem?

Remember what blur does in frequency domain.

A blur kernel is a low-pass filter.

Roughly:

```text
Kernel spectrum magnitude

low frequency                 high frequency
      1.0 ──────────\
                      \
                       \
                        \______ near 0
```

Original image might contain:

```text
low frequencies   → smooth regions
high frequencies  → edges, hair, text, fine details
```

Blurring performs:

$$
Y=XH.
$$

So if:

$$
H=1
$$

at a low frequency, nothing happens.

But if:

$$
H=0.01
$$

at a high frequency:

$$
100\times0.01=1.
$$

That detail becomes very weak.

To restore:

$$
1/0.01=100.
$$

Works mathematically.

But suppose because of image noise or numerical error you had:

$$
1.05
$$

instead.

Then:

$$
1.05/0.01=105.
$$

The error was only:

$$
0.05
$$

before division, but becomes:

$$
5
$$

after deconvolution.

So:

$$
\boxed{\text{deconvolution strongly amplifies noise at frequencies suppressed by the blur}}
$$

---

# Safer deblurring: don't divide blindly

Instead of:

```python
X = Y / K
```

you often use:

```python
eps = 1e-6

X = Y * np.conj(K) / (
    np.abs(K)**2 + eps
)
```

This corresponds to:

$$
\boxed{
\hat X
=
\frac{YK^*}
{|K|^2+\epsilon}
}
$$

This is a simple regularized inverse filter.

Why is this safer?

If:

$$
|K|\approx0,
$$

the denominator doesn't become exactly zero because of:

$$
+\epsilon.
$$

So extremely weak frequencies aren't amplified infinitely.

Code:

```python
def deblur_circular_safe(blurred, kernel, engine, eps=1e-6):
    H, W = blurred.shape
    kh, kw = kernel.shape

    kernel_full = np.zeros((H, W))
    kernel_full[:kh, :kw] = kernel

    kernel_full = np.roll(
        kernel_full,
        shift=(-(kh // 2), -(kw // 2)),
        axis=(0, 1)
    )

    Y = transform_2d(blurred, engine)
    K = transform_2d(kernel_full, engine)

    X = (
        Y * np.conj(K)
        / (np.abs(K)**2 + eps)
    )

    restored = inverse_2d(X, engine).real

    return restored
```

---

# What about your normal zero-padded `blurred.png`?

This is a little more complicated.

Your Task B linear convolution does:

```text
original H×W
       ↓
zero pad
       ↓
full convolution
       ↓
crop H×W
       ↓
blurred.png
```

Your code specifically takes the inverse result and then crops:

```python
r, c = kh // 2, kw // 2

return res[r:r+H, c:c+W]
```



That means some samples from the **full convolution result were thrown away**.

So if all you possess is the final cropped:

```text
blurred.png
```

then exact inversion is not as straightforward as circular deconvolution.

You've lost some boundary information.

---

# Even more important: saving PNG loses information

Your helper does:

```python
data = np.clip(array, 0.0, 1.0)
data = np.rint(data * 255.0).astype(np.uint8)
```

when saving. 

So before saving you might have:

$$
0.372849263...
$$

After PNG conversion it becomes one of only 256 possible intensity levels.

Approximately:

$$
95/255.
$$

That quantization error may seem tiny, but inverse filtering can amplify it enormously.

Therefore:

### If you deblur the in-memory float `blurred`

you can get a much better reconstruction.

### If you reload `blurred.png`

you have already lost precision, so restoration will usually be worse.

---

# Theoretical best case

Suppose:

* you use circular convolution;
* the exact kernel is known;
* no noise is added;
* you never save/reload as 8-bit PNG;
* every frequency of the kernel is nonzero.

Then:

$$
Y=XH
$$

and:

$$
X=Y/H
$$

can theoretically recover the original essentially perfectly, up to floating-point error.

You might see:

$$
10^{-12},10^{-13},10^{-14}
$$

type differences.

---

# Realistic case

Suppose you:

* blur strongly;
* save as PNG;
* have sensor noise;
* use a kernel with almost-zero high-frequency response.

Then exact inverse filtering can produce something like:

```text
blurred image
      ↓
naive deconvolution
      ↓
sharper image + HUGE ringing/noise
```

You may see strange halos and oscillations around edges.

That effect is called **ringing**.

---

# Gaussian blur is particularly hard to reverse strongly

A Gaussian kernel's Fourier transform also looks roughly Gaussian.

It aggressively shrinks high frequencies:

$$
H(\omega)\rightarrow0
$$

as frequency grows.

Therefore dividing by \(H\) at high frequencies becomes unstable.

A mild Gaussian blur can often be partly reversed.

A severe Gaussian blur destroys fine-detail information so strongly that perfect restoration becomes practically impossible.

---

# An intuitive example

Imagine original frequency coefficients:

$$
X=[100,\;50,\;20,\;10].
$$

Suppose blur response:

$$
H=[1,\;0.8,\;0.1,\;0.001].
$$

Blur gives:

$$
Y=XH
$$

so:

$$
Y=[100,\;40,\;2,\;0.01].
$$

Mathematically:

$$
Y/H
=
[100,\;50,\;20,\;10].
$$

Perfect.

But suppose slight noise changes the last value:

$$
0.01\rightarrow0.02.
$$

Then restore:

$$
0.02/0.001=20.
$$

Original was:

$$
10.
$$

You got:

$$
20.
$$

Just a \(0.01\) error became a \(10\) error.

That's the fundamental deblurring problem.

---

# This gives you another likely online-question pattern

If the examiner says:

> Image \(Y\) was blurred by known kernel \(H\). Recover \(X\).

Immediately think:

$$
\boxed{\text{deconvolution}}
$$

and write:

```python
Y = transform_2d(blurred, engine)
H = transform_2d(kernel_padded, engine)

X = Y / H

restored = inverse_2d(X, engine).real
```

If they mention:

> noise / unstable division / zeros in kernel spectrum

then think:

$$
\boxed{\text{regularized inverse}}
$$

```python
X = Y * np.conj(H) / (
    np.abs(H)**2 + eps
)
```

So your three major Fourier-domain operations now fit together very neatly:

$$
\boxed{\text{Convolution: }Y=XH}
$$

$$
\boxed{\text{Deconvolution: }X=Y/H}
$$

$$
\boxed{\text{Correlation: }R=X^*Y}
$$

Those are probably the three most useful equations to recognize quickly in your online.



Yes. **If you still have the full linear-convolution result before cropping, then in principle you can recover the original image much more cleanly.**

That is exactly the important distinction.

In your Task B linear branch, you effectively had:

$$
X_{\text{pad}}
\overset{FFT2}{\longrightarrow}
F_X
$$

$$
H_{\text{pad}}
\overset{FFT2}{\longrightarrow}
F_H
$$

then

$$
F_Y = F_XF_H
$$

and finally

$$
Y_{\text{full}}=IFFT2(F_Y).
$$

At that point, **before this line**:

```python
return res[r:r+H, c:c+W]
```

you still have the entire convolution result in `res`.

So if the kernel is known:

$$
F_X=\frac{F_Y}{F_H},
$$

and then:

$$
X_{\text{pad}}=IFFT2(F_X).
$$

Finally take:

```python
original = X_padded[:H, :W].real
```

## Why is this much better than trying to undo the cropped image?

Because cropping is:

```text
full convolution
      ↓
throw away border samples
      ↓
keep only H × W
```

Once those border samples are thrown away, you've discarded equations/information about the original.

Before cropping, nothing has been intentionally thrown away:

```text
original
   ↓
zero padding
   ↓
convolution with known kernel
   ↓
FULL convolution
```

So deconvolution has access to everything produced by the convolution.

---

## Using your exact Task B structure

Your linear convolution code does approximately:

```python
padded_plane = np.zeros((Nr, Nc))
padded_kernel = np.zeros((Nr, Nc))

padded_plane[:H, :W] = plane
padded_kernel[:kh, :kw] = kernel

X = transform_2d(padded_plane, engine)
K = transform_2d(padded_kernel, engine)

Y = X * K

res = inverse_2d(Y, engine).real
```

At this point, suppose we keep `res`.

To reverse:

```python
Y = transform_2d(res, engine)

K = transform_2d(padded_kernel, engine)

X = Y / K

recovered_padded = inverse_2d(X, engine).real

recovered = recovered_padded[:H, :W]
```

Conceptually:

$$
\boxed{
X_{\text{pad}}
\xrightarrow{\times H}
Y_{\text{full}}
\xrightarrow{\div H}
X_{\text{pad}}
}
$$

So yes, this is almost exactly undoing the operation you performed.

---

# Tiny 1D analogy

Suppose:

$$
x=[1,2,3]
$$

and:

$$
h=[1,1].
$$

Full linear convolution is:

$$
y=[1,3,5,3].
$$

Notice it has length:

$$
3+2-1=4.
$$

If you keep all four values, you can reconstruct \(x\).

In fact, even without Fourier:

$$
y_0=x_0=1
$$

then:

$$
y_1=x_0+x_1=3
$$

so:

$$
x_1=2.
$$

Then:

$$
y_2=x_1+x_2=5
$$

so:

$$
x_2=3.
$$

The full convolution contains enough information.

---

# But suppose you crop

Imagine you only kept:

$$
[3,5,3]
$$

or some same-length aligned section.

You've thrown away:

$$
1.
$$

Now recovering the exact original can become more difficult or ambiguous depending on the kernel and boundary assumptions.

That is why:

$$
\boxed{\text{full convolution is much better for deconvolution than cropped convolution}.}
$$

---

# One catch: division by zero in the kernel spectrum

Even if you keep the full convolution, this:

```python
X = Y / K
```

only works directly if:

$$
K[u,v]\neq0
$$

for every required frequency bin.

If some:

$$
K[u,v]=0,
$$

then that frequency was completely killed:

$$
Y[u,v]=X[u,v]\times0=0.
$$

You cannot recover \(X[u,v]\) by division.

And if \(K[u,v]\) is merely tiny, division can amplify floating-point error dramatically.

So in practice you'd often use:

```python
eps = 1e-10

X = (
    Y * np.conj(K)
    / (np.abs(K)**2 + eps)
)
```

instead of raw division.

---

# But mathematically, isn't full linear convolution unique?

There's a subtle distinction worth knowing.

For finite exact sequences, if the kernel is nonzero, full polynomial convolution generally uniquely determines the original finite sequence.

For example:

$$
Y(z)=X(z)H(z).
$$

If \(H(z)\) is known, conceptually polynomial division can recover \(X(z)\).

So even when one particular sampled DFT representation has a zero bin and simple:

$$
Y[k]/H[k]
$$

fails, that doesn't necessarily mean the full finite convolution itself contains no solution. It means **naive frequency-bin division is problematic at that representation**.

For your lab, though, the practical answer is:

> If the kernel spectrum has no problematic zeros, full-result FFT deconvolution is straightforward.

---

# What if your padded FFT size is bigger than the full convolution?

For your skyline example:

$$
512\times512
$$

with

$$
19\times19
$$

kernel.

Full linear convolution:

$$
530\times530.
$$

But radix-2 FFT used:

$$
1024\times1024.
$$

Your `res` coming directly from the IFFT is therefore actually:

$$
1024\times1024.
$$

The meaningful full convolution occupies the appropriate region, while the remaining space is essentially zero up to numerical error.

If you kept the entire `1024×1024` `res`, reversal is especially simple because it already has exactly the transform shape you originally used:

```python
Y = transform_2d(res, engine)
```

then divide by the exact same padded-kernel spectrum.

---

# So the ideal experiment would be

Instead of immediately doing:

```python
res = np.real(
    inverse_2d(
        spectrum_plane * spectrum_kernel,
        engine
    )
)

r, c = kh // 2, kw // 2

return res[r:r+H, c:c+W]
```

temporarily retain:

```python
full_padded_result = res.copy()
```

Then try:

```python
Y = transform_2d(full_padded_result, engine)

K = transform_2d(padded_kernel, engine)

X_recovered = Y / K

plane_recovered = inverse_2d(
    X_recovered,
    engine
).real

plane_recovered = plane_recovered[:H, :W]
```

and compare:

```python
error = np.max(
    np.abs(plane_recovered - plane)
)

print(error)
```

If the chosen kernel has no troublesome zero spectral bins and you're using the in-memory floating values, you could get extremely small error.

So the hierarchy is:

$$
\boxed{\text{full convolution before crop}}
$$

best chance of exact recovery,

$$
\boxed{\text{cropped floating-point blur}}
$$

less information / harder,

and

$$
\boxed{\text{cropped blur saved to 8-bit PNG}}
$$

even harder because cropping **and** quantization have already discarded information.
