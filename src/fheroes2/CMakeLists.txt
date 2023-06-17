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


file(GLOB_RECURSE FHEROES2_SOURCES CONFIGURE_DEPENDS *.cpp)

if(MINGW)
	set(MINGW_LIBRARIES mingw32 winmm)
endif(MINGW)

get_filename_component(CONFIGURE_FHEROES2_DATA_ABSOLUTE
	${CONFIGURE_FHEROES2_DATA} ABSOLUTE
	BASE_DIR ${CMAKE_INSTALL_PREFIX}
	CACHE
	)

add_executable(fheroes2 ${FHEROES2_SOURCES})
target_compile_definitions(fheroes2 PRIVATE
	CONFIGURE_FHEROES2_DATA=${CONFIGURE_FHEROES2_DATA_ABSOLUTE}
	)
target_include_directories(fheroes2 PRIVATE
	agg
	ai
	army
        audio
	battle
	campaign
	castle
	dialog
	editor
	game
	gui
	h2d
	heroes
	image
	kingdom
	maps
	monster
	objects
	pocketpc
	resource
	spell
	system
	world
	)
target_link_libraries(fheroes2
	${MINGW_LIBRARIES}  # Be sure define it first. Beware WinMain@16 error!!
	${SDL_MIXER_LIBRARIES}
        ${SDL_GFX_LIBRARIES}
	engine
	Threads::Threads
	ZLIB::ZLIB
        wayland-client
	)

install(TARGETS fheroes2 DESTINATION ${CMAKE_INSTALL_BINDIR})

#
# Sailfish
#
install(TARGETS fheroes2
    RUNTIME DESTINATION bin
)
