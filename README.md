# Unity Essentials

This module is part of the Unity Essentials ecosystem and follows the same lightweight, editor-first approach.
Unity Essentials is a lightweight, modular set of editor utilities and helpers that streamline Unity development. It focuses on clean, dependency-free tools that work well together.

All utilities are under the `UnityEssentials` namespace.

```csharp
using UnityEssentials;
```

## Installation

Install the Unity Essentials entry package via Unity's Package Manager, then install modules from the Tools menu.

- Add the entry package (via Git URL)
    - Window → Package Manager
    - "+" → "Add package from git URL…"
    - Paste: `https://github.com/CanTalat-Yakan/UnityEssentials.git`

- Install or update Unity Essentials packages
    - Tools → Install & Update UnityEssentials
    - Install all or select individual modules; run again anytime to update

---

# Camera Luminance Calculator

> Quick overview: GPU-based average luminance measurement from a camera’s render target, using a compute shader with async GPU readback and a normalized 0–1 output.

Average scene luminance is computed from the camera’s target texture each frame via a compute shader and asynchronous GPU readback. The result is exposed as a normalized value through a `Luminance` property for downstream systems such as auto‑exposure, UI, or diagnostics.

![screenshot](Documentation/Screenshot.png)

## Features
- GPU-driven luminance
  - Compute shader reduction over the camera’s render target
  - Asynchronous GPU readback to avoid CPU stalls
- Non-blocking, incremental
  - A single `uint` buffer accumulates luminance; result is read back when ready
  - Automatic, per‑frame update cycle (calculate then check completion)
- Normalized output
  - `Luminance` property yields a clamped 0..1 average luminance
- Lightweight integration
  - Requires only a Camera with a `targetTexture`
  - No dependency on Post‑Processing

## Requirements
- Unity 6000.0+ (per package manifest)
- A Camera with a valid `RenderTexture` assigned to `camera.targetTexture`
- A compute shader resource available via Resources loading
  - Expected resource name: `UnityEssentials_Shader_CameraLuminance`
  - The shader must implement the `CalculateLuminance` kernel compatible with the component
- Platform support for Compute Shaders and Async GPU Readback
  - Some mobile/console platforms may restrict compute or async readback

## Usage
1) Add to a camera
   - Select your Camera and add `CameraLuminanceCalculator`
   - Ensure `camera.targetTexture` references a valid `RenderTexture`
2) Ensure the compute shader resource exists
   - Place the compute shader (named `UnityEssentials_Shader_CameraLuminance`) under a `Resources/` folder so it can be loaded
3) Consume luminance
   - Read `GetComponent<CameraLuminanceCalculator>().Luminance` each frame
   - Pair with `CameraAutoExposureController` to drive exposure automatically

## How It Works
- Initialization
  - On Awake, the compute shader is loaded from Resources and a 1‑element `ComputeBuffer<uint>` is created and bound
- Per‑frame loop
  - Update alternates between dispatching luminance calculation and checking the async readback status
  - If a `targetTexture` is not yet present, the component waits until one is assigned
- Compute and readback
  - The source texture is set and the kernel is dispatched over the image at reduced resolution
  - An `AsyncGPUReadback.Request` is issued for the 1‑value buffer; when complete, the packed luminance is read
  - Average luminance is reconstructed and normalized; the property is updated

## Notes and Limitations
- Requires `camera.targetTexture`; no on‑screen backbuffer read is attempted by this component
- The compute shader must match the expected interface (kernel name, scaling factor) and be placed in Resources
- Downsampling is used internally for performance; results represent an approximation
- Async readback availability and cost vary by platform and active render pipeline
- RenderTexture format, size, and MSAA settings can affect accuracy and performance
- The component resets and clamps the computed luminance into 0..1; interpret as relative brightness, not absolute lux

## Files in This Package
- `Runtime/CameraLuminanceCalculator.cs` – GPU luminance measurement and readback
- `Runtime/UnityEssentials.CameraLuminanceCalculator.asmdef` – Runtime assembly definition

## Tags
unity, camera, luminance, exposure, auto‑exposure, compute-shader, async-gpu-readback, rendertexture, hdrp, urp, runtime
