
cmake_minimum_required(VERSION 3.2)

project(chiaki)

# Like option(), but the value can also be AUTO
macro(tri_option name desc default)
    set("${name}" "${default}" CACHE STRING "${desc}")
    set_property(CACHE "${name}" PROPERTY STRINGS AUTO ON OFF)
endmacro()

option(CHIAKI_ENABLE_TESTS "Enable tests for Chiaki" ON)
option(CHIAKI_ENABLE_CLI "Enable CLI for Chiaki" OFF)
option(CHIAKI_ENABLE_GUI "Enable Qt GUI" ON)
option(CHIAKI_ENABLE_ANDROID "Enable Android (Use only as part of the Gradle Project)" OFF)
option(CHIAKI_ENABLE_BOREALIS "Enable Borealis GUI (For Nintendo Switch or PC)" OFF)
tri_option(CHIAKI_ENABLE_SETSU "Enable libsetsu for touchpad input from controller" AUTO)
option(CHIAKI_LIB_ENABLE_OPUS "Use Opus as part of Chiaki Lib" ON)
if(CHIAKI_ENABLE_GUI OR CHIAKI_ENABLE_BOREALIS)
	set(CHIAKI_FFMPEG_DEFAULT ON)
else()
	set(CHIAKI_FFMPEG_DEFAULT AUTO)
endif()
tri_option(CHIAKI_ENABLE_FFMPEG_DECODER "Enable FFMPEG video decoder" ${CHIAKI_FFMPEG_DEFAULT})
tri_option(CHIAKI_ENABLE_PI_DECODER "Enable Raspberry Pi-specific video decoder (requires libraspberrypi0 and libraspberrypi-doc)" AUTO)
option(CHIAKI_LIB_ENABLE_MBEDTLS "Use mbedtls instead of OpenSSL as part of Chiaki Lib" OFF)
option(CHIAKI_LIB_MBEDTLS_EXTERNAL_PROJECT "Fetch Mbed TLS instead of using system-provided libs" OFF)
option(CHIAKI_LIB_OPENSSL_EXTERNAL_PROJECT "Use OpenSSL as CMake external project" OFF)
option(CHIAKI_GUI_ENABLE_SDL_GAMECONTROLLER "Use SDL Gamecontroller for Input" ON)
option(CHIAKI_CLI_ARGP_STANDALONE "Search for standalone argp lib for CLI" OFF)
tri_option(CHIAKI_USE_SYSTEM_JERASURE "Use system-provided jerasure instead of submodule" AUTO)
tri_option(CHIAKI_USE_SYSTEM_NANOPB "Use system-provided nanopb instead of submodule" AUTO)

set(CHIAKI_VERSION_MAJOR 2)
set(CHIAKI_VERSION_MINOR 2)
set(CHIAKI_VERSION_PATCH 0)
set(CHIAKI_VERSION ${CHIAKI_VERSION_MAJOR}.${CHIAKI_VERSION_MINOR}.${CHIAKI_VERSION_PATCH})

set(CPACK_PACKAGE_NAME "chiaki")
set(CPACK_PACKAGE_DESCRIPTION "Open Source PS4 remote play client")
set(CPACK_PACKAGE_VERSION_MAJOR ${CHIAKI_VERSION_MAJOR})
set(CPACK_PACKAGE_VERSION_MINOR ${CHIAKI_VERSION_MINOR})
set(CPACK_PACKAGE_VERSION_PATCH ${CHIAKI_VERSION_PATCH})
set(CPACK_DEBIAN_PACKAGE_SHLIBDEPS ON)
set(CPACK_DEBIAN_PACKAGE_DESCRIPTION ${CPACK_PACKAGE_DESCRIPTION})
set(CPACK_DEBIAN_PACKAGE_SECTION "games")
include(CPack)

set(CHIAKI_IS_SWITCH ${NSWITCH})

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake;${CMAKE_CURRENT_SOURCE_DIR}/setsu/cmake")

set(CMAKE_CXX_STANDARD 11)


if(CHIAKI_USE_SYSTEM_JERASURE)
	if(CHIAKI_USE_SYSTEM_JERASURE STREQUAL AUTO)
		find_package(Jerasure QUIET)
		set(CHIAKI_USE_SYSTEM_JERASURE ${Jerasure_FOUND})
	else()
		find_package(Jerasure REQUIRED)
		set(CHIAKI_USE_SYSTEM_JERASURE ON)
	endif()
endif()

find_package(PythonInterp 3 REQUIRED) # Make sure nanopb doesn't find Python 2.7 because Python 2 should just die.

if(CHIAKI_USE_SYSTEM_NANOPB)
	if(CHIAKI_USE_SYSTEM_NANOPB STREQUAL AUTO)
		find_package(Nanopb QUIET)
		set(CHIAKI_USE_SYSTEM_NANOPB ${Nanopb_FOUND})
	else()
		find_package(Nanopb REQUIRED)
		set(CHIAKI_USE_SYSTEM_NANOPB ON)
	endif()
endif()

add_subdirectory(third-party)

add_definitions(-DCHIAKI_VERSION_MAJOR=${CHIAKI_VERSION_MAJOR} -DCHIAKI_VERSION_MINOR=${CHIAKI_VERSION_MINOR} -DCHIAKI_VERSION_PATCH=${CHIAKI_VERSION_PATCH} -DCHIAKI_VERSION=\"${CHIAKI_VERSION}\")

if(CHIAKI_LIB_OPENSSL_EXTERNAL_PROJECT)
	include(OpenSSLExternalProject)
endif()

if(CHIAKI_LIB_ENABLE_MBEDTLS)
	add_definitions(-DCHIAKI_LIB_ENABLE_MBEDTLS)
	if(CHIAKI_LIB_MBEDTLS_EXTERNAL_PROJECT)
		set(FETCHCONTENT_QUIET CACHE BOOL FALSE)
		include(FetchContent)
		set(ENABLE_TESTING CACHE INTERNAL OFF)
		set(ENABLE_PROGRAMS CACHE INTERNAL OFF)
		set(USE_SHARED_MBEDTLS_LIBRARY CACHE INTERNAL OFF)
		FetchContent_Declare(
			mbedtls
			GIT_REPOSITORY https://github.com/Mbed-TLS/mbedtls.git
			GIT_TAG 8b3f26a5ac38d4fdccbc5c5366229f3e01dafcc0 # v2.28.0
			GIT_PROGRESS TRUE
		)
		FetchContent_MakeAvailable(mbedtls)
	endif()
endif()




add_subdirectory(lib)


if(CHIAKI_ENABLE_ANDROID)
	add_subdirectory(android/app)
endif()

if(CHIAKI_ENABLE_BOREALIS)
	add_subdirectory(switch)
endif()
