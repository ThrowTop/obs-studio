cmake_minimum_required(VERSION 3.28...3.30)

include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/common/bootstrap.cmake" NO_POLICY_SCOPE)

project(obs-studio VERSION ${OBS_VERSION_CANONICAL})

# Global optimizations for MSVC (x64 only)
if(MSVC AND CMAKE_SIZEOF_VOID_P EQUAL 8)
  add_compile_options(
    /arch:AVX2
    /O2
    /GL
    /Ob3
    /Oi
    /Ot
    /GS-
    /Gy
    /Qpar
  )

  # KEEP DEFAULT /MD (do NOT force static CRT)

  # Link-time optimization
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELEASE ON)
  set(CMAKE_INTERPROCEDURAL_OPTIMIZATION_RELWITHDEBINFO ON)

  # Global linker flags
  set(CMAKE_EXE_LINKER_FLAGS_RELEASE "/LTCG /SAFESEH:NO")
  set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "/LTCG /SAFESEH:NO")
endif()

if(CMAKE_HOST_SYSTEM_NAME MATCHES "Windows")
  include("${CMAKE_CURRENT_SOURCE_DIR}/cmake/windows/architecture.cmake")
  if(NOT OBS_PARENT_ARCHITECTURE STREQUAL CMAKE_VS_PLATFORM_NAME)
    return()
  endif()
endif()

include(compilerconfig)
include(defaults)
include(helpers)

# Keep UI / Disable heavy subsystems
option(ENABLE_UI "Enable building with UI (requires Qt)" ON)
option(ENABLE_SCRIPTING "Enable scripting support" ON)
option(ENABLE_HEVC "Enable HEVC encoders" ON)

# Disable heavy plugins
option(ENABLE_BROWSER "Enable browser source" OFF)
option(ENABLE_VST "Enable VST plugins" OFF)
option(ENABLE_VLC "Enable VLC source" OFF)
option(ENABLE_IMAGE_MAGICK "Enable ImageMagick" OFF)
option(ENABLE_WEBSOCKET "Enable Websocket server" OFF)
option(ENABLE_AJA "Enable AJA output" OFF)

add_subdirectory(libobs)
if(OS_WINDOWS)
  add_subdirectory(libobs-d3d11)
  add_subdirectory(libobs-winrt)
endif()
add_subdirectory(libobs-opengl)
if(OS_MACOS)
  add_subdirectory(libobs-metal)
endif()
add_subdirectory(plugins)
add_subdirectory(test/test-input)
add_subdirectory(frontend)

message_configuration()
