# PythonDancer — Build and Deploy Guide

## Prerequisites

- [Miniconda](https://docs.conda.io/en/latest/miniconda.html) (Python 3.13)
- Git configured with user name and email:
  ```bash
  git config --global user.name "lqr"
  git config --global user.email "lqr@liquidreleasing.com"
  ```

## First-Time Setup

Install all Python dependencies:

```bash
cd c:\Users\bruce\Projects\PythonDancer
pip install -r requirements.txt
```

This installs: `numpy`, `librosa`, `matplotlib`, `scipy`, `pyinstaller`, `requests`.

## Build

Run the builder script from the project root:

```bash
cd c:\Users\bruce\Projects\PythonDancer
python builder.py
```

The builder will:
1. Download `ffmpeg.exe` into `dist/` (skipped if already present)
2. Download `upx.exe` for executable compression
3. Run PyInstaller using `qt.spec` to produce `dist/PythonDancer.exe`

Output is in `dist/`:
- `PythonDancer.exe`
- `ffmpeg.exe`

## Deploy

Copy the build output to the local installation folder:

```bash
cp dist/PythonDancer.exe "c:\Users\bruce\Tools\PythonDancer\PythonDancer.exe"
cp dist/ffmpeg.exe "c:\Users\bruce\Tools\PythonDancer\ffmpeg.exe"
```

> **Note:** If the copy fails with "Device or resource busy", PythonDancer.exe is currently running. Close it and retry.

The file `config.json` in the Tools folder is user configuration and is never overwritten by the deploy step.

## Run (Development)

To run directly without building an exe:

```bash
cd c:\Users\bruce\Projects\PythonDancer
python -m dancer
```

CLI mode:

```bash
python -m dancer --cli -h
```

## Verify the Build

After deploying, launch the exe to confirm it starts correctly:

```bash
"c:\Users\bruce\Tools\PythonDancer\PythonDancer.exe"
```

Common startup errors and fixes:

| Error | Cause | Fix |
|-------|-------|-----|
| `No module named 'librosa'` | librosa not installed before build | `pip install librosa` then clean rebuild |
| `No module named 'matplotlib'` | matplotlib not installed before build | `pip install matplotlib` then clean rebuild |
| `Device or resource busy` | exe is running during deploy | Close the app and re-run the copy commands |

> **Important:** If you install a new dependency and rebuild without clearing the cache, PyInstaller may reuse stale analysis and still exclude the module. Always do a clean rebuild after installing new packages:
> ```bash
> rm -rf build dist
> python builder.py
> ```
