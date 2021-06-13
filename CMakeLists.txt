cmake_minimum_required(VERSION 3.13)
project(SOIL2)

# Check that the user knows we are only supporting Linux.
if (NOT (${CMAKE_SYSTEM_NAME} STREQUAL "Linux"))
  message(FATAL_ERROR "This CMakeLists.txt only supports Linux.")
endif()

set(default_build_type "debug")
 
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
  set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
      STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
    "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif()

message("BUILD_TYPE: ${CMAKE_BUILD_TYPE}")

# CMake matches against default DEBUG symbol doesn't work as well as I want.
string(TOLOWER "${CMAKE_BUILD_TYPE}" build_type_lower)
# Convert all release types just to "release".
if (NOT (${build_type_lower} MATCHES "debug"))
    set(build_type_lower "release")
endif()

set(BUILD_TYPE_SUFFIX ${build_type_lower})
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -MMD")
set(CMAKE_CXX_FLAGS_DEBUG "-Wall")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)


if (${build_type_lower} MATCHES "debug")
  add_definitions(-DDEBUG)
  set(SOIL_LIB "soil2-${BUILD_TYPE_SUFFIX}")
else()
  add_definitions(-DNDEBUG)
  set(SOIL_LIB "soil2")
endif()


# === SOIL2 Libs ===

# Normally wouldn't do a glob, but I am not worried about keeping the 
# project up to date. If I start to maintain this fork I should replace
# this.
file(GLOB SRCS "src/SOIL2/*.c")

add_library(${SOIL_LIB} SHARED ${SRCS})
add_library(${SOIL_LIB}static STATIC ${SRCS})
set_target_properties(${SOIL_LIB}static PROPERTIES OUTPUT_NAME ${SOIL_LIB})


target_link_libraries(${SOIL_LIB} GL)

# === SOIL2 Test ===

set(SOIL_TEST "soil2-test-${BUILD_TYPE_SUFFIX}")

file(GLOB TEST_SRCS "src/test/*.cpp")
file(GLOB COMMON_SRCS "src/common/*.cpp")

add_executable(${SOIL_TEST} ${TEST_SRCS} ${COMMON_SRCS})
target_link_libraries(${SOIL_TEST} GL SDL2 ${SOIL_LIB}static)

# === SOIL2 Perf Test === 

set(SOIL_PERF_TEST "soil2-perf-test-${BUILD_TYPE_SUFFIX}")

file(GLOB PERF_SRCS "src/perf_test/*.cpp")

add_executable(${SOIL_PERF_TEST} ${PERF_SRCS} ${COMMON_SRCS})
target_link_libraries(${SOIL_PERF_TEST} GL SDL2 ${SOIL_LIB}static)

# === Exporting some useful variables. ===

set(SOIL2_INCLUDE_DIRECTORIES "${CMAKE_CURRENT_LIST_DIR}/src/SOIL2" PARENT_SCOPE)
set(SOIL2_LIBRARY_DIRECTORIES "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}" PARENT_SCOPE)
set(SOIL2_LIBRARIES ${SOIL_LIB} PARENT_SCOPE)
