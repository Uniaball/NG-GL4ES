cmake_minimum_required(VERSION 3.10)

project(ng_gl4es)

# ============================================================
#  Optimization options
# ============================================================

if (DEFINED ENV{USE_EXTRA_OPTIMIZATIONS})
    set(USE_EXTRA_OPTIMIZATIONS "$ENV{USE_EXTRA_OPTIMIZATIONS}" CACHE STRING "Enable extra performance optimizations" FORCE)
endif()
if (NOT USE_EXTRA_OPTIMIZATIONS)
    set(USE_EXTRA_OPTIMIZATIONS "OFF")
endif()

if (USE_EXTRA_OPTIMIZATIONS STREQUAL "ON" OR USE_EXTRA_OPTIMIZATIONS STREQUAL "TRUE" OR USE_EXTRA_OPTIMIZATIONS STREQUAL "1")
    set(USE_EXTRA_OPTIMIZATIONS_ENABLED TRUE)
else()
    set(USE_EXTRA_OPTIMIZATIONS_ENABLED FALSE)
endif()

if (DEFINED ENV{LTO_FLAG})
    set(LTO_FLAG "$ENV{LTO_FLAG}" CACHE STRING "LTO flag (e.g., -flto or -flto=thin)" FORCE)
endif()

# ============================================================
#  Base optimization flags (non-Debug only)
# ============================================================

