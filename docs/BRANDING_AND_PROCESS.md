# DRM-TRGRR — Branding & Facelift Process Notes

## Branding (current, authoritative)

- Company: **yeahwaah audio** (previously "yeahwaah" / "YEAHWAAH").
- Product name: **DRM-TRGRR** (previously "DRMTRGRR").
- Applies to all user-visible text: header title, logo/wordmark, settings/about, window title, plugin name shown in the DAW, docs, and release artifacts.
- The reference image still shows the old "DRMTRGRR" / "YEAHWAAH" wording. Layout, typography and style follow the image, but the wording follows this section.
- Do NOT change internal identifiers as part of branding: parameter IDs, state/ValueTree keys, plugin codes, C++ class names (`DrmtrgrrAudioProcessor`), CMake target names. Changing them breaks automation, presets and saved sessions.
- Open decision: CMake `PLUGIN_NAME`/`PRODUCT_NAME`/`COMPANY_NAME` and `getName()` are host-visible. Renaming them changes the plugin name in the DAW; the owner must confirm before changing.

## Process borrowed from DRM-SMRTVRB (https://github.com/elvicw/DRM-SMRTVRB)

- Reference image is the primary visual authority. Inspect it before any UI edit.
- Work on a dedicated task branch (never directly on `main`); no merging or force-pushing without owner approval.
- UI-only changes: keep DSP, parameter IDs/ranges/defaults, presets and state untouched; reuse existing attachments.
- Build, screenshot the real rendered UI, compare with the reference, refine, repeat. A passing build is not completion.
- Keep `docs/` handover notes, changelog and honest validation status (never claim DAW/OS testing that was not done).
- CI builds only on merged PRs (owner rule in DRM-SMRTVRB); do not trigger CI builds on unmerged branches.

## Build & release (DRM-SMRTVRB model)

- Local: `cmake -S . -B build -G Ninja -DCMAKE_BUILD_TYPE=Release && cmake --build build && ctest --test-dir build --output-on-failure`.
- Release is tag-driven: tag `vX.Y.Z` must equal the CMake `project(... VERSION X.Y.Z)`; a workflow builds, tests, packages `<name>-<version>-<OS>-VST3.zip` + `.sha256` (with LICENSE and SOURCE.txt), and uploads it as an artifact.
- The GitHub Release itself is published manually by the owner, with notes and screenshots.
- Version bump happens on a `release/x.y.z` branch; after merge, tag, wait for the package workflow, then publish.

## Facelift implementation notes

- UI is laid out in a 1536x1024 design space (`hw::kDesignWidth/Height`) and scaled to the window with a fixed 3:2 aspect ratio.
- New files: `Source/UI/HardwareControls.*` (knob, lamp/key buttons, value pad, chassis drawing) and a warm `SlateLookAndFeel`.
- Known differences from the reference image: Detail/Sensitivity read in dB/% of the existing parameters rather than the mock's %; RETRIG and DYNAMICS are drag pads bound to the continuous `retrigger` / `dynamics` parameters; the preset box has Browser/Settings buttons beneath it; per-slot load/factory/hit/advanced controls live in the slot "waveform click" menu; gate/bleed enable LEDs sit by the panel titles; MIDI note/channel open from the MIDI OUT button.
- Parameter IDs, state keys and DSP are unchanged. Host-visible CMake names (`PLUGIN_NAME`, `PRODUCT_NAME`) are intentionally left as `DRMTRGRR` pending owner confirmation.
- Local build on macOS 15+ SDKs: JUCE 7.0.12's `juceaide` fails to compile unless built with an older deployment target. Use `CXXFLAGS=-mmacosx-version-min=11.0 OBJCXXFLAGS=-mmacosx-version-min=11.0 CFLAGS=-mmacosx-version-min=11.0 MACOSX_DEPLOYMENT_TARGET=11.0 cmake -B build -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_OSX_DEPLOYMENT_TARGET=11.0` (after removing `build/libs/JUCE/tools`).
- Screenshot: `./build/generate_screenshot` writes a 1:1 1536x1024 `assets/screenshot.png`.

## Release conventions (reuse for next projects)

- **Effects-list name = `NAME` + `VERSION`**, no separator, e.g. `DRM-TRGRR1.1.0`. Set via `PRODUCT_NAME "DRM-TRGRR${PROJECT_VERSION}"` in `juce_add_plugin` (JUCE derives the plugin name from it); `getName()` returns the same. Same pattern as DRM-SMRTVRB. Bump `project(... VERSION x.y.z)` and the artefact names in `.github/workflows/build.yml` together.
- **Stable identity never changes**: `PLUGIN_CODE` (DTrg), `PLUGIN_MANUFACTURER_CODE` (Ywah), `BUNDLE_ID`, parameter IDs.
- **This release is Windows-only** (VST3 + Standalone). macOS/AU is skipped; re-enable by adding AU back to `PLUGIN_FORMATS` under `if(APPLE)`.
- Release = tag `vX.Y.Z` matching the CMake version; workflow zips the plugin and standalone as the release zip.
- **Release zip name = `NAME.version-Win64.zip`**, e.g. `DRM-TRGRR.1.1.0-Win64.zip` (note the dot between name and version, unlike the plugin name which has none). Keep the same pattern for every project; the version must match the CMake version and tag. The workflow artifact name matches the zip name.
- **UI shows the version**: a tiny `vX.Y.Z` tag under the product title (header), taken from `JucePlugin_VersionString` so it always matches the CMake version. Do the same in every project.

## Accounts and workflow (reuse for next projects)

- **`elvicw89` owns the public repos** (`DRMTRGRR`, `DRM-SMRTVRB`, `DRM-RESHAPE`, and the `DRM-Plugins` download hub). Builds and releases happen there. Keep them public: the plugins use JUCE under its open-source licence.
- **`elvicw` is the developer account.** Fork the repo from `elvicw89`, work on a task branch in the fork, open a PR into `elvicw89`, and merge it there. Local remotes: `origin` = the `elvicw` fork, `upstream` = `elvicw89`.
- **Releases:** tag `vX.Y.Z` on `elvicw89`'s `main`; the Windows workflow builds and publishes `NAME.version-Win64.zip`.
- **Hub (`elvicw89/DRM-Plugins`)** auto-discovers repos with the `drm-series` topic under `elvicw89` and copies each latest release zip and screenshot into the hub daily (and on manual run). Tag every new DRM plugin repo with `drm-series`.
