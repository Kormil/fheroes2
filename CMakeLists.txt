###########################################################################
#   fheroes2: https://github.com/ihhub/fheroes2                           #
#   Copyright (C) 2022 - 2023                                             #
#                                                                         #
#   This program is free software; you can redistribute it and/or modify  #
#   it under the terms of the GNU General Public License as published by  #
#   the Free Software Foundation; either version 2 of the License, or     #
#   (at your option) any later version.                                   #
#                                                                         #
#   This program is distributed in the hope that it will be useful,       #
#   but WITHOUT ANY WARRANTY; without even the implied warranty of        #
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         #
#   GNU General Public License for more details.                          #
#                                                                         #
#   You should have received a copy of the GNU General Public License     #
#   along with this program; if not, write to the                         #
#   Free Software Foundation, Inc.,                                       #
#   59 Temple Place - Suite 330, Boston, MA  02111-1307, USA.             #
###########################################################################

cmake_minimum_required(VERSION 3.14)

project(fheroes2 VERSION 1.0.9 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_BUILD_TYPE Release)

cmake_policy(SET CMP0074 NEW)

include(GNUInstallDirs)
list(APPEND CMAKE_MODULE_PATH ${CMAKE_CURRENT_SOURCE_DIR}/cmake)

#
# Sailfish
#
include(FindPkgConfig)
pkg_search_module(SAILFISH sailfishapp REQUIRED)

install(DIRECTORY translations
    DESTINATION share/fheroes2
    FILES_MATCHING PATTERN "*.qm"
)
install(FILES fheroes2.desktop
    DESTINATION share/applications
)
install(FILES icons/86x86/fheroes2.png
    DESTINATION share/icons/hicolor/86x86/apps
)
install(FILES icons/108x108/fheroes2.png
    DESTINATION share/icons/hicolor/108x108/apps
)
install(FILES icons/128x128/fheroes2.png
    DESTINATION share/icons/hicolor/128x128/apps
)
install(FILES icons/172x172/fheroes2.png
    DESTINATION share/icons/hicolor/172x172/apps
)

add_definitions(-DEGL_NO_X11)
add_definitions(-DTARGET_MOBILE)

#
# Compile-time options
#
option(ENABLE_STRICT_COMPILATION "Enable strict compilation mode (turns warnings into errors)" OFF)
option(ENABLE_IMAGE "Enable the use of SDL_image (requires libpng)" ON)
option(ENABLE_TOOLS "Enable the build of additional tools" OFF)
set(CONFIGURE_FHEROES2_DATA "/usr/share/${PROJECT_NAME}" CACHE STRING "System fheroes2 game directory")

set(USE_SDL_VERSION SDL2 CACHE STRING "Version of the SDL library used")
set(FHEROES2_DATA ${CMAKE_INSTALL_DATADIR}/${PROJECT_NAME} CACHE STRING "Path to the fheroes2 data directory (relative to CMAKE_INSTALL_PREFIX)")

# For ComboBox in CMake GUI
set_property(CACHE USE_SDL_VERSION PROPERTY STRINGS SDL2)

#
# Library & feature detection
#
find_package(${USE_SDL_VERSION} REQUIRED)
find_package(${USE_SDL_VERSION}_mixer REQUIRED)
find_package(${USE_SDL_VERSION}_gfx REQUIRED)
find_package(ZLIB REQUIRED)
find_package(Threads)

# TODO: Modern versions of SDL2* packages provide the CMake config files that define the SDL2::SDL2, SDL2_mixer::SDL2_mixer and SDL2_image::SDL2_image
# TODO: imported targets, but older versions do not. To solve this problem, we have to use FindSDL2*.cmake modules from the cmake/ subdirectory and
# TODO: define aliases for imported targets provided by these modules.

# TODO: remove this alias and module from the cmake/ subdirectory as soon as the distributions start shipping a newer version of SDL_mixer
add_library(${USE_SDL_VERSION}_mixer::${USE_SDL_VERSION}_mixer ALIAS ${USE_SDL_VERSION}::Mixer)