if (NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
    include(CheckIPOSupported)
    check_ipo_supported(RESULT LTOSupported OUTPUT LTOError)
    if (LTOSupported)
        set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
    endif()

    if (CMAKE_C_COMPILER_ID MATCHES "GNU|Clang" AND NOT CMAKE_C_COMPILER_ID MATCHES "AppleClang")
        add_compile_options(-O3 -ffunction-sections -fdata-sections)
        add_link_options(-Wl,--gc-sections)
    elseif (CMAKE_C_COMPILER_ID MATCHES "MSVC")
        add_compile_options(/O2 /GL)
    endif()
endif()

# ============================================================
#  EXTRA OPTIMIZATIONS (GNU|Clang, non-Apple, non-Debug)
# ============================================================

if (CMAKE_C_COMPILER_ID MATCHES "GNU|Clang" AND NOT CMAKE_C_COMPILER_ID MATCHES "AppleClang")
    if (USE_EXTRA_OPTIMIZATIONS_ENABLED AND NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
        message(STATUS "")
        message(STATUS "============================================================")
        message(STATUS "  EXTRA OPTIMIZATIONS ENABLED")
        message(STATUS "============================================================")
        message(STATUS "  Compile flags:")
        message(STATUS "    -fomit-frame-pointer           : omit frame pointer")
        message(STATUS "    -fno-semantic-interposition     : allow more aggressive optimizations")
        message(STATUS "    -fmerge-all-constants          : merge identical constants")
        message(STATUS "    -fno-math-errno                : skip errno for math functions")
        message(STATUS "    -fno-signed-zeros              : ignore sign of zero")
        message(STATUS "    -fno-trapping-math             : skip FP exception trapping")
        message(STATUS "    -fno-unwind-tables             : omit unwind tables")
        message(STATUS "    -fno-asynchronous-unwind-tables : omit async unwind tables")
        message(STATUS "    -fstrict-aliasing              : enable strict aliasing")
        message(STATUS "    -fno-rtti                      : disable RTTI (CXX only)")
        message(STATUS "    -D_NDK_MATH_NO_SOFT=1         : force hardware FP (Android)")
        message(STATUS "    -ffast-math                    : aggressive FP optimizations")
        message(STATUS "    -Wno-nan-infinity-disabled     : suppress INF/NAN warnings from 3rdparty")
        message(STATUS "")
        message(STATUS "  Link flags:")
        message(STATUS "    -Wl,--as-needed                : link only used libraries")
        message(STATUS "    -Wl,--icf=all                  : merge identical code sections")
        message(STATUS "    -Wl,--hash-style=gnu           : GNU hash style")
        message(STATUS "    -Wl,-Bsymbolic                 : prefer local symbols")
        message(STATUS "    -Wl,--exclude-libs,ALL         : hide static lib symbols")
        if (LTO_FLAG)
            message(STATUS "    ${LTO_FLAG}                    : LTO")
        else()
            message(STATUS "    LTO                            : Disabled")
        endif()
        message(STATUS "============================================================")
        message(STATUS "")

        add_compile_options(
            -fomit-frame-pointer
            -fno-semantic-interposition
            -fmerge-all-constants
            -fno-math-errno
            -fno-signed-zeros
            -fno-trapping-math
            -fno-unwind-tables
            -fno-asynchronous-unwind-tables
            -fstrict-aliasing
            $<$<COMPILE_LANGUAGE:CXX>:-fno-rtti>
            -D_NDK_MATH_NO_SOFT=1
            -ffast-math
            -Wno-nan-infinity-disabled
        )
        add_link_options(
            -Wl,--as-needed
            -Wl,--icf=all
            -Wl,--hash-style=gnu
            -Wl,-Bsymbolic
            -Wl,--exclude-libs,ALL
        )

        # LTO
        if (LTO_FLAG)
            message(STATUS "LTO: ${LTO_FLAG}")
            add_compile_options(${LTO_FLAG})
            add_link_options(${LTO_FLAG})
        endif()

    else()
        message(STATUS "")
        message(STATUS "============================================================")
        message(STATUS "  EXTRA OPTIMIZATIONS DISABLED")
        message(STATUS "============================================================")
        message(STATUS "  USE_EXTRA_OPTIMIZATIONS = ${USE_EXTRA_OPTIMIZATIONS}")
        message(STATUS "  CMAKE_BUILD_TYPE        = ${CMAKE_BUILD_TYPE}")
        message(STATUS "============================================================")
        message(STATUS "")
    endif()
endif()

# glslang & SPIRV-Cross

set(ENABLE_RTTI                ON  CACHE BOOL "Enables RTTI (will be took by glslang)" FORCE)
set(ENABLE_HLSL                OFF CACHE BOOL "Disable HLSL input for glslang" FORCE)
set(BUILD_EXTERNAL             OFF CACHE BOOL "Don't build external deps in External/" FORCE)
set(ENABLE_GLSLANG_BINARIES    OFF CACHE BOOL "Disable glslangValidator/spirv-remap" FORCE)
set(ENABLE_OPT                 OFF CACHE BOOL "Disable SPIRV-Tools opt usage in glslang" FORCE)

set(SPIRV_CROSS_ENABLE_C_API   ON  CACHE BOOL "Enable C API" FORCE)
set(SPIRV_CROSS_ENABLE_GLSL    ON  CACHE BOOL "Enable GLSL backend" FORCE)
set(SPIRV_CROSS_ENABLE_REFLECT OFF CACHE BOOL "Disable reflect" FORCE)
set(SPIRV_CROSS_ENABLE_HLSL    OFF CACHE BOOL "Disable HLSL backend" FORCE)
set(SPIRV_CROSS_ENABLE_MSL     OFF CACHE BOOL "Disable MSL backend" FORCE)
set(SPIRV_CROSS_ENABLE_CPP     OFF CACHE BOOL "Disable C++ API target" FORCE)
set(SPIRV_CROSS_CLI            OFF CACHE BOOL "Disable CLI binary" FORCE)
set(SPIRV_CROSS_STATIC         ON  CACHE BOOL "Prefer static libs" FORCE)
set(SPIRV_CROSS_ENABLE_UTIL    OFF  CACHE BOOL "Disable util" FORCE)

add_subdirectory(3rdparty/glslang)
add_subdirectory(3rdparty/SPIRV-Cross)

enable_language(CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Platform-specific C/CXX flags
if(MACOS)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -fvisibility=default -funwind-tables -g -D_THREAD_SAFE -fPIC -stdlib=libc++")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w -g -std=gnu11 -funwind-tables -fPIC -fvisibility=hidden -DNO_GBM -DDEFAULT_ES=3")
else()
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w -fvisibility=hidden -funwind-tables -g -D_THREAD_SAFE -fPIC")
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -w -g -std=gnu11 -funwind-tables -fPIC -fvisibility=hidden -DNO_GBM -DDEFAULT_ES=3")
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -static-libstdc++")
endif()

if(ANDROID)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -DNOX11 -DDEBUG")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DDEBUG")
endif()

set(C_SRC_FILES
    src/gl/string_utils.c
    src/gl/cJSON.c
    src/gl/config.c
    src/gl/arbconverter.c
    src/gl/arbgenerator.c
    src/gl/arbhelper.c
    src/gl/arbparser.c
    src/gl/array.c
    src/gl/blend.c
    src/gl/blit.c
    src/gl/buffers.c
    src/gl/build_info.c
    src/gl/debug.c
    src/gl/decompress.c
    src/gl/depth.c
    src/gl/directstate.c
    src/gl/drawing.c
    src/gl/enable.c
    src/gl/envvars.c
    src/gl/eval.c
    src/gl/face.c
    src/gl/fog.c
    src/gl/fpe.c
    src/gl/fpe_cache.c
    src/gl/fpe_shader.c
    src/gl/framebuffers.c
    src/gl/gl_lookup.c
    src/gl/gl4es.c
    src/gl/glstate.c
    src/gl/hint.c
    src/gl/init.c
    src/gl/light.c
    src/gl/line.c
    src/gl/list.c
    src/gl/listdraw.c
    src/gl/listrl.c
    src/gl/loader.c
    src/gl/logs.c
    src/gl/matrix.c
    src/gl/matvec.c
    src/gl/oldprogram.c
    src/gl/pixel.c
    src/gl/planes.c
    src/gl/pointsprite.c
    src/gl/preproc.c
    src/gl/program.c
    src/gl/queries.c
    src/gl/raster.c
    src/gl/render.c
    src/gl/shader.c
    src/gl/shaderconv.c
    src/gl/shader_hacks.c
    src/gl/stack.c
    src/gl/stencil.c
    src/gl/stubs.c
    src/gl/texenv.c
    src/gl/texgen.c
    src/gl/texture.c
    src/gl/texture_compressed.c
    src/gl/texture_params.c
    src/gl/texture_read.c
    src/gl/texture_3d.c
    src/gl/uniform.c
    src/gl/vertexattrib.c
    src/gl/wrap/gl4eswraps.c
    src/gl/wrap/gles.c
    src/gl/wrap/glstub.c
    src/gl/wrap/glesnative.cpp
    src/gl/math/matheval.c
    src/glx/hardext.c
    src/glx/lookup.c
    src/glx/gbm.c
    src/glx/streaming.c
    src/gl/vgpu/shaderconv.c
    src/gl/samplers.c
)

if(NOT ANDROID)
    list(APPEND C_SRC_FILES src/glx/glx.c)
endif()

set(CPP_SRC_FILES
    src/gl/glsl/glsl_for_es.cpp
    src/gl/getter.cpp
)

set(SRC_FILES ${CPP_SRC_FILES} ${C_SRC_FILES})

add_library(ng_gl4es SHARED ${SRC_FILES})

target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE
    glslang::glslang
    spirv-cross-c
)

if(MACOS)
    # iOS
    target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC
        "-F${CMAKE_SOURCE_DIR}/external"
        "-framework libEGL"
        "-framework libGLESv2"
        "-framework Metal"
        "-framework Foundation"
        "-framework QuartzCore"
    )
else()
if(ANDROID)
    # Android
    target_link_libraries(${CMAKE_PROJECT_NAME} PUBLIC
        android
        log
    )
else()
    # Nothing to link
endif()
endif()

target_include_directories(ng_gl4es PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}/include
)

set_target_properties(${CMAKE_PROJECT_NAME} PROPERTIES
    LINK_FLAGS "-Wl,--strip-all"
)
