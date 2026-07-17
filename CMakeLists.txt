cmake_minimum_required(VERSION 3.10)

# 1. Project name and language
project(aruco2 LANGUAGES CXX)

# 2. Set the C++ standard
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# ---------------------------------------------------------------------------
# 3. Handle Build Type Portability (Release mode)
# ---------------------------------------------------------------------------

# For Linux/Unix (Single-config): Default to Release if nothing is specified
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "No build type selected, defaulting to Release")
    set(CMAKE_BUILD_TYPE "Release" CACHE STRING "Build type" FORCE)
endif()

# ---------------------------------------------------------------------------
# 4. Find Dependencies
# ---------------------------------------------------------------------------
find_package(OpenCV REQUIRED)
find_package(Python3 REQUIRED COMPONENTS Interpreter)

#include opencv dir
include_directories(${OpenCV_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
include_directories(src)
include_directories(opencv2/objdetect)

# Generate C++ wrappers for OpenCL kernels
add_custom_command(
    OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.hpp"
           "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.cpp"
    COMMAND Python3::Interpreter "${CMAKE_CURRENT_SOURCE_DIR}/utils/cl2cpp.py"
            "${CMAKE_CURRENT_SOURCE_DIR}/opencl/arucodetect.cl"
            "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.hpp"
            "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.cpp"
    DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/opencl/arucodetect.cl"
            "${CMAKE_CURRENT_SOURCE_DIR}/utils/cl2cpp.py"
    COMMENT "Generating C++ wrappers from OpenCL kernels"
    VERBATIM
)

# Include the binary directory so the compiler can find the generated header
include_directories(${CMAKE_CURRENT_BINARY_DIR})

SET(SRCARUCO2
    opencv2/objdetect/aruco2.hpp
    src/aruco2.cpp
    src/aruco_dictionary.hpp
    src/aruco_dictionary.cpp
    src/aruco2_fractal.cpp
    "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.hpp"
    "${CMAKE_CURRENT_BINARY_DIR}/opencl_kernels_aruco2.cpp"
)

# ---------------------------------------------------------------------------
# 5. Compiler warnings — pedantic, treat warnings as errors
# ---------------------------------------------------------------------------
# if(CMAKE_CXX_COMPILER_ID MATCHES "GNU|Clang")
#     add_compile_options(-Wall -Wextra -Wpedantic -Werror)
# elseif(MSVC)
#     add_compile_options(/W4 /WX)
# endif()

# Create static library for Aruco2 core
add_library(aruco2 STATIC ${SRCARUCO2})
target_link_libraries(aruco2 PUBLIC ${OpenCV_LIBS})

# 5. Add Executable
OPTION(BUILD_DEV OFF)
IF(BUILD_DEV)
add_executable(devtest1 dev/devtest1.cpp)
target_link_libraries(devtest1 PRIVATE aruco2)
add_executable(devtest2 dev/devtest2.cpp)
target_link_libraries(devtest2 PRIVATE aruco2)
ENDIF()

add_executable(aruco2_camera_calibration utils/aruco2_camera_calibration.cpp)
target_link_libraries(aruco2_camera_calibration PRIVATE aruco2)

add_executable(aruco2_print utils/aruco2_print.cpp)
target_link_libraries(aruco2_print PRIVATE aruco2)

add_executable(aruco2_detect_markers utils/aruco2_detect_markers.cpp)
target_link_libraries(aruco2_detect_markers PRIVATE aruco2)

add_executable(aruco2_detect_diamonds utils/aruco2_detect_diamonds.cpp)
target_link_libraries(aruco2_detect_diamonds PRIVATE aruco2)

add_executable(aruco2_detect_fractal utils/aruco2_detect_fractal.cpp)
target_link_libraries(aruco2_detect_fractal PRIVATE aruco2)

add_executable(aruco2_testdictspeedup aruco2_testdictspeedup.cpp)
target_link_libraries(aruco2_testdictspeedup PRIVATE aruco2)

# ---------------------------------------------------------------------------
# 8. Status Printout
# ---------------------------------------------------------------------------
message(STATUS "---------------------------------------------------")
message(STATUS "OpenCV version: ${OpenCV_VERSION}")
message(STATUS "Build type:     ${CMAKE_BUILD_TYPE}")
message(STATUS "Compiler ID:    ${CMAKE_CXX_COMPILER_ID}")
message(STATUS "---------------------------------------------------")
