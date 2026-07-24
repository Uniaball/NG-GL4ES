# Krypton Wrapper (NG-GL4ES)

[![Build NG-GL4ES for Android](https://github.com/Uniaball/NG-GL4ES/actions/workflows/main.yml/badge.svg)](https://github.com/Uniaball/NG-GL4ES/actions/workflows/main.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Release](https://img.shields.io/github/v/release/Uniaball/NG-GL4ES)](https://github.com/Uniaball/NG-GL4ES/releases)

**Krypton Wrapper** *(aka NG-GL4ES)* is an OpenGL-to-OpenGL ES translation layer, forked from [gl4es](https://github.com/ptitSeb/gl4es) and [gl4es-114-extra](https://github.com/PojavLauncherTeam/gl4es-114-extra), with modern enhancements targeting Android.

## Features

1. Capability to handle more advanced shaders (GLSL → SPIR-V → ESSL via glslang & SPIRV-Cross);
2. Capability to run Minecraft in almost all versions;
3. More advanced OpenGL features on GLES hardware.

## Project Structure

```
NG-GL4ES/
├── CMakeLists.txt        # CMake build entry
├── include/              # Public headers (GL/EGL/GLES/KHR, version.h, ...)
├── src/
│   ├── gl/               # Core translation layer
│   └── glx/              # GLX / platform glue
├── 3rdparty/             # Submodules: glslang, SPIRV-Cross
├── external/             # MetalANGLE.framework (macOS/iOS)
├── docs/                 # COMPILE.md (build guide), USAGE.md (env vars)
├── packaging/debian/     # Debian packaging files
└── tools/                # Spec-based code generation tools
```

## Building

### Android (NDK)

```bash
git clone https://github.com/Uniaball/NG-GL4ES.git
cd NG-GL4ES
git submodule update --init --recursive

cmake -B build \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=android-29 \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK_HOME/build/cmake/android.toolchain.cmake \
  -DANDROID_TOOLCHAIN=clang

cmake --build build --config Release --parallel
```

The output library is `build/libng_gl4es.so`.

For other platforms and detailed options, see [docs/COMPILE.md](docs/COMPILE.md).
For runtime environment variables, see [docs/USAGE.md](docs/USAGE.md).

## Change Log

See [CHANGELOG.md](CHANGELOG.md) and [Releases](https://github.com/Uniaball/NG-GL4ES/releases).

## Credits & Upstream

This project stands on the shoulders of:

- [ptitSeb/gl4es](https://github.com/ptitSeb/gl4es) — the original OpenGL-to-GLES translation layer (MIT License)
- [BZLZHH/NG-GL4ES](https://github.com/BZLZHH/NG-GL4ES) — the direct upstream of this fork (MIT License)
- [PojavLauncherTeam/gl4es-114-extra](https://github.com/PojavLauncherTeam/gl4es-114-extra) (MIT License)
- Some code is from MobileGlues.

### Third-party components

- **glslang** by KhronosGroup — [Various Licenses](https://github.com/KhronosGroup/glslang/blob/main/LICENSE.txt) — [github](https://github.com/KhronosGroup/glslang)
- **SPIRV-Cross** by KhronosGroup — [Apache License 2.0](https://github.com/KhronosGroup/SPIRV-Cross/blob/master/LICENSE) — [github](https://github.com/KhronosGroup/SPIRV-Cross)
- **cJSON** by DaveGamble — [MIT License](https://github.com/DaveGamble/cJSON/blob/master/LICENSE) — [github](https://github.com/DaveGamble/cJSON)

## Sponsor

**ptitSeb (gl4es)**: [paypal](https://paypal.me/0ptitSeb)

## License

MIT License — see [LICENSE](LICENSE).
