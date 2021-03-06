#
#  For more information, please see: http://software.sci.utah.edu
# 
#  The MIT License
# 
#  Copyright (c) 2009 Scientific Computing and Imaging Institute,
#  University of Utah.
# 
#  
#  Permission is hereby granted, free of charge, to any person obtaining a
#  copy of this software and associated documentation files (the "Software"),
#  to deal in the Software without restriction, including without limitation
#  the rights to use, copy, modify, merge, publish, distribute, sublicense,
#  and/or sell copies of the Software, and to permit persons to whom the
#  Software is furnished to do so, subject to the following conditions:
# 
#  The above copyright notice and this permission notice shall be included
#  in all copies or substantial portions of the Software.
# 
#  THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS
#  OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
#  FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL
#  THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
#  LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING
#  FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER
#  DEALINGS IN THE SOFTWARE.
#

###########################################
# GLEW project

CMAKE_MINIMUM_REQUIRED(VERSION 2.8)
PROJECT(glew)

###########################################
# Add include directories

INCLUDE_DIRECTORIES(include)

###########################################
# Find OpenGL directories

FIND_PACKAGE( OpenGL )
IF(OPENGL_LIBRARIES AND OPENGL_INCLUDE_DIR)
  SET(GL_LIBRARY ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY} CACHE STRING "OpenGL Libs")
ENDIF(OPENGL_LIBRARIES AND OPENGL_INCLUDE_DIR)
IF (WIN32)
  SET(GL_LIBRARY ${OPENGL_gl_LIBRARY} ${OPENGL_glu_LIBRARY} gdi32 CACHE STRING "OpenGL Libs")
ENDIF(WIN32)

###########################################
# Set the library name and include path
# so the other projects can use it

SET(SCI_GLEW_LIBRARY glew CACHE INTERNAL "Glew library name." FORCE)
SET(SCI_GLEW_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE INTERNAL "Public Glew include directories." FORCE)

###########################################
# Add the sources needed for the project

SET(GLEW_SRCS
  src/glew.c
  src/glewinfo.c
  src/visualinfo.c
  include/GL/glew.h
  include/GL/glxew.h
  include/GL/wglew.h)

###########################################
# Specific defines that need to be set
# STATIC - we want glew to be a static library

ADD_DEFINITIONS(-DGLEW_STATIC)

###########################################
# Build the libary statically

ADD_LIBRARY(${SCI_GLEW_LIBRARY} STATIC ${GLEW_SRCS})
TARGET_LINK_LIBRARIES(${SCI_GLEW_LIBRARY} ${GL_LIBRARY})

###########################################
# Set Windows options (Visual Studio)
# Force Visual Studio to link against the 
# version that is in the redistributable
# Options for Visual Studio to limit 
# the number of warnings.

IF(WIN32)
  SET_TARGET_PROPERTIES(${SCI_GLEW_LIBRARY}
    PROPERTIES
    COMPILE_FLAGS "-W3 -wd4267 -wd4244"
  )
  ADD_DEFINITIONS(-D_BIND_TO_CURRENT_VCLIBS_VERSION=1)
ENDIF(WIN32)

IF (NOT WIN32)
  SET_TARGET_PROPERTIES(glew PROPERTIES 
    COMPILE_FLAGS "-Wno-deprecated-declarations"
  )
ENDIF (NOT WIN32)

SET(SCI_GLEW_LIBRARY_DIR ${CMAKE_CURRENT_BINARY_DIR})
SET(GLEW_USE_FILE "${CMAKE_CURRENT_SOURCE_DIR}/UseGlew.cmake")

CONFIGURE_FILE(GlewConfig.cmake.in "${CMAKE_CURRENT_BINARY_DIR}/GlewConfig.cmake" @ONLY)
EXPORT(TARGETS ${SCI_GLEW_LIBRARY} FILE GlewExports.cmake)
