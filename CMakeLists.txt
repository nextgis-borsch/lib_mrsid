################################################################################
# Project:  CMake build scripts for ECW library
# Purpose:  CMake build scripts
# Author:   Alexander Lisovenko, alexander.lisovenko@gmail.com
################################################################################
# Copyright (C) 2016, NextGIS <info@nextgis.com>
#
# Permission is hereby granted, free of charge, to any person obtaining a
# copy of this software and associated documentation files (the "Software"),
# to deal in the Software without restriction, including without limitation
# the rights to use, copy, modify, merge, publish, distribute, sublicense,
# and/or sell copies of the Software, and to permit persons to whom the
# Software is furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included
# in all copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
# OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
# THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
# FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
# DEALINGS IN THE SOFTWARE.
################################################################################

cmake_minimum_required(VERSION 2.8.12 FATAL_ERROR)

project(mrsid NONE)

# some init settings
set(CMAKE_COLOR_MAKEFILE ON)
# set path to additional CMake modules
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH})


include(GNUInstallDirs)

set(PUBLIC_HDRS
)

set(DOWNLOAD_PATH_BASE "http://176.9.38.120/third-party/mrsid/current")
if(WIN32)
    if(CMAKE_CL_64)
        if(MSVC11)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86_64/vc110/lib.zip" )
        elseif(MSVC12)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86_64/vc120/lib.zip" )
        elseif(MSVC14)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86_64/vc140/lib.zip" )
        endif()
    else()
        if(MSVC11)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86/vc110/lib.zip" )
        elseif(MSVC12)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86/vc120/lib.zip" )
        elseif(MSVC14)
            set(DOWNLOAD_PATH "${DOWNLOAD_PATH_BASE}/win/x86/vc140/lib.zip" )
        endif()
    endif()
endif()

if (NOT DOWNLOAD_PATH)
    message(FATAL_ERROR "There is no relevant libs")
    return()
endif()

file(DOWNLOAD ${DOWNLOAD_PATH} "${CMAKE_BINARY_DIR}/lib.zip" STATUS DOWNLOAD_STATUS SHOW_PROGRESS)

list(GET DOWNLOAD_STATUS 0 DOWNLOAD_STATUS_CODE)
if (NOT ${DOWNLOAD_STATUS_CODE} EQUAL 0)
    message(FATAL_ERROR "Can not download mrsid lib by url:\n\t${DOWNLOAD_PATH}.\nGet error:\n\t${DOWNLOAD_STATUS_CODE}")
    return()
endif()

execute_process(
    COMMAND ${CMAKE_COMMAND} -E tar xzf ${CMAKE_BINARY_DIR}/lib.zip
    WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
)

set(LIB_NAME ${PROJECT_NAME})

add_library(${LIB_NAME} INTERFACE)

include(util)
check_version(MRSID_MAJOR_VERSION MRSID_MINOR_VERSION MRSID_REV_VERSION)
set(VERSION ${MRSID_MAJOR_VERSION}.${MRSID_MINOR_VERSION}.${MRSID_REV_VERSION})
#report version
report_version(${PROJECT_NAME} ${VERSION})


target_link_libraries(
    ${LIB_NAME} INTERFACE "${CMAKE_BINARY_DIR}/lib/lib/lti_dsdk.lib"
)

export(TARGETS ${LIB_NAME} FILE ${LIB_NAME}-exports.cmake)

if(NOT SKIP_INSTALL_LIBRARIES AND NOT SKIP_INSTALL_ALL )
    install(DIRECTORY "${CMAKE_BINARY_DIR}/lib/lib/" DESTINATION "${CMAKE_INSTALL_FULL_LIBDIR}")
    install(DIRECTORY "${CMAKE_BINARY_DIR}/lib/bin/" DESTINATION "${CMAKE_INSTALL_FULL_BINDIR}")
endif()
if(NOT SKIP_INSTALL_HEADERS AND NOT SKIP_INSTALL_ALL )
    install(DIRECTORY "${CMAKE_BINARY_DIR}/lib/include/" DESTINATION "${CMAKE_INSTALL_FULL_INCLUDEDIR}/libmrsid")
endif() 