if(ENABLE_IMAGE)
	find_package(${USE_SDL_VERSION}_image REQUIRED)
	find_package(PNG REQUIRED)

	# TODO: remove this alias and module from the cmake/ subdirectory as soon as the distributions start shipping a newer version of SDL_image
	add_library(${USE_SDL_VERSION}_image::${USE_SDL_VERSION}_image ALIAS ${USE_SDL_VERSION}::Image)
endif(ENABLE_IMAGE)

set (BOOST_CXX_FLAGS "-Wno-deprecated-register -std=c++14")
find_package(Boost 1.66.0 COMPONENTS filesystem system REQUIRED)

#
# Source files
#
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR})

add_subdirectory(src)

#
# Translation files
#



if(NOT WIN32)
        find_program(MAKE_EXECUTABLE make)
        find_program(SED_EXECUTABLE sed)
        find_program(XGETTEXT_EXECUTABLE xgettext)

        file(GLOB_RECURSE ALL_SOURCES CONFIGURE_DEPENDS src/*.cpp)
        add_custom_target(
                ${PROJECT_NAME}.pot
                COMMAND ${XGETTEXT_EXECUTABLE} -d ${PROJECT_NAME} -C -F -k_ -k_n:1,2 -o ${PROJECT_NAME}.pot ${ALL_SOURCES}
                COMMAND ${SED_EXECUTABLE} -i~ -e "s/, c-format//" ${PROJECT_NAME}.pot
                DEPENDS fheroes2
                WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/src/dist
                )

        add_custom_target(
                translations
                ALL
                COMMAND ${MAKE_EXECUTABLE}
                DEPENDS ${PROJECT_NAME}.pot
                WORKING_DIRECTORY ${CMAKE_SOURCE_DIR}/files/lang
                )
endif(NOT WIN32)

#
# Installation of auxiliary files
#
install(
	FILES fheroes2.key
        DESTINATION share/fheroes2
	)

install(
	DIRECTORY files/data/
        DESTINATION share/fheroes2/files/data
	FILES_MATCHING
	PATTERN *.h2d
	)

install(
        FILES LICENSE changelog.txt
        DESTINATION share/fheroes2
	)

if(GET_HOMM2_DEMO)
	include(FetchContent)
	set(FETCHCONTENT_QUIET OFF)
	FetchContent_Declare(
		demodata
		URL https://archive.org/download/HeroesofMightandMagicIITheSuccessionWars_1020/h2demo.zip
		URL_HASH SHA256=12048c8b03875c81e69534a3813aaf6340975e77b762dc1b79a4ff5514240e3c
		)
	FetchContent_MakeAvailable(demodata)
	file(COPY ${demodata_SOURCE_DIR}/DATA/ DESTINATION data PATTERN *.*)
	file(COPY ${demodata_SOURCE_DIR}/MAPS/ DESTINATION maps PATTERN *.*)

	install(DIRECTORY ${demodata_SOURCE_DIR}/DATA/ DESTINATION
		${CONFIGURE_FHEROES2_DATA}/data FILES_MATCHING PATTERN *.*)
	install(DIRECTORY ${demodata_SOURCE_DIR}/MAPS/ DESTINATION
		${CONFIGURE_FHEROES2_DATA}/maps FILES_MATCHING PATTERN *.*)
endif(GET_HOMM2_DEMO)

if(WIN32)
	install(
		FILES script/demo/download_demo_version.bat script/demo/download_demo_version.ps1
		DESTINATION ${CMAKE_INSTALL_DOCDIR}/demo
		)
	install(
		FILES script/homm2/extract_homm2_resources.bat script/homm2/extract_homm2_resources.ps1
		DESTINATION ${CMAKE_INSTALL_DOCDIR}/homm2
		)
else(WIN32)
	install(
		DIRECTORY files/lang/
		DESTINATION ${CONFIGURE_FHEROES2_DATA}/files/lang
		FILES_MATCHING
		PATTERN *.mo
		)
endif(WIN32)
