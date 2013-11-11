# Copyright (c) 2011 Prime Focus Film.
#
# All rights reserved.
#
# Redistribution and use in source and binary forms, with or without
# modification, are permitted provided that the following conditions
# are met:
#
# Redistributions of source code must retain the above copyright
# notice, this list of conditions and the following disclaimer.
# Redistributions in binary form must reproduce the above copyright
# notice, this list of conditions and the following disclaimer in the
# documentation and/or other materials provided with the
# distribution. Neither the name of Prime Focus Film nor the
# names of its contributors may be used to endorse or promote
# products derived from this software without specific prior written
# permission.
#
# THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS
# "AS IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT
# LIMITED TO, THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS
# FOR A PARTICULAR PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE
# COPYRIGHT OWNER OR CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT,
# INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
# (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
# SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION)
# HOWEVER CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT,
# STRICT LIABILITY, OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE)
# ARISING IN ANY WAY OUT OF THE USE OF THIS SOFTWARE, EVEN IF ADVISED
# OF THE POSSIBILITY OF SUCH DAMAGE.

# Author : Antoine P.

## CMake compatibility issues: don't modify this, please!
CMAKE_MINIMUM_REQUIRED( VERSION 2.8)

if(COMMAND cmake_policy)
     cmake_policy(SET CMP0003 NEW)
	 cmake_policy(SET CMP0015 NEW)  # to supress relative library directory warnings
endif(COMMAND cmake_policy)

set( CMAKE_VERBOSE_MAKEFILE on )



FIND_PACKAGE (HDF5)

SET(CMAKE_MODULE_PATH "/redpawFX/dev/gitHub/redpawFX/Field3DMayaPlugin/cmake")
FIND_PACKAGE (Maya)

if (MAYA_FOUND)

IF ( UNIX AND NOT APPLE )
	ADD_DEFINITIONS(-DLINUX)
ENDIF()
if( APPLE )
	ADD_DEFINITIONS(-DMAC_PLUGIN)
	ADD_DEFINITIONS(-DOSMac_)
	ADD_DEFINITIONS(-DOSMac_MachO)
ENDIF()

SET(CMAKE_CXX_WARNING_LEVEL 4)

MARK_AS_ADVANCED(CMAKE_BACKWARDS_COMPATIBILITY)
## allow more human readable "if then else" constructs
SET( CMAKE_ALLOW_LOOSE_LOOP_CONSTRUCTS TRUE )
## Use verbose make file
SET ( CMAKE_VERBOSE_MAKEFILE TRUE )


set (PLUGIN_NAME "field3DCache")
set (CMAKE_PROJECT_NAME "field3DCache")

if(APPLE)
	set (PLUGIN_SUFFIX ".bundle")
	set (CMAKE_SHARED_LIBRARY_SUFFIX ".bundle")
	set (SUFFIX ".bundle")
ENDIF()

if(WIN32)
	set (PLUGIN_SUFFIX ".mll")
	set (CMAKE_SHARED_LIBRARY_SUFFIX ".mll")
	set (SUFFIX ".mll")
ENDIF()

if (UNIX)
	set (PLUGIN_SUFFIX ".so")
	set (CMAKE_SHARED_LIBRARY_SUFFIX ".so")
	set (SUFFIX ".so")
ENDIF()

set (STD_INCLUDES "." ".." "/usr/X11R6/include" "/usr/include" "/usr/include/CC")
set (STD_LIB "/usr/lib/" "/usr/lib64")


add_definitions(-D_BOOL)
add_definitions(-DREQUIRE_IOSTREAM)
if (!WIN32)
	add_definitions(-fPIC)
	add_definitions(-O3)
	add_definitions(-pthread)
endif()

FILE(GLOB field3DCPP "src/*.cpp")
FILE(GLOB field3DINCLUDES "/src*.h")

SET(TARGET_H ${field3DINCLUDES})
SET(TARGET_SRC ${field3DCPP})

SET(MAYA_LIBS OpenMaya Foundation OpenMayaAnim OpenMayaFX OpenMayaUI OpenMayaRender)

INCLUDE_DIRECTORIES (${STD_INCLUDES})
LINK_DIRECTORIES( ${STD_LIB}  )

# use system installed versions of these instead
#add_custom_library ( HDF5    hdf5.h          ${LINK_TYPE}  "hdf5"      )
INCLUDE_DIRECTORIES( "/redpawFX/release/field3d/1.4.0/include" )
INCLUDE_DIRECTORIES( ${HDF5_INCLUDE_DIRS} )
INCLUDE_DIRECTORIES( ${MAYA_INCLUDE_DIRS} )

SET(FIELD3DLIBS "/redpawFX/release/field3d/1.4.0/lib" )
LINK_DIRECTORIES( ${MAYA_LIBRARY_DIR} ${FIELD3DLIBS} )

message(STATUS "MAYA LIB DIR: " ${MAYA_LIBRARY_DIR})

ADD_LIBRARY(${PLUGIN_NAME} SHARED ${TARGET_H} ${TARGET_SRC})
set_target_properties(${PLUGIN_NAME} PROPERTIES PREFIX "")

## need these to build for windows or else the plugin won't load
IF(WIN32)
	set_target_properties(${PLUGIN_NAME} PROPERTIES LINK_FLAGS "/export:initializePlugin /export:uninitializePlugin")
ENDIF()

SET(MAYA_LIBS OpenMaya Foundation OpenMayaAnim OpenMayaFX OpenMayaUI OpenMayaRender)
TARGET_LINK_LIBRARIES(${PLUGIN_NAME} ${MAYA_LIBS} Field3D hdf5 IlmThread Iex Imath Half)


# Hack:
# We must add ilmbase/OpenEXR in include directories as well
# because ilmbase reference OpenEXR header as
#       #include "Iex.h"
# instead of
#       #include "OpenEXR/Iex.h"

#if( ILMBASE_ROOT_DIR )
include_directories( /usr/include/OpenEXR )
#endif()


ENDIF(MAYA_FOUND)

