#
# This file is part of the µOS++ distribution.
#   (https://github.com/micro-os-plus)
# Copyright (c) 2021 Liviu Ionescu
#
# This Source Code Form is subject to the terms of the MIT License.
# If a copy of the license was not distributed with this file, it can
# be obtained from https://opensource.org/licenses/MIT/.
#
# -----------------------------------------------------------------------------

# https://cmake.org/cmake/help/v3.18/
cmake_minimum_required(VERSION 3.18)
message(STATUS "CMake version: ${CMAKE_VERSION}")

# -----------------------------------------------------------------------------

if(CMAKE_BINARY_DIR STREQUAL CMAKE_SOURCE_DIR)
  message(FATAL_ERROR "In-source builds are not supported. Please use separate folder for build.")
endif()

# -----------------------------------------------------------------------------

enable_language(C)
enable_language(CXX)
enable_language(ASM)

# Specify the C/C++ standards.
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

message(STATUS "Compiler: ${CMAKE_C_COMPILER_ID} ${CMAKE_C_COMPILER_VERSION}")

# Generate the compile_commands.json file to feed the indexer.
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Enable this to see the dependency graph.
# set_property(GLOBAL PROPERTY GLOBAL_DEPENDS_DEBUG_MODE 1)

# -----------------------------------------------------------------------------

get_filename_component(xpack_tests_folder ${CMAKE_CURRENT_LIST_DIR} DIRECTORY)
get_filename_component(xpack_project_folder ${xpack_tests_folder} DIRECTORY)

include("${xpack_project_folder}/xpacks/micro-os-plus-build-helper/cmake/xpack-helper.cmake")

# Add the current project.
list(APPEND CMAKE_PREFIX_PATH "${xpack_project_folder}/meta")
# Add folders that have a *-config.cmake file.
xpack_append_cmake_prefix_path("${xpack_project_folder}/xpacks")

# -----------------------------------------------------------------------------

# Informative. Leave the result in PACKAGE_JSON_VERSION[*]
xpack_parse_package_json_semver("${xpack_project_folder}/package.json")

project(
  tests
  DESCRIPTION "@micro-os-plus/diag-trace tests"
  VERSION "${PACKAGE_JSON_VERSION}"
)

# -----------------------------------------------------------------------------

message(STATUS "Platform name: ${PLATFORM_NAME}")
# CMAKE_BUILD_TYPE: Debug, Release, RelWithDebInfo, MinSizeRel
message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")
message(STATUS "Project path: ${xpack_project_folder}")
message(STATUS "Prefix path: ${CMAKE_PREFIX_PATH}")

# -----------------------------------------------------------------------------


# Common to all platforms.
add_compile_definitions(
  # NDEBUG provided by the toolchain definitions on release.
  $<$<STREQUAL:"${CMAKE_BUILD_TYPE}","Debug">:DEBUG>

  # $<$<STREQUAL:"${CMAKE_BUILD_TYPE}","Debug">:TRACE>
  # Define it always, even on Release.
  TRACE
)

# -----------------------------------------------------------------------------

if(PLATFORM_NAME STREQUAL "native")
  include("${xpack_tests_folder}/platform-native/meta/config.cmake")
else()
  message(FATAL_ERROR, "Unsupported PLATFORM_NAME")
endif()

# -----------------------------------------------------------------------------

find_package(micro-os-plus-diag-trace REQUIRED)
find_package(micro-os-plus-micro-test-plus REQUIRED)

# -----------------------------------------------------------------------------

function(add_test_executable name)

  add_executable(${name})

  set_target_properties(
    ${name}

    PROPERTIES
      OUTPUT_NAME "${name}"
  )

  message(STATUS "+ src/${name}.cpp")

  # Application sources.
  target_sources(
    ${name}

    PRIVATE
      ${xpack_tests_folder}/src/${name}.cpp
  )

  # Include folders.
  target_include_directories(
    ${name}

    PRIVATE
      ${xpack_tests_folder}/include
  )

  target_compile_definitions(
    ${name}

    PRIVATE
      # ...
  )

  target_compile_options(
    ${name}

    PRIVATE
      # ...
  )

  target_link_libraries(
    ${name}
    
      micro-os-plus::micro-test-plus
      micro-os-plus::diag-trace
  )

  message(STATUS "=> ${name}")

endfunction()

# -----------------------------------------------------------------------------

add_test_executable(unit-test)

add_test_executable(sample-test)

# -----------------------------------------------------------------------------

enable_testing()

if(PLATFORM_NAME STREQUAL "native")

  add_test(
    NAME "unit-test"
    COMMAND unit-test
  )

  add_test(
    NAME "sample-test"
    COMMAND sample-test 1 2 3
  )

endif()

# -----------------------------------------------------------------------------

# Display an empty line, to better see the configuration output.
message(STATUS "")

# -----------------------------------------------------------------------------
