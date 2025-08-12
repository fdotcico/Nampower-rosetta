cmake_minimum_required(VERSION 3.24)
project(nampower_dll_only LANGUAGES C CXX)

# Percorso hadesmem vendorizzato
set(HADESMEM_ROOT "${CMAKE_SOURCE_DIR}/../external/hadesmem")

# --- udis86: compila la libreria C dai sorgenti in libudis86 ---
file(GLOB UDIS86_SRC "${HADESMEM_ROOT}/deps/udis86/udis86/libudis86/*.c")
add_library(udis86 STATIC ${UDIS86_SRC})
target_include_directories(udis86 PUBLIC
  "${HADESMEM_ROOT}/deps/udis86/udis86"
  "${HADESMEM_ROOT}/deps/udis86/udis86/libudis86"
)

# --- Progetto originale (definisce il target 'nampower') ---
add_subdirectory(../nampower ${CMAKE_BINARY_DIR}/nampower)

# Include necessari anche per i .cpp di nampower che includono hadesmem/udis86
target_include_directories(nampower PRIVATE
  "${HADESMEM_ROOT}/include/memory"
  "${HADESMEM_ROOT}/deps/asmjit/asmjit/src"
  "${HADESMEM_ROOT}/deps/udis86/udis86"
  "${HADESMEM_ROOT}/deps/udis86/udis86/libudis86"
  "/opt/homebrew/include" # Boost headers di Homebrew
)

# Evita errori pedanti del compilatore GNU su codice pensato per MSVC
target_compile_options(nampower PRIVATE -fpermissive)

# Link: udis86 + classiche Win32 che spesso servono con hadesmem
target_link_libraries(nampower PRIVATE udis86 rpcrt4 dbghelp psapi ws2_32)
