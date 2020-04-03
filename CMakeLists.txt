cmake_minimum_required(VERSION 3.16)

project(sdlvkcube LANGUAGES C CXX)

find_package(SDL2 CONFIG REQUIRED)
find_package(Vulkan REQUIRED)
find_program(GLSLANG_VALIDATOR
    NAMES glslangValidator
    HINTS $ENV{HOME}/.local/bin "/usr/local/bin" "/usr/bin"
)

add_custom_command(COMMENT "Compiling cube vertex shader"
                   OUTPUT cube.vert.inc
                   COMMAND ${GLSLANG_VALIDATOR} -V -x -o ${CMAKE_CURRENT_BINARY_DIR}/cube.vert.inc
                           ${PROJECT_SOURCE_DIR}/cube.vert
                   MAIN_DEPENDENCY ${PROJECT_SOURCE_DIR}/cube.vert
                   DEPENDS ${PROJECT_SOURCE_DIR}/cube.vert ${GLSLANG_VALIDATOR})
add_custom_command(COMMENT "Compiling cube fragment shader"
                   OUTPUT cube.frag.inc
                   COMMAND ${GLSLANG_VALIDATOR} -V -x -o ${CMAKE_CURRENT_BINARY_DIR}/cube.frag.inc
                           ${PROJECT_SOURCE_DIR}/cube.frag
                   MAIN_DEPENDENCY ${PROJECT_SOURCE_DIR}/cube.frag
                   DEPENDS ${PROJECT_SOURCE_DIR}/cube.frag ${GLSLANG_VALIDATOR})
include_directories(${CMAKE_CURRENT_BINARY_DIR} ${CMAKE_CURRENT_SOURCE_DIR})

if(UNIX AND NOT APPLE) # ex: Linux and BSD
    add_definitions(-DVK_USE_PLATFORM_XLIB_KHR)
    add_definitions(-DVK_USE_PLATFORM_XCB_KHR)
    add_definitions(-DVK_USE_PLATFORM_WAYLAND_KHR)
    add_definitions(-DVK_USE_PLATFORM_DISPLAY_KHR)
else()
    message(FATAL_ERROR "Unsupported Platform!")
endif()

add_executable(vkcube
                cube.c
                cube.vert
                cube.frag
                cube.vert.inc
                cube.frag.inc)

target_link_libraries(vkcube Vulkan::Vulkan SDL2::SDL2)

include(CheckLibraryExists)
CHECK_LIBRARY_EXISTS("rt" clock_gettime "" NEED_RT)
if (NEED_RT)
    target_link_libraries(vkcube rt)
endif()
