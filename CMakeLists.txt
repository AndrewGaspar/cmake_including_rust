#--------------------------------------------------------
# The CMakeLists.txt for:
# Author(s):                              cmoran
#--------------------------------------------------------

cmake_minimum_required (VERSION 3.10)
project (say_hello)

# Boiler plate to import cmake-cargo CMake modules
list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/deps/cmake-cargo/cmake")

# Bootstraps building cmake-cargo-gen tool - in the future, cmake-cargo will
# support installation rather than having to build the helper-tool every time
# you configure
set(CARGO_DEV_MODE ON CACHE INTERNAL "")

# includes the Cargo CMake module
include(Cargo)

# Imports the rust_hello crate and all its targets into CMake.
# This will result in two build targets:
# - cargo_rust_hello
# - cargo-clean_rust_hello
# And three library targets:
# - rust_hello-static
# - rust_hello-shared
# - rust_hello (if BUILD_SHARED_LIBS, points to rust_hello-shared, eles rust_hello-static)
#
# cmake-cargo builds debug for CMAKE_BUILD_TYPE=Debug, and release for
# Release, RelWithDebInfo (with -g), and MinSizeRel (+ other flags)
add_crate(rust_hello/Cargo.toml)

# Searches for cbindgen
find_program(CBINDGEN cbindgen)

# Generates rust_hello.h
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/binding/rust_hello.h
    COMMAND
        ${CBINDGEN} ${CMAKE_CURRENT_SOURCE_DIR}/rust_hello/
        -o ${CMAKE_CURRENT_BINARY_DIR}/binding/rust_hello.h
    DEPENDS rust_hello/src/lib.rs)

# Adds target for generating rust_hello.h
add_custom_target(rust_hello-include DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/binding/rust_hello.h)

# Defines the say_hello executable
add_executable(say_hello src/main.cpp)
target_compile_features(say_hello PRIVATE cxx_std_11)

# Make sure to add dependency on rust_hello.h generation
add_dependencies(say_hello rust_hello-include)
target_include_directories(say_hello PRIVATE ${CMAKE_CURRENT_BINARY_DIR}/binding)

# This links static rust_hello
target_link_libraries(say_hello rust_hello-static)

# This links shared rust_hello
# target_link_libraries(say_hello rust_hello-shared)
