﻿project(ZLMediaKit)
cmake_minimum_required(VERSION 3.1.3)
#使能c++11
set(CMAKE_CXX_STANDARD 11)

#加载自定义模块
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${PROJECT_SOURCE_DIR}/cmake")
#设置库文件路径
set(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)
#设置可执行程序路径
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)

#设置工程源码根目录
set(ToolKit_Root ${CMAKE_SOURCE_DIR}/3rdpart/ZLToolKit/src)
set(MediaKit_Root ${CMAKE_SOURCE_DIR}/src)

#设置头文件目录
INCLUDE_DIRECTORIES(${ToolKit_Root})
INCLUDE_DIRECTORIES(${MediaKit_Root})

#收集源代码
file(GLOB ToolKit_src_list ${ToolKit_Root}/*/*.cpp ${ToolKit_Root}/*/*.h ${ToolKit_Root}/*/*.c)
file(GLOB MediaKit_src_list ${MediaKit_Root}/*/*.cpp ${MediaKit_Root}/*/*.h ${MediaKit_Root}/*/*.c)

#去除win32的适配代码
if (NOT WIN32)
    list(REMOVE_ITEM ToolKit_src_list ${ToolKit_Root}/win32/getopt.c)
else()
     #防止Windows.h包含Winsock.h
    add_definitions(-DWIN32_LEAN_AND_MEAN -DMP4V2_NO_STDINT_DEFS)
endif ()

set(ENABLE_HLS true)
set(ENABLE_OPENSSL true)
set(ENABLE_MYSQL true)
set(ENABLE_MP4V2 true)
set(ENABLE_FAAC true)
set(ENABLE_X264 true)

#添加两个静态库
if(ENABLE_HLS)
    message(STATUS "ENABLE_HLS defined")
    add_definitions(-DENABLE_HLS)
    set(MediaServer_Root ${CMAKE_SOURCE_DIR}/3rdpart/media-server)
    set(LINK_LIB_LIST zlmediakit zltoolkit mpeg)
else()
    set(LINK_LIB_LIST zlmediakit zltoolkit)
endif()

#查找openssl是否安装
find_package(OpenSSL QUIET)
if (OPENSSL_FOUND AND ENABLE_OPENSSL)
    message(STATUS "found library:${OPENSSL_LIBRARIES},ENABLE_OPENSSL defined")
    include_directories(${OPENSSL_INCLUDE_DIR})
    add_definitions(-DENABLE_OPENSSL)
    list(APPEND LINK_LIB_LIST ${OPENSSL_LIBRARIES})
endif ()

#查找mysql是否安装
find_package(MYSQL QUIET)
if (MYSQL_FOUND AND ENABLE_MYSQL)
    message(STATUS "found library:${MYSQL_LIBRARIES},ENABLE_MYSQL defined")
    include_directories(${MYSQL_INCLUDE_DIR})
    add_definitions(-DENABLE_MYSQL)
    list(APPEND LINK_LIB_LIST ${MYSQL_LIBRARIES})
endif ()

#查找MP4V2是否安装
find_package(MP4V2 QUIET)
if (MP4V2_FOUND AND ENABLE_MP4V2)
    include_directories(${MP4V2_INCLUDE_DIR})
    list(APPEND LINK_LIB_LIST ${MP4V2_LIBRARY})
    add_definitions(-DENABLE_MP4V2)
    message(STATUS "found library:${MP4V2_LIBRARY},ENABLE_MP4V2 defined")
endif ()

#查找x264是否安装
find_package(X264 QUIET)
if (X264_FOUND AND ENABLE_X264)
    message(STATUS "found library:${X264_LIBRARIES},ENABLE_X264 defined")
    include_directories(${X264_INCLUDE_DIRS})
    add_definitions(-DENABLE_X264)
    list(APPEND LINK_LIB_LIST ${X264_LIBRARIES})
endif ()

#查找faac是否安装
find_package(FAAC QUIET)
if (FAAC_FOUND AND ENABLE_FAAC)
    message(STATUS "found library:${FAAC_LIBRARIES},ENABLE_FAAC defined")
    include_directories(${FAAC_INCLUDE_DIR})
    add_definitions(-DENABLE_FAAC)
    list(APPEND LINK_LIB_LIST ${FAAC_LIBRARIES})
endif ()

#添加库
add_library(zltoolkit STATIC ${ToolKit_src_list})
add_library(zlmediakit STATIC ${MediaKit_src_list})

#libmpeg
if(ENABLE_HLS)
    aux_source_directory(${MediaServer_Root}/libmpeg/include src_mpeg)
    aux_source_directory(${MediaServer_Root}/libmpeg/source src_mpeg)
    include_directories(${MediaServer_Root}/libmpeg/include)
    add_library(mpeg STATIC ${src_mpeg})
endif()

if (WIN32)
    list(APPEND LINK_LIB_LIST WS2_32 Iphlpapi shlwapi)
elseif(NOT ANDROID OR IOS)
    list(APPEND LINK_LIB_LIST pthread)
endif ()

#测试程序
add_subdirectory(tests)

#主服务器
if (CMAKE_SYSTEM_NAME MATCHES "Linux")
    add_subdirectory(server)
elseif (CMAKE_SYSTEM_NAME MATCHES "Darwin")
    add_subdirectory(server)
endif ()

























