# Nampower for Apple Silicon - Build (Windows x86 via MinGW)

### Prerequisites
- macOS or Linux
- [CMake], [Ninja], [mingw-w64] (toolchain i686)

### macOS (Homebrew):

```
brew install cmake ninja mingw-w64
```

### Configure
Note: this is a cross-build for Windows x86; do not use the system Clang/arm64 compilers.

```
cmake -S . -B build.nampower -G Ninja \
  -DCMAKE_SYSTEM_NAME=Windows \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_C_COMPILER=i686-w64-mingw32-gcc \
  -DCMAKE_CXX_COMPILER=i686-w64-mingw32-g++ \
  -DCMAKE_RC_COMPILER=i686-w64-mingw32-windres
```

### Build

```
cmake --build build.nampower --config Release -v
```

### Troubleshooting (udis86 headers)

The project includes pre-generated udis86 files (e.g., itab.h/.c, mnemonics.h/.c).
If after a clean clone you see errors like fatal error: 'itab.h' file not found, make sure these files exist in:

```
external/hadesmem/deps/udis86/udis86/libudis86/
```

### Installation
- Compile or grab the latest nampower.dll from https://github.com/fdotcico/nampower-rosetta/releases and place in the same directory as WoW.exe.
- Add nampower.dll to your dlls.txt (or your loader’s DLL list).
- (Optional) Install the helper addon into Interface/AddOns.

## Credits & Lineage

This project builds on prior work:
- Namreeb — original nampower for WoW 1.12.1 (BSD-2).
Upstream: https://github.com/namreeb/nampower
- Pepo — major rework adding spell queuing, quickcast, retries, and timing improvements.
Upstream: https://github.com/pepopo978/nampower

This fork focuses on cross-compiling (MinGW i686) and build/tooling tweaks while keeping Pepo’s behavior and fixes.

## Licenses
This repository is BSD-2-Clause (see LICENSE.txt). It vendors third-party code
(hadesmem, udis86, asmjit). Their original LICENSE files are included in-tree.
When distributing binaries, include these LICENSE files.

## Third-party licenses (vendored code)

This repository vendors third-party code. Their original copyrights and licenses are preserved in-tree.
When redistributing binaries (e.g. `nampower.dll`), include these licenses in your release package.

- **hadesmem** — vendored under `external/hadesmem/`  
  License: see `external/hadesmem/LICENSE` (and upstream repository)

- **udis86** — vendored under `external/hadesmem/deps/udis86/udis86/`  
  (includes pre-generated files like `itab.h/.c`, `mnemonics.h/.c`)  
  License: see `external/hadesmem/deps/udis86/udis86/LICENSE` (and upstream repository)

- **asmjit** — vendored under `external/hadesmem/deps/asmjit/asmjit/`  
  License: see `external/hadesmem/deps/asmjit/asmjit/LICENSE` or `LICENSE.md` (and upstream repository)


