# ximea

Python wrapper for the [XIMEA](https://www.ximea.com/) camera xiApi SDK.

Provides access to XIMEA cameras for image acquisition, parameter control, and image data retrieval.

## Requirements

- Python >= 3.8
- XIMEA Linux software package (`libm3api.so.2` must be installed)
- [uv](https://docs.astral.sh/uv/) for package management

## Installation

```bash
uv add git+https://github.com/mpinb/ximea-py
```

With numpy support for array-based image data:

```bash
uv add "ximea[numpy] @ git+https://github.com/mpinb/ximea-py"
```

### Development

```bash
git clone https://github.com/elhananby/ximea-py.git
cd ximea-py
uv sync --group dev
```

This installs `pytest`, `ruff`, and `numpy` for development.

## Usage

```python
from ximea import Camera, Image

cam = Camera()
cam.open_device()
cam.set_exposure(10000)

img = Image()
cam.start_acquisition()
cam.get_image(img)
data = img.get_image_data_numpy()
cam.stop_acquisition()

cam.close_device()
```

### Sensor Corrections

As of the latest version, `open_device()` automatically enables sensor corrections
(bad pixel correction, column/row fixed-pattern noise correction) to match xiCamTool
defaults. To disable them after opening:

```python
cam.disable_bpc()
cam.set_column_fpn_correction("XI_OFF")
cam.set_row_fpn_correction("XI_OFF")
```

### Frame Buffer Management

The default buffer queue holds up to 33 frames (~825 MB at 5060x5060). In
applications that change optical parameters between grabs (e.g. autofocus sweeps),
stale frames from the queue can be returned instead of fresh captures. To avoid this:

```python
cam.open_device()
cam.enable_recent_frame()       # prefer newest buffered frame
cam.set_buffers_queue_size(2)   # keep queue small to limit staleness
cam.start_acquisition()
```

With a queue size of 2, discarding one frame before each measurement guarantees a
fresh capture that reflects the current camera/optics state.

See the [`examples/`](examples/) directory for more:

| Example | Description |
|---------|-------------|
| [basic_capture.py](examples/basic_capture.py) | Single image capture |
| [device_discovery.py](examples/device_discovery.py) | Find cameras and read device info |
| [continuous_acquisition.py](examples/continuous_acquisition.py) | Capture frames in a loop |
| [format_and_roi.py](examples/format_and_roi.py) | Set image format, ROI, exposure, and gain |
| [auto_exposure_wb.py](examples/auto_exposure_wb.py) | Auto exposure/gain and white balance |
| [hardware_trigger.py](examples/hardware_trigger.py) | External hardware trigger |
| [temperature.py](examples/temperature.py) | Read sensor temperature |
| [error_handling.py](examples/error_handling.py) | Proper error handling with `Xi_error` |
| [benchmark_fps.py](examples/benchmark_fps.py) | Benchmark maximum framerate with a circular buffer |
| [triggered_capture.py](examples/triggered_capture.py) | Triggered capture with pre/post-trigger circular buffer and GPU encoding |

## Package Contents

| Module | Description |
|--------|-------------|
| `ximea.xiapi` | `Camera` and `Image` classes wrapping the C xiApi |
| `ximea.xidefs` | Constants, error codes, and parameter definitions |
| `ximea.xi_wintypes` | Windows type compatibility mappings via ctypes |

## License

GPLv3
