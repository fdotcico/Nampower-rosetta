cmake_minimum_required(VERSION 3.16)

project(nampower_rosetta LANGUAGES C CXX)

# C++14 come l'originale
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_DISABLE_SOURCE_CHANGES ON)
set(CMAKE_DISABLE_IN_SOURCE_BUILD ON)

# Per default NON costruiamo il loader (richiede MSVC + Boost)
option(BUILD_LOADER "Build the GUI/loader (requires MSVC + Boost)" OFF)

# Percorso hadesmem (submodule)
set(HADESMEM_ROOT "${CMAKE_SOURCE_DIR}/external/hadesmem" CACHE PATH "Path to hadesmem root")

# Verifica hadesmem
if(NOT EXISTS "${HADESMEM_ROOT}/include/memory/hadesmem")
  message(FATAL_ERROR "hadesmem non trovato in: ${HADESMEM_ROOT}/include/memory/hadesmem
Hai inizializzato i submodule? Esegui:  git submodule update --init --recursive")
endif()

# Boost: ci servono solo gli header (Boost.Exception)
find_path(BOOST_INCLUDE_DIR
  NAMES boost/exception/all.hpp
  HINTS $ENV{BOOST_ROOT} /opt/homebrew /usr/local /usr
  PATH_SUFFIXES include
)
if(NOT BOOST_INCLUDE_DIR)
  message(FATAL_ERROR "Boost headers non trovati. Installa con: brew install boost")
endif()
message(STATUS "Boost headers: ${BOOST_INCLUDE_DIR}")

# ------------ udis86 (deps di hadesmem) ------------
set(UDIS86_TOP    "${HADESMEM_ROOT}/deps/udis86/udis86")
set(UDIS86_LIBDIR "${UDIS86_TOP}/libudis86")

if(NOT EXISTS "${UDIS86_TOP}/udis86.h" OR NOT EXISTS "${UDIS86_LIBDIR}")
  message(FATAL_ERROR "Struttura udis86 non trovata sotto hadesmem. Hai fatto --recursive?")
endif()

file(GLOB UDIS86_SRC "${UDIS86_LIBDIR}/*.c")
add_library(udis86 STATIC ${UDIS86_SRC})
target_include_directories(udis86 PUBLIC
  "${UDIS86_TOP}"     # per udis86.h
  "${UDIS86_LIBDIR}"  # per libudis86/*.h (es. itab.h, types.h)
)
set_property(TARGET udis86 PROPERTY C_STANDARD 99)

# ------------ DLL di Nampower ------------
add_subdirectory(nampower)

if(TARGET nampower)
  target_include_directories(nampower PRIVATE
    "${CMAKE_CURRENT_SOURCE_DIR}/nampower/Include"
    "${CMAKE_CURRENT_SOURCE_DIR}/nampower"
    "${HADESMEM_ROOT}/include/memory"
    "${HADESMEM_ROOT}/deps/asmjit/asmjit/src"
    "${BOOST_INCLUDE_DIR}"
    "${UDIS86_TOP}"      # include per udis86.h
    "${UDIS86_LIBDIR}"   # include per itab.h & co.
  )

  target_compile_definitions(nampower PRIVATE
    UNICODE _UNICODE _SCL_SECURE_NO_WARNINGS _CRT_SECURE_NO_WARNINGS
    ASMJIT_STATIC ASMJIT_BUILD_X86 HADESMEM_NO_PUGIXML
  )

  # MinGW: rilassiamo alcuni errori e linkiamo statici libgcc/libstdc++ e winpthread
if(MINGW)
  # forza pthread statico
  execute_process(
    COMMAND ${CMAKE_CXX_COMPILER} -print-file-name=libwinpthread.a
    OUTPUT_VARIABLE WINPTHREAD_STATIC
    OUTPUT_STRIP_TRAILING_WHITESPACE
  )
  if (WINPTHREAD_STATIC AND EXISTS "${WINPTHREAD_STATIC}")
    message(STATUS "Linking static winpthread: ${WINPTHREAD_STATIC}")
    target_link_libraries(nampower PRIVATE "${WINPTHREAD_STATIC}")
  else()
    message(WARNING "libwinpthread.a non trovata, uso DLL winpthread")
    target_link_libraries(nampower PRIVATE winpthread)
  endif()

  target_link_options(nampower PRIVATE -static-libgcc -static-libstdc++)
  target_compile_options(nampower PRIVATE -fpermissive)
endif()

  # Link al nostro udis86 e alle Win32 libs (niente -lasmjit/-ludis86 esterni)
  target_link_libraries(nampower PRIVATE
    udis86
    shlwapi
    kernel32 user32 gdi32 winspool shell32 ole32 oleaut32 uuid comdlg32 advapi32
  )

  # Uscita come "nampower.dll" (senza prefisso lib)
  set_target_properties(nampower PROPERTIES PREFIX "" OUTPUT_NAME "nampower")
endif()

# Loader opzionale (solo con MSVC)
if(BUILD_LOADER AND MSVC)
  add_subdirectory(loader)
endif()

# Install di file testo (opzionale)
install(FILES LICENSE.txt README.md DESTINATION "${CMAKE_INSTALL_PREFIX}")