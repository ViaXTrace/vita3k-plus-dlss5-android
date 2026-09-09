# Vita3K Android — no-cost neural-style rendering specification

Status: first Android research slice, based on Vita3K Plus.

## Scope and terminology

The reference project [1-Click-DLSS5](https://github.com/reiluisii/1-Click-DLSS5)
is a Windows launcher/injector. Its feature model is useful as a product
reference, but it is not the NVIDIA DLSS 5 SDK and it cannot be copied into an
Android emulator as-is.

NVIDIA describes DLSS 5 as 3D-Guided Neural Rendering. The official pipeline
uses the rendered color frame and motion vectors, then applies a proprietary
neural model as a final rendering stage. The public Windows reference project
organizes its workflow around:

- title and API detection;
- per-title profiles and automatic mode selection;
- direct native-DLSS interception;
- an OptiScaler bridge for FSR2/XeSS titles;
- a universal feeder for titles without an upscaler;
- optical-flow or motion estimation;
- live status and performance feedback;
- clean restoration of the original files/settings.

Those concepts are adapted here as renderer-level concepts, not as DLL
injection. Android does not provide the Windows DLL/Streamline/ReShade
interception path, and NVIDIA's proprietary DLSS-NR model is not redistributed
by this project.

## No-cost implementation boundary

The first version uses only code that can be built from this public repository
and original shader source:

1. Keep Vita3K Plus's existing Vulkan presentation path.
2. Add an opt-in `NSS (Experimental)` screen filter.
3. Use a shader-only spatial pass with local luminance, edge direction,
   adaptive reconstruction, and restrained sharpening.
4. Preserve Nearest, Bilinear, Bicubic, FXAA, and FSR as fallbacks.
5. Require no cloud inference, paid API, subscription, proprietary NVIDIA
   binary, or runtime network access.
6. Never label the approximation as NVIDIA DLSS or imply NVIDIA endorsement.

This gives Android users a useful no-cost post-processing experiment while
being technically and legally honest: it is not equivalent to DLSS 5's
temporal neural model.

## Android feature map

| Windows reference concept | Android implementation |
| --- | --- |
| Direct native DLSS injection | Renderer-level post-process filter |
| OptiScaler bridge | Backend capability and filter selection |
| Universal feeder | Final-frame filter when no temporal inputs exist |
| Game discovery | Existing Vita3K title/configuration storage |
| API detection | Existing renderer/backend checks |
| Optical-flow calibration | Future motion-vector/frame-history input |
| HUD/live status | Future debug overlay with frame time and GPU cost |
| Clean restoration | One setting back to an existing filter |

## First-release controls

The initial release exposes the filter as a normal Vulkan screen-filter choice.
The next controls should be added only after measurements on real Android GPUs:

- enable/disable;
- reconstruction strength;
- sharpening strength;
- exposure and tone response;
- aspect-ratio preservation;
- per-game profiles;
- debug views for original, reconstructed, edge confidence, and difference;
- automatic fallback after shader or pipeline failure;
- frame-time and GPU-cost reporting.

## Roadmap

### v0.1

- Vita3K Plus Android base;
- Vulkan NSS shader pass;
- existing filters preserved;
- public GitHub Actions debug/reldebug APK build;
- tagged release notes identifying NSS as an experimental approximation.

### v0.2

- frame history with motion rejection;
- optional motion-vector input where Vita3K can provide it;
- per-game profile fields;
- on-screen cost overlay.

### v0.3

- benchmark a small, redistributable local model on representative Adreno,
  Mali, and Immortalis devices;
- prefer Vulkan compute or Android GPU delegate only if latency, thermal
  behavior, package size, and license are acceptable;
- ship no model unless all of those gates pass.

## Validation gates

- Vulkan devices that support the existing screen-filter pipeline can select NSS.
- Unsupported or failing devices retain the existing fallback filters.
- The APK runs without an external service or paid account.
- GitHub Actions builds the public reldebug artifact and creates a release for
  version tags.
- Release notes explicitly distinguish NSS from NVIDIA DLSS.