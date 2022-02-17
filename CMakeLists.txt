# CMake commands to run (assuming ran from bin directory):
# Configure cmake and create build files:
# cmake ..
#
# Add option:
# cmake .. -DUSE_MYMATH=OFF
#
# Build (compile and link) application:
# cmake --build .
#
# Install local
# TODO
#
# Create installer:
# Windows: Need NSIS for installer (run "winget install NSIS.NSIS").
# Build the project in the usual manner and then run the cpack executable.
# To build a binary distribution, from the binary directory run:
# cpack
#
# To specify the generator, use the -G option. For multi-config builds, use -C
# to specify the configuration. For example:
# cpack -G ZIP -C Debug
#
# To create an archive of the full source tree you would type:
# cpack --config CPackSourceConfig.cmake
#

# Minimum CMake version required.
cmake_minimum_required(VERSION 3.15)

# Set the project name and version.
project(Tutorial VERSION 1.0)

# Specify the C++ standard (Must be above the call to add_executable).
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Should we use our own math functions?
# Must be above configure_file so that the option is placed into the header.
option(USE_MYMATH "Use tutorial provided math implementation" ON)

# Configure a header file to pass the version number to the source code (and
# possibly other CMake settings).
configure_file(TutorialConfig.h.in TutorialConfig.h)

# Add the MathFunctions library (if ON).
if(USE_MYMATH)
    # Add the MathFunctions library for searching.
    add_subdirectory(MathFunctions)

    # Add the MathFunctions library to the variable list of libraries to link
    # the executable to.
    list(APPEND EXTRA_LIBS MathFunctions)
endif()

# Add the executable.
add_executable(Tutorial tutorial.cxx)

# Add the libraries to link to the executable (possibly empty).
# target_link_libraries(Tutorial PUBLIC MathFunctions)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# Include files.
# Add the binary tree to the search path for include files so that we will find
# TutorialConfig.h (version header).
target_include_directories(Tutorial PUBLIC "${PROJECT_BINARY_DIR}")

# Install rules.
install(TARGETS Tutorial DESTINATION bin)
install(FILES "${PROJECT_BINARY_DIR}/TutorialConfig.h"
        DESTINATION include
        )

# Testing
enable_testing()

# Does the application run?
add_test(NAME Runs COMMAND Tutorial 25)

# Does the usage message work?
add_test(NAME Usage COMMAND Tutorial)
set_tests_properties(Usage PROPERTIES PASS_REGULAR_EXPRESSION "Usage:.*number")

# Define a function to simplify adding tests.
function(do_test target arg result)
    add_test(NAME Comp${arg} COMMAND ${target} ${arg})
    set_tests_properties(
        Comp${arg}
        PROPERTIES PASS_REGULAR_EXPRESSION ${result}
    )
endfunction()

# Do a bunch of result based tests.
do_test(Tutorial 4 "4 is 2")
do_test(Tutorial 9 "9 is 3")
do_test(Tutorial 5 "5 is 2.236")
do_test(Tutorial 7 "7 is 2.645")
do_test(Tutorial 25 "25 is 5")
do_test(Tutorial -25 "-25 is (-nan|nan|0)")
do_test(Tutorial 0.0001 "0.0001 is 0.01")

# Step 7: Packaging an Installer
# Setup installer.
include(InstallRequiredSystemLibraries)
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_CURRENT_SOURCE_DIR}/LICENSE")
set(CPACK_PACKAGE_VERSION_MAJOR "${Tutorial_VERSION_MAJOR}")
set(CPACK_PACKAGE_VERSION_MINOR "${Tutorial_VERSION_MINOR}")
set(CPACK_SOURCE_GENERATOR "TGZ")
include(CPack)
