cmake_minimum_required(VERSION 3.0)
project(rencoder_solution)

#################################
# project common settings
#################################
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin")
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${PROJECT_SOURCE_DIR}/bin/lib")
set(CMAKE_CXX_STANDARD 14)
if (MSVC)
    #set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    #set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
    #set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
    set(CMAKE_CXX_FLAGS_RELEASE "/MT /Ot")
    set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Od")
    set(CMAKE_C_FLAGS_RELEASE "/MT /Ot")
    set(CMAKE_C_FLAGS_DEBUG "/MTd /Zi /Od")
    add_definitions(-D_CRT_SECURE_NO_WARNINGS)
elseif (UNIX)
    set(CMAKE_POSITION_INDEPENDENT_CODE ON)
endif()

#################################
# Build path configuration (generic no-config case e.g. mingw)
#################################
set( CMAKE_RUNTIME_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/bin" )
set( CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/lib" )
set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY "${CMAKE_SOURCE_DIR}/lib" )
# Build path configuration (multi-config builds e.g. msvc)
foreach( OUTPUTCONFIG ${CMAKE_CONFIGURATION_TYPES} )
    string( TOUPPER ${OUTPUTCONFIG} OUTPUTCONFIG )
    set( CMAKE_RUNTIME_OUTPUT_DIRECTORY_${OUTPUTCONFIG} "${CMAKE_SOURCE_DIR}/bin_${OUTPUTCONFIG}" )
    set( CMAKE_LIBRARY_OUTPUT_DIRECTORY_${OUTPUTCONFIG} "${CMAKE_SOURCE_DIR}/lib_${OUTPUTCONFIG}" )
    set( CMAKE_ARCHIVE_OUTPUT_DIRECTORY_${OUTPUTCONFIG} "${CMAKE_SOURCE_DIR}/lib_${OUTPUTCONFIG}" )
endforeach( OUTPUTCONFIG CMAKE_CONFIGURATION_TYPES )
set (RENCODER_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/src")

#################################
# for extern library
#################################
set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")
if (EXISTS ${PROJECT_SOURCE_DIR}/../rhythmus-lib)
    if (UNIX)
      set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${PROJECT_SOURCE_DIR}/../rhythmus-lib/cmake/modules_linux")
    else()
      set(CMAKE_MODULE_PATH "${CMAKE_MODULE_PATH};${PROJECT_SOURCE_DIR}/../rhythmus-lib/cmake/modules")
    endif()
    message("rhythmus-lib found. Use precompiled library.")
else()
    message("No rhythmus-lib found. Use system library.")
endif()

find_package(Iconv QUIET)       # might fail in MSVC but okay
find_package(ZLIB)
find_package(ZIP)
find_package(OpenSSL)
find_package(OGG)
find_package(Vorbis)
find_package(VorbisEnc QUIET)   # no cmake in Win32
find_package(FLAC)

if (ZLIB_FOUND AND ZIP_FOUND)
    message("Archive file feature available.")
	add_definitions(-DUSE_ZLIB)
endif ()

if (OPENSSL_FOUND)
    message("OpenSSL Found.")
	add_definitions(-DUSE_OPENSSL)
endif ()

#################################
# add rparser
#################################
add_subdirectory("${CMAKE_SOURCE_DIR}/../rparser/src"
                 "${CMAKE_SOURCE_DIR}/build/rparser"
                 EXCLUDE_FROM_ALL)
set (RPARSER_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/../rparser/include")

#################################
# timidity
#################################
add_subdirectory(timidity)
set (TIMIDITY_INCLUDE_DIR "${CMAKE_SOURCE_DIR}/timidity")

#################################
# targets
#################################

# library
add_subdirectory("${PROJECT_SOURCE_DIR}/src")

# encoder binary
add_subdirectory("${PROJECT_SOURCE_DIR}/encoder")

# don't use test proj if you just want to use rencoder library
add_subdirectory("${PROJECT_SOURCE_DIR}/test")
