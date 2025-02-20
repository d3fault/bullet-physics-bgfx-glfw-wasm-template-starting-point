cmake_minimum_required(VERSION 3.10)

# If the EMSDK environment variable is set, configure the toolchain file. this has to be done before the project() line below
if(DEFINED ENV{EMSDK})
    set(CMAKE_TOOLCHAIN_FILE "$ENV{EMSDK}/upstream/emscripten/cmake/Modules/Platform/Emscripten.cmake" CACHE STRING "Emscripten toolchain file")
    message(STATUS "WebAssembly build enabled")
    message(STATUS "CMAKE_TOOLCHAIN_FILE='${CMAKE_TOOLCHAIN_FILE}'")
else()
    message(STATUS "WebAssembly build NOT enabled. EMSDK environment variable is not set. Falling back to native build. If you want to compile for WebAssembly, first `source emsdk_env.sh` in the emsdk directory and then do `mkdir buildwasm && cd buildwasm && emcmake cmake .. && emmake make`")
endif()

project(bullet-physics-bgfx-glfw-wasm-template-starting-point)

set(CMAKE_CXX_STANDARD 14)
set(CMAKE_COMPILE_WARNING_AS_ERROR ON)

set(BGFX_DIR ${CMAKE_SOURCE_DIR}/bgfx)

if(NOT DEFINED ENV{EMSDK})
    find_package(glfw3 REQUIRED)
    find_package(OpenGL REQUIRED)
    find_package(Bullet COMPONENTS BulletCollision BulletDynamics REQUIRED)
    include_directories(${BULLET_INCLUDE_DIR})
endif()

include_directories(${BGFX_DIR}/include)
include_directories(${CMAKE_SOURCE_DIR}/bx/include)
include_directories(${CMAKE_SOURCE_DIR}/bimg/include)

add_definitions(-DBX_CONFIG_DEBUG)

add_executable(bullet-physics-bgfx-glfw-wasm-template-starting-point main.cpp)

file(INSTALL DESTINATION ${CMAKE_BINARY_DIR}
    TYPE FILE
    FILES ${BGFX_DIR}/examples/runtime/shaders/glsl/vs_cubes.bin
    ${BGFX_DIR}/examples/runtime/shaders/glsl/fs_cubes.bin)

if(DEFINED ENV{EMSDK})
    set(CMAKE_CXX_FLAGS "-s MIN_WEBGL_VERSION=2 \
        -s MAX_WEBGL_VERSION=2 \
        -s EXCEPTION_DEBUG \
        -fexceptions \
        --preload-file vs_cubes.bin \
        --preload-file fs_cubes.bin \
        --bind \
        --use-preload-plugins \
        -Wall \
        -Wextra \
        -Werror=return-type \
        -s ASSERTIONS=1 \
        -w \
        -g4 \
        -s DISABLE_EXCEPTION_CATCHING=0 \
        -Os \
        -s USE_GLFW=3 \
        -s USE_BULLET=1 \
        --no-heap-copy \
        -s GL_ENABLE_GET_PROC_ADDRESS \
        -static \
        -s ALLOW_MEMORY_GROWTH=1 \
        -s EXIT_RUNTIME=1")

    target_link_libraries(bullet-physics-bgfx-glfw-wasm-template-starting-point PRIVATE
        ${BGFX_DIR}/.build/wasm/bin/bgfxDebug.bc
        ${BGFX_DIR}/.build/wasm/bin/bxDebug.bc
        ${BGFX_DIR}/.build/wasm/bin/bimgDebug.bc)

    file(INSTALL DESTINATION ${CMAKE_BINARY_DIR}
        TYPE FILE
        FILES ${CMAKE_SOURCE_DIR}/index.html)
else() # Native Linux/X11
    target_link_libraries(bullet-physics-bgfx-glfw-wasm-template-starting-point PRIVATE
        glfw
        OpenGL::GL
        ${BGFX_DIR}/.build/linux64_gcc/bin/libbgfx-shared-libDebug.so
	${BULLET_LIBRARIES})
endif()
