# Minimum CMake version required.
cmake_minimum_required(VERSION 3.10)

# Set the project name and version.
project(Tutorial VERSION 1.0)

# Specify the C++ standard (Must be above the call to add_executable).
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Configure a header file to pass the version number to the source code.
configure_file(TutorialConfig.h.in TutorialConfig.h)

# Add the executable.
add_executable(Tutorial tutorial.cxx)

# Include files.
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")
