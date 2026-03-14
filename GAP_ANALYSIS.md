# PythonDancer — Gap Analysis

This document captures known gaps, incomplete features, and code quality issues as of March 2026. Intended to inform a rewrite decision.

---

## Critical Bugs

| Location | Issue |
|---|---|
| `libfun.py:13` | TODO: action lag that happens sometimes — hop_length may need tuning |
| `libfun.py:268` | Hardcoded test path `/mnt/newfiles/Video/MBad2/...` will crash if `__main__` is run |
| `ui.py:271,280` | `FigureCanvas` created without a `Figure` argument — `.figure` is unreliable across matplotlib versions (savefig bug) |

---

## Incomplete Implementations

| Location | Issue |
|---|---|
| `libfun.py:222` | `render_heatmap()` marked `TODO: Do better` — fixed dimensions (4096×128), no customization |
| `libfun.py:36` | `TODO: The fuck does this do` — weighted pitch average with no documentation |
| `ui.py:629` | `TODO: While loop it without UI freeze` — render polling uses `after(10ms)` instead of event-driven |
| `ui.py:437` | `TODO: More binds` — keyboard/mouse bindings incomplete |
| `ui.py:747–753` | Previous heatmap save path commented out, replaced but not cleaned up |

---

## Error Handling Gaps

- `cli.py:35–39`: Catches `Exception` generically, prints nothing useful, silently returns
- `ui.py:107–110`: Exception caught but not logged; user sees only generic progress message
- `ui.py:158–162`: Exception printed to console only — not visible in GUI
- `util.py:5–18`: Only catches `FileNotFoundError` for ffmpeg; other subprocess errors (version mismatch, bad binary) not handled
- No validation on: audio file format, array bounds in action creation, empty/very short audio files, out-of-memory on large files
- `autoval()` in `libfun.py` calls `scipy.optimize.minimize` with no convergence check or error handling

---

## Thread Safety

- `self.data` and `self.result` shared between `LoadWorker` and `RenderWorker` threads with no locking
- Race condition possible if load and render overlap

---

## CLI Gaps

- No progress output during processing
- No batch file processing
- No config file / preset support from CLI
- `--amplitude_centering` and `--center_offset` args missing or inconsistent
- No structured exit codes

---

## Missing Features

| Feature | Notes |
|---|---|
| Recent files list | No quick access to previous files |
| Preset save/load | Can't save slider configurations |
| Undo/redo | No way to revert parameter changes |
| Audio playback | No preview of source audio in GUI |
| Batch processing | Single file only |
| Export formats | Only funscript JSON and CSV |
| Real logging | All output via `print()`; no log file |
| Input validation | No format/duration/size checks before loading |

---

## Code Quality

- No type hints anywhere
- No docstrings on any functions
- Magic numbers throughout (`hop_length=1024`, energy multiplier `50`, speed normalization `400.0`, heatmap `4096`/`128`)
- Inconsistent style: `if (condition):` vs `if condition:`, variable naming inconsistency (`tpi`, `auto_pitch`)
- `sys` imported in `ui.py` but never used
- Commented-out Qt code in `ui.py:73–85` (dead code from original port)
- `VERSION = "?"` hardcoded in `libfun.py` — not wired to package metadata

---

## Architecture

- No MVC or clear separation between data and presentation in `ui.py` (765 lines, does too much)
- Core logic in `libfun.py` is tightly coupled to specific librosa API — hard to swap audio backends
- No plugin or extension points
- Config stored in `config.json` in working directory — not platform-appropriate (`~/.config` etc.)

---

## Dependencies

| Package | Issue |
|---|---|
| `numpy <1.24` | Very old constraint; current numpy is 2.x |
| `scipy ==1.12.0` | Exact pin is inflexible |
| `librosa ==0.10.0.post2` | Post-release pin, unusual |
| `audioread` | Used in `libfun.py` but not listed in `requirements.txt` |

---

## Build & Deployment

- `BUILD_AND_DEPLOY.md` references Python 3.13 but `pyproject.toml` specifies 3.8+
- `BUILD_AND_DEPLOY.md` contains a hardcoded user-specific path (`c:\Users\bruce\...`)
- No automated tests — no test directory, no CI
- Version not managed — must be updated manually in source

---

## Summary by Priority

**High** — would break a rewrite if not addressed:
1. Action timing lag (algorithm correctness)
2. Thread safety between load and render
3. No input validation (crashes on bad input)
4. Algorithm documentation missing (pitch weighting especially)

**Medium** — significant UX or maintainability debt:
1. Error handling — users get no useful feedback on failure
2. CLI is minimal and incomplete
3. Architecture coupling makes testing hard
4. Dependency pins are very conservative

**Low** — polish and completeness:
1. Missing features (batch, presets, audio playback)
2. Code style, type hints, docstrings
3. Build automation and version management
4. Config file location
