# EV-FFT (MATLAB)

MATLAB Live Script pipeline for **pixel-wise FFT analysis** of time-lapse microscopy (ND2/TIFF) to generate **frequency-domain stacks** and **low-frequency energy (RSS) maps** for EV/exosome signal visualization.

> This repository currently provides the workflow as **.mlx (MATLAB Live Scripts)**. You can run them as-is in MATLAB, or export to `.m` if you want a function/script-only version.

---

## Contents

| Script | Purpose |
|---|---|
| `00_nd2_tiff_export.mlx` | Export ND2 → TIFF time-series (requires Bio-Formats `bfmatlab`) |
| `01_kjs_fft_whole_240327.mlx` | Build per-pixel **one-sided FFT amplitude stacks** and save as `.mat` |
| `02_kjs_fft_load_240327.mlx` | Batch-load saved FFT stacks into a structured variable (e.g., `exosomeData`) |
| `03_ver2_kjs_fft_ImageProcessing_251116.mlx` | Post-processing: low-frequency RSS maps, ROI cropping/normalization, cutoff iFFT, metrics (e.g., Frechet distance) |

---

## Requirements

- MATLAB (Live Scripts supported)
- Image Processing Toolbox (recommended)
- Parallel Computing Toolbox (optional; for `parfor`)
- **Bio-Formats `bfmatlab`** (only for ND2 export in `00_nd2_tiff_export.mlx`)

---

## Recommended Folder Structure

EV-FFT/
00_nd2_tiff_export.mlx
01_kjs_fft_whole_240327.mlx
02_kjs_fft_load_240327.mlx
03_ver2_kjs_fft_ImageProcessing_251116.mlx

(optional) raw_nd2/
sample1.nd2
sample2.nd2

(generated after export)
sample1/
sample1_T0001.tif
sample1_T0002.tif
...
sample2/
sample2_T0001.tif
...



---

## Quick Start

### 1) (Optional) Convert ND2 → TIFF
1. Place `*.nd2` files in your working directory (or update the script path logic).
2. Open and run: `00_nd2_tiff_export.mlx`
3. The script typically creates a folder per ND2 file and exports frames as:
   - `FileName_T0001.tif`, `FileName_T0002.tif`, ...

> **Note:** Update `addpath(...)` inside the script to your local `bfmatlab` path.

---

### 2) Generate FFT amplitude stacks (core step)
1. Ensure each sample folder contains a TIFF time-series (`*.tif`).
2. Run: `01_kjs_fft_whole_240327.mlx`

Output (per folder):
- `*_exosome_frequency_stack.mat` (saved with `-v7.3`)
- Contains `exosome_frequency_stack` with shape: **(height × width × nFreqBins)**

**Sampling note:** The script assumes the original acquisition is ~**40 Hz** and uses:
- `Sr` (downsampling factor)
- `Fs = round(40/Sr)` (effective sampling rate)

---

### 3) Batch-load all stacks into one variable
Run: `02_kjs_fft_load_240327.mlx`

This script finds `*_exosome_frequency_stack.mat` files and loads them into a single structure
(e.g., `exosomeData.<fieldName> = exosome_frequency_stack`).

---

### 4) Post-processing / visualization
Run: `03_ver2_kjs_fft_ImageProcessing_251116.mlx`

Typical operations included:
- Low-frequency **RSS energy map** (excluding DC)
- Global normalization and ROI cropping
- Cutoff / reconstruction with iFFT
- Similarity metrics (e.g., Frechet distance)

Because this script is analysis-oriented, you will likely need to **select the target dataset**
and adjust **ROI coordinates** / **thresholding parameters** for your data.

---

## Key Outputs

- **FFT amplitude stack**: `exosome_frequency_stack(y, x, f)`
- **Energy/RSS maps** (example concept; varies by section in script):
  - Exclude DC and sum low-frequency bins:
    - `E(x,y) = sqrt( sum_k |A(x,y,k)|^2 )`

---

## Large File Notes (GitHub)

Time-series TIFFs and FFT stacks can be very large. A common approach is:
- Upload **code only**
- Exclude raw/derived data via `.gitignore`
- Use Git LFS if you must version large binaries

Suggested `.gitignore` snippet:
```gitignore
# raw data
*.nd2
*.tif
*.tiff

# large outputs
*.mat

# MATLAB autosave
*.asv
