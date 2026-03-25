# ximea

Python wrapper for the [XIMEA](https://www.ximea.com/) camera xiApi SDK.

Provides access to XIMEA cameras for image acquisition, parameter control, and image data retrieval.

## Requirements

- Python >= 3.8
- XIMEA Linux software package (`libm3api.so.2` must be installed)
- [uv](https://docs.astral.sh/uv/) for package management

## Installation

```bash
uv add /path/to/ximea-py
```

With numpy support for array-based image data:

```bash
uv add "/path/to/ximea-py[numpy]"
```

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

## Package Contents

| Module | Description |
|--------|-------------|
| `ximea.xiapi` | `Camera` and `Image` classes wrapping the C xiApi |
| `ximea.xidefs` | Constants, error codes, and parameter definitions |
| `ximea.xi_wintypes` | Windows type compatibility mappings via ctypes |

## License

MIT
