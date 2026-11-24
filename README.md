# OpenComposite community fork (OpenXR)

A community fork of [OpenComposite](https://gitlab.com/znixian/OpenOVR/-/tree/openxr), translating OpenVR calls to OpenXR so compatible games can run through an OpenXR runtime.

This fork adds newer OpenVR interface compatibility and standard OpenXR eye-gaze input. It is an independent, experimental fork. Credit for the original project belongs to Campbell Suter (ZNix) and the upstream contributors; their commit history and license notices are preserved.

> **torsteinvh/OpenComposite** is a personal fork of [ClarkCheekyKent/OpenComposite](https://github.com/ClarkCheekyKent/OpenComposite). It adds a fix for apps that abort with `Hit stubbed file at ...BaseOverlay.cpp:819 func CreateDashboardOverlay`, and builds `openvr_api.dll` automatically with GitHub Actions. Everything else is inherited unchanged from ClarkCheekyKent's fork.

## Changes in this fork

- OpenVR 2.15.6 headers and registrations for `IVRCompositor_029`, `IVRSystem_026`, `IVROverlay_028`, and `IVRInput_011`.
- Eye-tracking actions through `XR_EXT_eye_gaze_interaction`, alongside existing controller and skeletal input.
- Inactive eye data when the headset or runtime does not expose gaze support.
- Overlay curvature values can be set and read; rendering remains flat OpenXR quads.

Added in torsteinvh/OpenComposite:

- **Dashboard overlays no longer abort the app.** OpenComposite has no SteamVR dashboard, and `CreateDashboardOverlay` used to stop the app with a "Hit stubbed file" error. It now creates the dashboard and thumbnail overlays as ordinary overlays that are never shown (`IsDashboardVisible()` is always `false`). The related calls `IsActiveDashboardOverlay`, `SetDashboardOverlaySceneProcess`, `GetDashboardOverlaySceneProcess`, `ShowDashboard` and `GetPrimaryDashboardDevice` return harmless defaults instead of aborting.
- **Raw and file overlays no longer abort the app.** `SetOverlayRaw` and `SetOverlayFromFile` are accepted and logged once instead of stopping the app with a "Hit stubbed file" error. The image is not rendered: OpenComposite only renders overlays from GPU textures (`SetOverlayTexture`). Apps typically use these calls for small icons such as dashboard thumbnails.
- **Automatic Windows builds.** The [Build openvr_api.dll (Windows)](.github/workflows/build-windows.yml) workflow builds x64 and Win32 DLLs on every push to `main`.

See [interface support and limitations](dist/interface-support.md) for details. Interface registration does not imply every method is implemented; some inherited unsupported calls can still abort.

## Download and install (Windows)

The easiest way is to download a zip for your game's architecture (x64 for 64-bit games, x86 for 32-bit games) from this fork's [Releases](https://github.com/torsteinvh/OpenComposite/releases). To make a new release, run the [Build openvr_api.dll (Windows)](https://github.com/torsteinvh/OpenComposite/actions/workflows/build-windows.yml) workflow with **Run workflow** and enter a version such as `v0.2.0`. Release notes are taken from `dist/release-notes/<version>.md` if that file exists.

The newest build of `main` is also available as GitHub Actions artifacts:

1. Open the [Actions page](https://github.com/torsteinvh/OpenComposite/actions/workflows/build-windows.yml) and select the latest successful run.
2. Under **Artifacts**, download `openvr_api-x64` for 64-bit games (most current VR games) or `openvr_api-Win32` for 32-bit games. You must be signed in to GitHub to download artifacts, and GitHub deletes them after 90 days; push a commit or run the workflow manually (**Run workflow**) to get a fresh build.

Then install it per game:

1. Close the game and back up its original `openvr_api.dll`, for example by renaming it to `openvr_api.dll.bak`.
2. Extract the downloaded zip.
3. Replace the game's DLL with the extracted `openvr_api.dll`.
4. Select your headset's OpenXR runtime and start the game. Restore the original DLL to undo the change.

Upstream's launcher and download services distribute upstream builds. Use this fork's releases or build artifacts to obtain these changes.

## Validation and known limitations

Windows Release x64 and x86 builds succeed in GitHub Actions. The dashboard-overlay fix has been confirmed to get past `CreateDashboardOverlay` in a game; the `SetOverlayRaw`/`SetOverlayFromFile` change has so far only been verified to compile. The focused gaze test uses simulated OpenXR responses to exercise inactive tracking, position and direction, requested timestamps, tracking loss, and runtime errors.

No Man's Sky and the Pimax Crystal Super OLED have not been tested with this build. Linux clock conversion is implemented but has not been compiled or tested in this Windows workspace.

Standard OpenXR gaze supplies origin and direction without fixation depth. `vGazeTarget` is one metre along the gaze ray. Raw tracking uses stage space as an approximation; relative timing uses clock conversion where available and otherwise the latest frame time.

## Build from source

Clone this repository with `git clone --recurse-submodules https://github.com/torsteinvh/OpenComposite.git`. Existing clones should run `git submodule update --init --recursive`.

Windows prerequisites: Visual Studio C++ tools with a Windows SDK, CMake, Python 3, and the Vulkan headers/import libraries under `libs/vulkan` as described in the [upstream build documentation](doc/upstream-readme.md#windows-specific). The Vulkan SDK is not included in this repository. The GitHub Actions workflow downloads the same Vulkan mini SDK that upstream's AppVeyor build uses, and falls back to the official LunarG SDK if that download fails.

From a Visual Studio developer shell, using the Visual Studio generator installed on your system:

```powershell
cmake -S . -B build/windows-x64 -A x64
cmake --build build/windows-x64 --config Release --target OCOVR
cmake -S . -B build/windows-x86 -A Win32
cmake --build build/windows-x86 --config Release --target OCOVR
```

The DLLs are `build/windows-x64/bin/Release/vrclient_x64.dll` and `build/windows-x86/bin/Release/vrclient.dll`. Rename the matching DLL to `openvr_api.dll` for per-game installation. For Linux instructions and general configuration, see the [preserved upstream README](doc/upstream-readme.md).

### Focused gaze test

The standalone test needs CMake, a C++20 compiler, Python 3, and the OpenXR SDK submodule. It does not require a headset or Vulkan SDK.

```powershell
cmake -S tests -B build/gaze-tests
cmake --build build/gaze-tests --config Release
ctest --test-dir build/gaze-tests -C Release --output-on-failure
```

## Reporting issues and contributing

For problems with the dashboard fix or the automatic builds, open an issue in this repository. For everything else, see [ClarkCheekyKent/OpenComposite](https://github.com/ClarkCheekyKent/OpenComposite). Include the game, headset, OpenXR runtime and version, build used, and reproduction steps. Include relevant log excerpts after removing personal information. Contributions and hardware test reports are welcome; ongoing maintenance is on a best-effort basis.

## License and credits

The project retains the upstream [GNU GPLv3 license](LICENSE.txt) and third-party notices. Each build artifact is built from the commit shown on its Actions run, and that commit is the corresponding source; build instructions and dependency submodules are included in that source. See the individual license files for bundled components.
