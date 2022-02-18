# declspec(dllexport) convenience for Windows MSVC.
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# Add the library that runs.
add_library(MathFunctions MathFunctions.cxx)

# State that anybody linking to us needs to include the current source dir
# to find MathFunctions.h, while we don't.
target_include_directories(MathFunctions
                           INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})

# Should we use our own math functions?
option(USE_MYMATH "Use tutorial provided math implementation" ON)
if(USE_MYMATH)
    # Define "USE_MYMATH" macro.
    target_compile_definitions(MathFunctions PRIVATE "USE_MYMATH")

    # Add the executable that generates the table.
    add_executable(MakeTable MakeTable.cxx)

    # Add the command to generate the source code.
    add_custom_command(
        OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        COMMAND MakeTable ${CMAKE_CURRENT_BINARY_DIR}/Table.h
        DEPENDS MakeTable
    )

    # Add library that just does sqrt.
    add_library(SqrtLibrary STATIC
                mysqrt.cxx
                ${CMAKE_CURRENT_BINARY_DIR}/Table.h)

    # State that SqrtLibrary need PIC when the default is shared libraries.
    set_target_properties(SqrtLibrary PROPERTIES
                          POSITION_INDEPENDENT_CODE ${BUILD_SHARED_LIBS})

    # State that we depend on our binary dir to find Table.h.
    target_include_directories(SqrtLibrary PRIVATE
                               ${CMAKE_CURRENT_BINARY_DIR})

    target_link_libraries(MathFunctions PRIVATE SqrtLibrary)
endif()

# Step 5: Adding System Introspection
# # Does this system provide the log and exp functions?---------------------------
# # Include CheckSymbolExists CMake module (for check_symbol_exists()).
# include(CheckSymbolExists)

# # Check for log and exp symbols.
# check_symbol_exists(log "math.h" HAVE_LOG)
# check_symbol_exists(exp "math.h" HAVE_EXP)

# if(NOT (HAVE_LOG AND HAVE_EXP))
#     # Reset check_symbol_exists() results.
#     unset(HAVE_LOG CACHE)
#     unset(HAVE_EXP CACHE)

#     # Add the "m" library to the libraries to use when looking for symbols with
#     # the check_symbol_exists macro.
#     # I.e., add "m" library to the link command.
#     # Note: "m" is the linker name of the math library.
#     set(CMAKE_REQUIRED_LIBRARIES "m")

#     # Recheck log and exp symbols (has access to "m" library now).
#     check_symbol_exists(log "math.h" HAVE_LOG)
#     check_symbol_exists(exp "math.h" HAVE_EXP)

#     # If found, set as required library to link with.
#     # Otherwise, a linker error will occur later?
#     if(HAVE_LOG AND HAVE_EXP)
#         target_link_libraries(MathFunctions PRIVATE m)
#     endif()
# endif()
# # Does this system provide the log and exp functions?---------------------------

# # Specify HAVE_LOG and HAVE_EXP as PRIVATE compile definitions.
# if(HAVE_LOG AND HAVE_EXP)
#     # Adds macro definitions for "HAVE_LOG" "HAVE_EXP".
#     target_compile_definitions(MathFunctions
#                                PRIVATE "HAVE_LOG" "HAVE_EXP")
# endif()

# Install rules.
set(installable_libs MathFunctions)
if(TARGET SqrtLibrary)
    list(APPEND installable_libs SqrtLibrary)
endif()
install(TARGETS ${installable_libs} DESTINATION lib)
install(FILES MathFunctions.h DESTINATION include)
