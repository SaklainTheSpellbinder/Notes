[Noisy Image (64x64)]
           │
           ▼ (Row-by-Row 1D Forward FT)
  [Row Spectra Matrix (64 x N_f)]
           │
           ▼ (Find abnormally large peak across rows -> f_noise)
  [Identify Noise Frequency f_noise]
           │
           ▼ (Zero out values where f ≈ ±f_noise)
  [Filtered Row Spectra Matrix]
           │
           ▼ (Row-by-Row 1D Inverse FT)
  [Denoised Clean Image] ───> Reveals the hidden secret letter!



'''python
import matplotlib.pyplot as plt
import numpy as np


# 1. 1D Numerical Forward & Inverse Continuous Fourier Transforms
def cft_1d(signal_1d, x_grid, f_grid):
    """X(f) = integral( x(t) * exp(-j 2pi f t) dt )[cite: 1]"""
    kernel = np.exp(-1j * 2 * np.pi * f_grid[:, None] * x_grid[None, :])
    if hasattr(np, 'trapezoid'):
        return np.trapezoid(signal_1d[None, :] * kernel, x_grid, axis=1)
    return np.trapz(signal_1d[None, :] * kernel, x_grid, axis=1)


def icft_1d(X_f, f_grid, x_grid):
    """x(t) = integral( X(f) * exp(+j 2pi f t) df )"""
    kernel = np.exp(1j * 2 * np.pi * x_grid[:, None] * f_grid[None, :])
    if hasattr(np, 'trapezoid'):
        return np.real(
            np.trapezoid(X_f[None, :] * kernel, f_grid, axis=1)
        )
    return np.real(np.trapz(X_f[None, :] * kernel, f_grid, axis=1))


# -------------------------------------------------------------
# 2. Main Denoising Pipeline
# -------------------------------------------------------------
def denoise_image_row_by_row(noisy_img, x_grid, f_grid):
    H, W = noisy_img.shape
    N_f = len(f_grid)

    # Step 1: Forward FT row by row
    spectrum_rows = np.zeros((H, N_f), dtype=complex)
    for i in range(H):
        spectrum_rows[i, :] = cft_1d(noisy_img[i, :], x_grid, f_grid)

    # Step 2: Identify noise frequency
    # Average magnitude spectrum across all rows to locate the common noise spike
    avg_magnitude = np.mean(np.abs(spectrum_rows), axis=0)

    # Ignore the DC center bin (f = 0) and find the dominant high-frequency spike
    non_dc_mask = np.abs(f_grid) > 0.5
    f_noise_idx = np.argmax(avg_magnitude * non_dc_mask)
    f_noise = np.abs(f_grid[f_noise_idx])
    print(f'Detected interference stripe frequency: {f_noise:.3f} Hz')

    # Step 3: Suppress the noise frequencies (Notch Filter)
    # Zero out coefficients within a narrow notch width around +/- f_noise
    notch_width = 0.2
    noise_filter_mask = (np.abs(f_grid - f_noise) < notch_width) | (
        np.abs(f_grid + f_noise) < notch_width
    )

    filtered_spectrum = spectrum_rows.copy()
    filtered_spectrum[:, noise_filter_mask] = 0.0

    # Step 4: Inverse FT row by row to reconstruct the image
    clean_img = np.zeros_like(noisy_img)
    for i in range(H):
        clean_img[i, :] = icft_1d(filtered_spectrum[i, :], f_grid, x_grid)

    return clean_img


Here is a simple, intuitive breakdown of how noise detection and notch filtering work.

---

### Part 1: How We Identify the Noise Frequency

#### The Problem: Finding the Spike

Every row in your image has the secret letter (smooth shapes) plus a rapid vertical stripe oscillation ($A \cos(2\pi f_{\text{noise}} x)$).

When you take the Fourier transform of a row:

1. **$f = 0$ (DC component):** Contains the average brightness of the image (always the biggest value).
2. **Low frequencies:** Contain the actual shapes and letters.
3. **$f_{\text{noise}}$:** A sharp, artificial spike sticking out above everything else.

#### How the Code Finds It:

```python
# 1. Average the magnitude across all 64 rows to make the noise spike stand out clearly
avg_magnitude = np.mean(np.abs(spectrum_rows), axis=0)

# 2. Ignore f = 0 (the DC brightness peak) so argmax doesn't just pick f = 0
non_dc_mask = np.abs(f_grid) > 0.5

# 3. Find the index where the magnitude is highest (excluding DC)
f_noise_idx = np.argmax(avg_magnitude * non_dc_mask)

# 4. Read the frequency value at that peak index
f_noise = np.abs(f_grid[f_noise_idx])

```

* If `avg_magnitude * non_dc_mask` peaks at index `420`, and `f_grid[420] = 5.2`, then **$f_{\text{noise}} = 5.2\text{ Hz}$**.

---

### Part 2: How We Clean (Zero Out) That Frequency

A Fourier transform is symmetric: an oscillation at frequency $f_{\text{noise}}$ produces two peaks in the spectrum — one at $+f_{\text{noise}}$ and one at $-f_{\text{noise}}$.

To remove the noise, we create a **Notch Filter** (a mask that zeros out only those frequencies).

```python
notch_width = 0.2

# Find all frequency indices close to +f_noise OR close to -f_noise
noise_filter_mask = (np.abs(f_grid - f_noise) < notch_width) | (
    np.abs(f_grid + f_noise) < notch_width
)

# Set the spectrum values at those frequencies to 0 for every row
filtered_spectrum = spectrum_rows.copy()
filtered_spectrum[:, noise_filter_mask] = 0.0

```

#### What `np.abs(f_grid - f_noise) < notch_width` means:

* Suppose $f_{\text{noise}} = 5.2$ and `notch_width = 0.2`.
* `np.abs(f_grid - 5.2) < 0.2` selects all frequencies between **$5.0$ and $5.4$**.
* `np.abs(f_grid + 5.2) < 0.2` selects all frequencies between **$-5.4$ and $-5.0$**.

By setting those frequency bins to `0.0`, you surgically delete the sinusoidal stripes while leaving all other image frequencies untouched. When you do the inverse transform back to the spatial domain, the stripes are gone and only the clean letter remains.


In an exam or lab setting, you don't even need complicated code to find the noise frequency. You just **plot the spectrum and look at it with your eyes first**, then zero out that number!

Here is the exact step-by-step procedure to do it manually in 4 simple steps:

---

### Step 1: Compute the CFT of Row 0 and Plot It

Pick the first row of your image (or row 30), run the 1D CFT on it, and plot the magnitude spectrum:

```python
# Transform just the first row to see what the spectrum looks like
row_spectrum = cft_1d(noisy_img[0, :], x, f)

import matplotlib.pyplot as plt

plt.plot(f, np.abs(row_spectrum))
plt.title('Row 0 Spectrum')
plt.xlabel('Frequency f')
plt.grid(True)
plt.show()

```

---

### Step 2: Read the Peak Off Your Plot

When the plot pops up:

1. You will see a big peak at $f = 0$ (that's just image brightness).
2. You will see a distinct, sharp spike sticking out at some number—say, $f = 4.0$ (and $-4.0$).
3. **Boom, you found it.** The noise frequency is $f_0 = 4.0$.

---

### Step 3: Zero Out That Column / Frequency in a Loop

Now transform all 64 rows, and set that specific column to 0:

```python
# Transform all rows
filtered_spectrum = np.zeros((64, len(f)), dtype=complex)
for i in range(64):
    filtered_spectrum[i, :] = cft_1d(noisy_img[i, :], x, f)

# Manual Notch Filter: Zero out around the peak you saw on your plot (e.g., 4.0)
f_noise = 4.0  # (replace with whatever number you saw on the plot!)

# Zero out everything near +4.0 and -4.0
bad_freqs = (np.abs(f - f_noise) < 0.2) | (np.abs(f + f_noise) < 0.2)
filtered_spectrum[:, bad_freqs] = 0.0

```

---

### Step 4: Inverse Transform Row-by-Row & Show the Secret Letter

```python
clean_img = np.zeros_like(noisy_img)

for i in range(64):
    clean_img[i, :] = icft_1d(filtered_spectrum[i, :], f, x)

# Display the clean image to read the letter
plt.imshow(clean_img, cmap='gray')
plt.title('Denoised Image')
plt.show()

```

That’s all there is to it: **Plot 1 row $\rightarrow$ find the spike $\rightarrow$ zero it out for all rows $\rightarrow$ inverse transform.**''




Finding the noise spike in 2D is the exact 2D equivalent of what we did in 1D. You can do it visually (easiest/recommended) or with a few lines of code.

---

### Method 1: The Visual Way (Easiest)

Call your `plot_magnitude()` method or display the log-magnitude spectrum:

```python
# Compute CFT
real, imag = cft2d.compute_cft()
magnitude = np.sqrt(real**2 + imag**2)

# Plot the 2D spectrum with frequency axes
plt.figure(figsize=(7, 6))
plt.imshow(
    np.log(1 + magnitude),
    extent=[cft2d.u[0], cft2d.u[-1], cft2d.v[-1], cft2d.v[0]],
    cmap="gray",
)
plt.title("2D Magnitude Spectrum (Look for bright dots!)")
plt.xlabel("Horizontal Frequency u")
plt.ylabel("Vertical Frequency v")
plt.colorbar(label="log(1 + |F|)")
plt.show()

```

#### What to Look For:

1. **Center $(0, 0)$:** A large glowing dot at the center (the DC brightness).
2. **Noise Spikes:** Two bright, symmetric isolated dots located away from the center:
* **Vertical stripes** $\rightarrow$ dots appear along the horizontal axis at $(\pm u_0, 0)$.
* **Horizontal stripes** $\rightarrow$ dots appear along the vertical axis at $(0, \pm v_0)$.
* **Diagonal stripes** $\rightarrow$ dots appear diagonally at $\pm(u_0, v_0)$.


3. Hover your cursor over the dots or read the axis values: those coordinates are your $(u_0, v_0)$!

---

### Method 2: The Automatic Code Way (`np.argmax` with 2D Masking)

To find the coordinates automatically without getting trapped by the center DC peak at $(0, 0)$:

```python
def find_2d_noise_spike(real, imag, u_grid, v_grid, dc_radius=3.0):
    magnitude = np.sqrt(real**2 + imag**2)

    # Create 2D frequency coordinate matrices
    U, V = np.meshgrid(u_grid, v_grid)

    # 1. Mask out the center DC region so argmax ignores (0,0)
    dist_from_dc = np.sqrt(U**2 + V**2)
    non_dc_magnitude = magnitude.copy()
    non_dc_magnitude[dist_from_dc < dc_radius] = 0.0

    # 2. Find the (row, col) index of the brightest peak outside DC
    max_idx_flat = np.argmax(non_dc_magnitude)
    row_idx, col_idx = np.unravel_index(max_idx_flat, magnitude.shape)

    # 3. Look up the exact frequency coordinates (u0, v0)
    u0 = u_grid[col_idx]
    v0 = v_grid[row_idx]

    print(
        f"Detected 2D noise spike at: u0 = {u0:.3f}, v0 = {v0:.3f} (and symmetric pair)"
    )
    return u0, v0

```

---

### Putting It All Together to Filter the Image

Once you have `u0` and `v0`:

```python
# 1. Detect spike
u0, v0 = find_2d_noise_spike(real, imag, cft2d.u, cft2d.v)

# 2. Notch filter out the spikes at +(u0, v0) and -(u0, v0)
filt = FrequencyFilter()
real_clean, imag_clean = filt.notch_filter(
    real, imag, cft2d.u, cft2d.v, u0=u0, v0=v0, notch_radius=0.5
)

# 3. Reconstruct
icft2d = InverseCFT2D(
    real_clean, imag_clean, cft2d.u, cft2d.v, img.x, img.y
)
clean_img = icft2d.reconstruct()

# 4. View the result
plt.imshow(clean_img, cmap="gray")
plt.title("Clean Denoised Image")
plt.axis("off")
plt.show()

```



