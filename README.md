# Build (Windows x86 via MinGW)

### Prerequisites
- macOS or Linux
- [CMake], [Ninja], [mingw-w64] (toolchain i686)
- macOS (Homebrew)

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

### Build
The project includes pre-generated udis86 files (e.g., itab.h/.c, mnemonics.h/.c).
If after a clean clone you see errors like fatal error: 'itab.h' file not found, make sure these files exist in:

```
external/hadesmem/deps/udis86/udis86/libudis86/
```

### Installation
- Grab the latest nampower.dll from https://github.com/fdotcico/nampower-rosetta/releases and place in the same directory as WoW.exe.
- Add nampower.dll in dlls.txt
- You can also get the helper addon and place that in Interface/Addons.

## Credits & Lineage

This project is a fork of earlier work by:
- **Namreeb** — original *nampower* for WoW 1.12.1.  
  Repository: https://github.com/namreeb/nampower (or the upstream you used)
- **Pepo** — major rework adding spell queuing, quickcast and other features.  
  Repository: https://github.com/pepopo978/nampower

This fork focuses on cross-compiling and build tooling improvements while keeping Pepo’s behavior and fixes.

## Third-party notices (vendored code)

This repository vendors third-party code. Their original copyrights and licenses are preserved in-tree.
When redistributing binaries (e.g. `nampower.dll`), include these licenses in your release package.

- **hadesmem** — vendored under `external/hadesmem/`  
  License: see `external/hadesmem/LICENSE` (and upstream repository)

- **udis86** — vendored under `external/hadesmem/deps/udis86/udis86/`  
  (includes pre-generated files like `itab.h/.c`, `mnemonics.h/.c`)  
  License: see `external/hadesmem/deps/udis86/udis86/LICENSE` (and upstream repository)

- **asmjit** — vendored under `external/hadesmem/deps/asmjit/asmjit/`  
  License: see `external/hadesmem/deps/asmjit/asmjit/LICENSE` or `LICENSE.md` (and upstream repository)
