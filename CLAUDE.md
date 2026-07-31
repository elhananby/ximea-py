# ximea-py

Python wrapper for the [XIMEA](https://www.ximea.com/) camera xiApi SDK. Wraps
the vendor's C API (`libm3api.so.2`, installed system-wide, not via `uv`) to
provide `Camera` and `Image` classes for image acquisition, parameter control,
and image data retrieval.

This is the canonical XIMEA driver used across the `mpinb` optogenetics/
tracking ecosystem — `liquid-lens-calibration` and `optofly` both depend on
it directly via `ximea @ git+https://github.com/mpinb/ximea-py.git`. It is
**not** the same package as PyPI's `ximea-py`, which is unrelated vendor
bindings without the `Camera`/`Image` top-level re-export or the sensor
corrections described below.

## Structure

```
ximea/
├── __init__.py       # re-exports Camera, Image at top level
├── xiapi.py          # Camera and Image classes wrapping the C xiApi
├── xidefs.py         # constants, error codes, parameter definitions
├── xi_wintypes.py    # Windows type compatibility mappings via ctypes
└── libs/x64/xiArrOps.so   # prebuilt shared library, shipped in the wheel
examples/              # one script per usage pattern, referenced from README
```

## Build

Packaged with `hatchling`. `[tool.hatch.build.targets.wheel] packages =
["ximea"]` in `pyproject.toml` is sufficient to include `ximea/libs` — do not
add a `force-include` entry for the same path, hatchling will try to add
`xiArrOps.so` to the wheel twice and the build will fail with:

```
ValueError: A second file is being added to the wheel archive at the same
path: ximea/libs/x64/xiArrOps.so.
```

(This exact bug existed in this repo's history until it was fixed — don't
reintroduce it.)

Verify a build actually includes the shared library, not just that it exits 0:

```bash
uv build
python3 -c "
import zipfile, glob
z = zipfile.ZipFile(glob.glob('dist/*.whl')[0])
assert [n for n in z.namelist() if 'xiArrOps' in n]
"
```

## Development

```bash
uv sync --group dev   # installs pytest, ruff, numpy, and example deps (av, pyzmq, matplotlib)
ruff check .
```

There is no test suite in this repo yet (`dev` group installs `pytest` but no
`tests/` directory exists) — verification is currently build + manual import
checks like the one above, and exercising the `examples/` scripts against
real hardware.

## Sensor corrections and buffer staleness

`open_device()` (and the reopen path) auto-enables bad-pixel correction and
column/row fixed-pattern-noise correction to match `xiCamTool` defaults —
see the README's "Sensor Corrections" section before changing this behavior,
since disabling it silently reintroduces artifacts that corrupt downstream
image metrics. Similarly, see "Frame Buffer Management" in the README before
touching acquisition/buffering code — the default queue size (33 frames) can
return stale frames in autofocus-style workflows that change optics between
grabs.

## Requirements

- Python >= 3.8
- XIMEA Linux driver package (`libm3api.so.2`) installed system-wide — not a
  Python dependency, see the `optofly` repo's `install_ximea_driver.sh`
- [`uv`](https://docs.astral.sh/uv/) for package management
