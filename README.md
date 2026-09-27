# Capture

Portable live-camera library. Enumerates OS cameras, opens one or more at a
time, and returns the latest frame as straight-alpha RGBA8 (top-down).

This is a standalone CMake project (`Capture::Capture`). It has no Sneeze,
ENGINE, SDL, OpenCV, or Ocean dependency. Each OS uses its native capture
API. Sneeze consumes it through a thin engine shim that owns lifetime and
routes logs to `ENGINE::Log`. Other hosts can `#include <Capture/Capture.h>`
directly.

## Platforms

| OS | Backend | Notes |
|----|---------|-------|
| Windows | Media Foundation (`IMFSourceReader`) | Prefers RGB32, falls back to NV12 / YUY2 |
| macOS / iOS | AVFoundation | 32BGRA sample buffers |
| Linux | V4L2 | Prefers YUYV 640x480, then NV12 / RGB24 |
| Android | Camera2 NDK (`AImageReader`) | YUV_420_888. Covers Quest 3 as well |

Quest 3 and Quest 3S run the Android backend. Passthrough cameras appear in
Camera2 only after the host holds both `android.permission.CAMERA` and
`horizonos.permission.HEADSET_CAMERA`. `Device_Intrinsics` fills the pinhole
6-tuple (fx, fy, cx, cy, size, EXIF orientation) when the OS supplies
calibration. OpenXR environment blend is a separate path and is not a Capture
device.

The embedding application still declares OS camera permissions (Windows
privacy prompt, iOS `NSCameraUsageDescription`, Android `CAMERA` /
`HEADSET_CAMERA`, macOS camera entitlement). This library cannot do that.

## API

```cpp
#include <Capture/Capture.h>

Capture::CAPTURE capture;
capture.Initialize ();

int nCount = capture.Device_Count ();
std::string sName = capture.Device_Name (0);

if (capture.Device_Open (0))
{
   Capture::CAPTURE::INTRINSICS Intrinsics;
   capture.Device_Intrinsics (0, Intrinsics);

   int nWidth = 0, nHeight = 0;
   uint64_t nFrameIx = 0;
   std::vector<uint8_t> aRgba;
   if (capture.Frame_Latest (0, nWidth, nHeight, aRgba, nFrameIx))
   {
      // straight RGBA8, top-down, A=255
   }
   capture.Device_Close (0);
}
```

`Device_Open` is refcounted. `Frame_Latest` returns false until the first
sample arrives. `nFrameIx` increments per new sample.

Optional logging:

```cpp
void OnLog (Capture::eLOG eLevel, const char* szMessage, void* pUser);
Capture::CAPTURE capture (OnLog, pUser);
```

## Build

```
cmake -S . -B build -DCMAKE_INSTALL_PREFIX=install
cmake --build build --config Release
cmake --install build --config Release
```

Headers install to `include/Capture/`. The static lib is `Capture.lib` /
`libCapture.a`. CMake consumers use the imported target `Capture::Capture`.

## License

Apache 2.0. See [LICENSE](LICENSE).
