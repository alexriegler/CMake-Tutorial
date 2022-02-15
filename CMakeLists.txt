# Minimum CMake version required.
cmake_minimum_required(VERSION 3.10)

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

    # Add the MathFunctions directory to the header file include directories
    # variable list.
    list(APPEND EXTRA_INCLUDES "${PROJECT_SOURCE_DIR}/MathFunctions")
endif()

# Add the executable.
add_executable(Tutorial tutorial.cxx)

# Add the libraries to link to the executable (possibly empty).
# target_link_libraries(Tutorial PUBLIC MathFunctions)
target_link_libraries(Tutorial PUBLIC ${EXTRA_LIBS})

# Include files.
# Add the binary tree to the search path for include files so that we will find
# TutorialConfig.h (version header).
target_include_directories(Tutorial PUBLIC
                           "${PROJECT_BINARY_DIR}"
                           "${EXTRA_INCLUDES}"
                           )
