## Process this file with cmake
#==============================================================================
#  NeXus - Neutron & X-ray Common Data Format
#
#  CMakeLists for building the NeXus file validation library
#  and command line validator nxvalidate
#
#  Copyright (C) 2015 Mark Koennecke
#
#  This library is free software; you can redistribute it and/or modify it under
#  the terms of the GNU Lesser General Public License as published by the Free
#  Software Foundation; either version 2 of the License, or (at your option) any
#  later version.
#
#  This library is distributed in the hope that it will be useful, but WITHOUT
#  ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS
#  FOR A PARTICULAR PURPOSE.  See the GNU Lesser General Public License for more
#  details.
#
#  You should have received a copy of the GNU Lesser General Public License
#  along with this library; if not, write to the Free Software Foundation, Inc.,
#  59 Temple Place, Suite 330, Boston, MA  02111-1307  USA
#
#  For further information, see <http://www.nexusformat.org>
#
#
#==============================================================================
cmake_minimum_required(VERSION 3.10.0)
include(FetchContent)

if(NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE "Debug" CACHE STRING
      "Choose the type of build, options are: Debug Release RelWithDebInfo MinSizeRel."
      FORCE)
endif(NOT CMAKE_BUILD_TYPE)

#The name of our project
project (cnxvalidate)
enable_testing()

message(STATUS ${PROJECT_SOURCE_DIR})
#set the module path
set(CMAKE_MODULE_PATH "${PROJECT_SOURCE_DIR}/cmake_include")
message(STATUS ${CMAKE_MODULE_PATH})

#------------------------------------------------------------------------------
# set the default installation prefix
#------------------------------------------------------------------------------
if(WIN32)
    #don't know yet what to put here
else()
    set(CMAKE_INSTALL_PREFIX "${PROJECT_SOURCE_DIR}/dist")
    #load module for GNU/Linux default installation directories
    include(GNUInstallDirs)
endif()

#-----------------------------------------------------------------------------
# define package version
#-----------------------------------------------------------------------------
set (cnxvalidate_VERSION_MAJOR 0)
set (cnxvalidate_VERSION_MINOR 5)
set (NXVERSINFO '1:0:0')

#-----------------------------------------------------------------------------
# include some general purpose modules
#-----------------------------------------------------------------------------
#include(Utilities)
#include(CompilerChecks)

include(FindPkgConfig)

FetchContent_Declare(
  libhdf5
  URL https://github.com/usnistgov/libhdf5-wasm/releases/download/v0.1.1/libhdf5-1_12_1-wasm.tar.gz
  URL_HASH SHA256=e9bb11d89c4f26fa79b9cf1dab6159640c7b184ebf00dc97b098cd4f6de49bfe
)
FetchContent_MakeAvailable(libhdf5)

# add libxml2 dependency that was previously built:
set (LIBXML2_INCLUDE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/libxml2/dist/include/libxml2)
set (LIBXML2_LIBRARY ${CMAKE_CURRENT_SOURCE_DIR}/libxml2/dist/lib/libxml2.a)
add_link_options(-sNODERAWFS=1 -sFORCE_FILESYSTEM=1)
#----------------------------------------------------------------
# Dependencies
#----------------------------------------------------------------
# find_package(HDF5 REQUIRED COMPONENTS HL)
find_package(LibXml2)

#--------------------------------------------------------------
# Our own stuff, really....
#-------------------------------------------------------------

include_directories(include ${libhdf5_SOURCE_DIR}/include ${LIBXML2_INCLUDE_DIR})
add_subdirectory(include)
add_subdirectory(src)
#add_subdirectory(test)

link_directories(${HDF5_LIBRARY_DIRS} )

add_executable (nxvalidate nxvmain.c)


target_link_libraries(nxvalidate hdf5-wasm ${LIBXML2_LIBRARIES} NXV_Static_Library z)

install (TARGETS nxvalidate
         DESTINATION ${CMAKE_INSTALL_BINDIR} COMPONENT Runtime)
#install (FILES nxvalidate.1
#         DESTINATION ${CMAKE_INSTALL_MANDIR}/man1 COMPONENT Documentation)
