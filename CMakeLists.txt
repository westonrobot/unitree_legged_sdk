cmake_minimum_required(VERSION 3.5)
project(unitree_legged_sdk VERSION 3.8.0)

## Project Options
option(BUILD_EXAMPLES "Build examples" ON)

## Set compiler to use c++ 14 features
set(CMAKE_CXX_STANDARD 14)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

## Chosse build type
set(default_build_type "Release")
if (NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    message(STATUS "Setting build type to '${default_build_type}' as none was specified.")
    set(CMAKE_BUILD_TYPE "${default_build_type}" CACHE
            STRING "Choose the type of build." FORCE)
    # Set the possible values of build type for cmake-gui
    set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS
            "Debug" "Release" "MinSizeRel" "RelWithDebInfo")
endif ()

## Use GNUInstallDirs to install libraries into correct locations on all platforms.
include(GNUInstallDirs)

## Put all binary files into /bin and libraries into /lib
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_LIBDIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/${CMAKE_INSTALL_BINDIR})

## Check system architecture
message(STATUS "Current system architecture: ${CMAKE_SYSTEM_PROCESSOR}")

## Import Unitree SDK2 library
if ("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "x86_64.*")
    set(UNITREE_LIB_PATH ${CMAKE_CURRENT_LIST_DIR}/lib/cpp/amd64)
elseif ("${CMAKE_SYSTEM_PROCESSOR}" MATCHES "aarch64.*")
    set(UNITREE_LIB_PATH ${CMAKE_CURRENT_LIST_DIR}/lib/cpp/arm64)
endif ()
find_library(UNITREE_SDK_LIB unitree_legged_sdk PATHS ${UNITREE_LIB_PATH})

if (NOT UNITREE_SDK_LIB)
    message(FATAL_ERROR "Unitree SDK2 library not found")
else ()
    message(STATUS "Unitree SDK2 library found at: ${UNITREE_SDK_LIB}")
endif ()

find_package(Threads REQUIRED)
find_package(Boost REQUIRED)

#find_package(PkgConfig REQUIRED)
#pkg_check_modules(LCM REQUIRED IMPORTED_TARGET lcm)

add_library(unitree_sdk1 SHARED IMPORTED GLOBAL)
set_target_properties(unitree_sdk1 PROPERTIES
        IMPORTED_LOCATION ${UNITREE_SDK_LIB})
target_link_libraries(unitree_sdk1 INTERFACE
#        PkgConfig::LCM
        Threads::Threads
        ${Boost_LIBRARIES})
target_include_directories(unitree_sdk1 INTERFACE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>)

if(BUILD_EXAMPLES)
    add_subdirectory(example)
endif()
