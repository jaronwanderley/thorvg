cmake_minimum_required(VERSION 3.10)
project(ThorVG LANGUAGES CXX)

# --- 1. AUTO-GENERATE CONFIG.H ---
set(THORVG_VERSION_STRING "0.14.1")

file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/config.h"
"#ifndef THORVG_CONFIG_H
#define THORVG_CONFIG_H
#define THORVG_VERSION_STRING \"${THORVG_VERSION_STRING}\"
#define THORVG_FILE_IO_SUPPORT 1
#define THORVG_TTF_LOADER_SUPPORT 1
#define THORVG_SW_RASTER_SUPPORT 1
#define THORVG_TVG_LOADER_SUPPORT 1

#endif
")

# --- 2. CONFIGURATION OPTIONS ---
option(THORVG_ENGINE_SW "Enable Software Rasterizer (CPU)" ON)
option(THORVG_ENGINE_GL "Enable OpenGL Engine" OFF)
option(THORVG_LOADER_SVG "Enable SVG Loader" ON)
option(THORVG_LOADER_LOTTIE "Enable Lottie Loader" ON)
option(THORVG_LOADER_TVG "Enable Native TVG Loader" ON)
option(THORVG_LOADER_RAW "Enable Raw Loader" ON)
option(THORVG_LOADER_TTF "Enable TTF Loader" ON)

# --- 3. COMPILER SETTINGS ---
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

# --- 4. SOURCE COLLECTION ---
set(THORVG_SOURCES "")

# Core: Common (Recursive)
file(GLOB_RECURSE TVG_COMMON_SOURCES "src/common/*.cpp")
list(APPEND THORVG_SOURCES ${TVG_COMMON_SOURCES})

# Core: Renderer Base (NOT Recursive)
file(GLOB TVG_RENDERER_SOURCES "src/renderer/*.cpp")
list(APPEND THORVG_SOURCES ${TVG_RENDERER_SOURCES})

# --- 5. MODULE COLLECTION (Sources + Includes) ---

# List to store extra private include directories
set(TVG_PRIVATE_INCLUDES "")

# Engine: Software Rasterizer
if(THORVG_ENGINE_SW)
    file(GLOB_RECURSE ENGINE_SW_SOURCES "src/renderer/sw_engine/*.cpp")
    list(APPEND THORVG_SOURCES ${ENGINE_SW_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/sw_engine")
    add_compile_definitions(THORVG_SW_RASTER_SUPPORT)
endif()

# Engine: OpenGL
if(THORVG_ENGINE_GL)
    file(GLOB_RECURSE ENGINE_GL_SOURCES "src/renderer/gl_engine/*.cpp")
    list(APPEND THORVG_SOURCES ${ENGINE_GL_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/gl_engine")
    add_compile_definitions(THORVG_GL_RASTER_SUPPORT)
endif()

# Loader: SVG
if(THORVG_LOADER_SVG)
    file(GLOB_RECURSE LOADER_SVG_SOURCES "src/loaders/svg/*.cpp")
    list(APPEND THORVG_SOURCES ${LOADER_SVG_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/svg")
    add_compile_definitions(THORVG_SVG_LOADER_SUPPORT)
endif()

# Loader: Lottie
if(THORVG_LOADER_LOTTIE)
    file(GLOB_RECURSE LOADER_LOTTIE_SOURCES "src/loaders/lottie/*.cpp")
    list(FILTER LOADER_LOTTIE_SOURCES EXCLUDE REGEX ".*jerryscript.*")
    list(APPEND THORVG_SOURCES ${LOADER_LOTTIE_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/lottie")
    add_compile_definitions(THORVG_LOTTIE_LOADER_SUPPORT)
endif()

# Loader: TVG
if(THORVG_LOADER_TVG)
    file(GLOB_RECURSE LOADER_TVG_SOURCES "src/loaders/tvg/*.cpp")
    list(APPEND THORVG_SOURCES ${LOADER_TVG_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/tvg")
    add_compile_definitions(THORVG_TVG_LOADER_SUPPORT)
endif()

# Loader: RAW
if(THORVG_LOADER_RAW)
    file(GLOB_RECURSE LOADER_RAW_SOURCES "src/loaders/raw/*.cpp")
    list(APPEND THORVG_SOURCES ${LOADER_RAW_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/raw")
    add_compile_definitions(THORVG_RAW_LOADER_SUPPORT)
endif()

# Loader: TTF
if(THORVG_LOADER_TTF)
    file(GLOB_RECURSE LOADER_TTF_SOURCES "src/loaders/ttf/*.cpp")
    list(APPEND THORVG_SOURCES ${LOADER_TTF_SOURCES})
    list(APPEND TVG_PRIVATE_INCLUDES "${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/ttf")
    add_compile_definitions(THORVG_TTF_LOADER_SUPPORT)
endif()

# --- 6. TARGET CREATION ---
add_library(thorvg STATIC ${THORVG_SOURCES})

# --- 7. INCLUDE CONFIGURATION ---
target_include_directories(thorvg PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}/inc"
)

# Private Includes (Core + Modules detected above)
target_include_directories(thorvg PRIVATE
    "${CMAKE_CURRENT_BINARY_DIR}"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/common"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/renderer"
    ${TVG_PRIVATE_INCLUDES}
)

# --- 8. GENERAL DEFINITIONS ---
target_compile_definitions(thorvg PUBLIC TVG_STATIC THORVG_FILE_IO_SUPPORT)

# --- 9. SYSTEM DEPENDENCIES ---
find_package(Threads REQUIRED)
target_link_libraries(thorvg PRIVATE Threads::Threads)
