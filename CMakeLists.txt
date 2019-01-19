# Based on original work by David Manura
# Copyright (C) 2007-2012 LuaDist.
# Copyright (C) 2013 Brian Sidebotham

# Redistribution and use of this file is allowed according to the terms of the
# MIT license.
#
# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:
#
# The above copyright notice and this permission notice shall be included in all
# copies or substantial portions of the Software.
#
# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.


project( openssl )
cmake_minimum_required( VERSION 3.1.0 )

set( VERSION_MAJOR 1  )
set( VERSION_MINOR 1  )
set( VERSION_PATCH 1  )

set( VERSION_STRING ${VERSION_MAJOR}.${VERSION_MINOR}.${VERSION_PATCH} )
set( LIB_VERSION ${VERSION_MAJOR}.${VERSION_MINOR} )
set( LIB_SOVERSION ${VERSION_MAJOR}.${VERSION_MINOR} )

include_directories ( BEFORE SYSTEM
    ${CMAKE_CURRENT_BINARY_DIR}/crypto
    ${CMAKE_CURRENT_BINARY_DIR}/ssl
    ${CMAKE_CURRENT_BINARY_DIR}/include
    crypto . )

add_definitions( -DOPENSSL_NO_ASM )
add_definitions( -DOPENSSL_NO_STATIC_ENGINE )
add_definitions( "-DOPENSSLDIR=\"/usr/local/ssl\"" )
add_definitions( "-DENGINESDIR=\"/usr/local/engines-1.1\"" )

if ( APPLE )
  set( CMAKE_MACOSX_RPATH ON )
  add_definitions( -DOPENSSL_SYSNAME_MACOSX )
endif()

if( WIN32 AND NOT CYGWIN )
  add_definitions( -DOPENSSL_SYSNAME_WIN32 )
  add_definitions( -DWIN32_LEAN_AND_MEAN )

  # avoid conflict: ocsp.h and wincrypt.h
  add_definitions( -D_WINDLL )
endif()

if( MINGW )
  set( CMAKE_SHARED_LINKER_FLAGS "-Wl,--export-all" )
endif()

include(CheckTypeSize)
check_type_size( "long" LONG_INT )
check_type_size( "long long" LONG_LONG_INT )
check_type_size( "int" INT )
if ( HAVE_LONG_INT AND (${LONG_INT} EQUAL 8) )
  set ( SIXTY_FOUR_BIT_LONG ON )
elseif ( HAVE_LONG_LONG_INT AND (${LONG_LONG_INT} EQUAL 8) )
  set ( SIXTY_FOUR_BIT ON )
else()
  set ( THIRTY_TWO_BIT ON )
endif()

if ( WIN32 AND MINGW AND NOT CYGWIN )
  set( OPENSSL_EXPORT_VAR_AS_FUNCTION true )
endif()

add_subdirectory( crypto )
add_subdirectory( ssl )
add_subdirectory( apps )

file( COPY ${PROJECT_SOURCE_DIR}/include/internal DESTINATION include )
file( COPY ${PROJECT_SOURCE_DIR}/include/openssl DESTINATION include )

configure_file( ${PROJECT_SOURCE_DIR}/opensslconf.h.cmake
  ${PROJECT_BINARY_DIR}/include/openssl/opensslconf.h )

file( READ ${PROJECT_SOURCE_DIR}/c_rehash.cmake C_REHASH )
string( REPLACE "@OPENSSLDIR@" "${OPENSSLDIR}" C_REHASH "${C_REHASH}" )
string( REPLACE "@CMAKE_INSTALL_PREFIX@" "${CMAKE_INSTALL_PREFIX}" C_REHASH "${C_REHASH}" )
file( WRITE ${PROJECT_BINARY_DIR}/c_rehash "${C_REHASH}" )

install( DIRECTORY ${PROJECT_BINARY_DIR}/include/openssl DESTINATION include )

install( FILES
  ${PROJECT_BINARY_DIR}/c_rehash
  FAQ LICENSE README README.ENGINE
  DESTINATION share/openssl )

install( DIRECTORY doc DESTINATION share )

# Generate the package target
set( CPACK_GENERATOR ZIP TGZ )
set( CPACK_PACKAGE_NAME "openssl-cmake" )
set( CPACK_PACKAGE_VERSION_MAJOR ${VERSION_MAJOR} )
set( CPACK_PACKAGE_VERSION_MINOR ${VERSION_MINOR} )
set( CPACK_PACKAGE_VERSION_PATCH ${VERSION_PATCH} )

include( CPack )

