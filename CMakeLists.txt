cmake_minimum_required(VERSION 3.22 FATAL_ERROR)

project(
    VMAware
    DESCRIPTION "VM detection library"
    LANGUAGES CXX
)

# 1. C++ Standard Configuration
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# 2. Define Targets
# Create the static library (.a / .lib)
add_library(vmaware_static STATIC src/cli.cpp) 
# Note: Ensure you have a .cpp for the library logic, or use the header-only approach if applicable.
# If it's header-only, use: add_library(vmaware_static INTERFACE)

# Create the CLI executable (bin)
add_executable(vmaware_cli src/cli.cpp)

# Link the executable to the library
target_link_libraries(vmaware_cli PRIVATE vmaware_static)

# 3. Compiler Options (Modern Target-based approach)
if(MSVC)
    target_compile_options(vmaware_static PRIVATE /Wall /W4 /EHsc /wd5039 /wd4820 /wd4626 /wd5045 /wd4668 /Zc:enumTypes)
    target_compile_options(vmaware_cli PRIVATE /Wall /W4 /EHsc)
else()
    target_compile_options(vmaware_static PRIVATE -Wall -Wextra -Wconversion -Wdouble-promotion -Wno-unused-parameter)
    target_compile_options(vmaware_cli PRIVATE -Wall -Wextra)
endif()

# 4. Build Types & Macros
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "Choose the type of build" FORCE)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug")
    target_compile_definitions(vmaware_static PUBLIC __VMAWARE_DEBUG__)
    if(NOT MSVC)
        target_compile_options(vmaware_static PRIVATE -g -O0 -fsanitize=address)
    endif()
else()
    target_compile_definitions(vmaware_static PUBLIC __VMAWARE_RELEASE__)
    target_compile_options(vmaware_static PRIVATE $<$<NOT:$<CXX_COMPILER_ID:MSVC>>:-O2 -march=native>)
    target_compile_options(vmaware_static PRIVATE $<$<CXX_COMPILER_ID:MSVC>:/O2>)
endif()

# 5. Output Directories (Binaries and Libs)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# 6. Installation Rules
include(GNUInstallDirs) # Provides standard paths like CMAKE_INSTALL_BINDIR

install(TARGETS vmaware_static
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}   # .a / .lib
)

install(TARGETS vmaware_cli
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}   # executable
)

install(FILES "src/vmaware.hpp" DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

# 7. Testing
include(CTest)
enable_testing()
add_test(NAME TestRun COMMAND vmaware_cli --all)