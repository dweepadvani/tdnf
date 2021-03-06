#
# Copyright (C) 2019-2020 VMware, Inc. All Rights Reserved.
#
# Licensed under the GNU General Public License v2 (the "License");
# you may not use this file except in compliance with the License. The terms
# of the License are located in the COPYING file of this distribution.
#
#   Author: Siddharth Chandrasekaran <csiddharth@vmware.com>

CMAKE_MINIMUM_REQUIRED(VERSION 3.0 FATAL_ERROR)

project(tdnf VERSION 2.1.0 LANGUAGES C)
set(PROJECT_YEAR 2019)

# Ensure correct directory paths are used for installation
include(GNUInstallDirs)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(AddCCompilerFlag)
include(CreatePackages)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Ensure that we don't accidentally install into /usr/etc
set(CMAKE_INSTALL_FULL_SYSCONDIR "/etc")

##
## C Flags
##

add_c_compiler_flag(-Wall)
add_c_compiler_flag(-Wextra)
add_c_compiler_flag(-std=gnu99)
add_c_compiler_flag(-fPIC)

add_c_compiler_flag(-O3 RELEASE)
add_c_compiler_flag(-s RELEASE)
add_c_compiler_flag(-g DEBUG)

include_directories(${CMAKE_CURRENT_SOURCE_DIR}/include)

### External dependency: rpm-devel
find_package(PkgConfig REQUIRED)
pkg_check_modules(RPM REQUIRED rpm)
include_directories(${RPM_INCLUDE_DIRS})

### External dependency: libsolv
find_package(LibSolv REQUIRED ext)

### External dependency: libcurl
find_package(CURL REQUIRED)

##
## check target
##
configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/pytests/config.json.in"
    "${CMAKE_CURRENT_SOURCE_DIR}/pytests/config.json" @ONLY
)

add_custom_target(check
    COMMAND pytest --build-dir=${CMAKE_CURRENT_BINARY_DIR}
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/pytests
    COMMENT "Running tests.."
)

install(DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/include/" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/tdnf")

set(LIB_TDNF "tdnf")
set(LIB_TDNF_SOLV "tdnfsolv")
set(LIB_TDNF_COMMON "common")
set(LIB_TDNF_CLI tdnfcli)

add_subdirectory("${PROJECT_SOURCE_DIR}/common")
add_subdirectory("${PROJECT_SOURCE_DIR}/client")
add_subdirectory("${PROJECT_SOURCE_DIR}/plugins")
add_subdirectory("${PROJECT_SOURCE_DIR}/python")
add_subdirectory("${PROJECT_SOURCE_DIR}/resources")
add_subdirectory("${PROJECT_SOURCE_DIR}/solv")
add_subdirectory("${PROJECT_SOURCE_DIR}/tools")
