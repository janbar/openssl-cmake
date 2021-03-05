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

if( BUILD_SHARED_LIBS )
  add_definitions( -DOPENSSL_BUILD_SHLIBSSL )
endif()

set( LIBSRC
  bio_ssl.c d1_lib.c d1_msg.c d1_srtp.c methods.c packet.c pqueue.c s3_cbc.c s3_enc.c s3_lib.c
  s3_msg.c ssl_asn1.c ssl_cert.c ssl_ciph.c ssl_conf.c ssl_err.c ssl_init.c ssl_lib.c
  ssl_mcnf.c ssl_rsa.c ssl_sess.c ssl_stat.c ssl_txt.c ssl_utst.c t1_enc.c t1_lib.c
  t1_trce.c tls13_enc.c tls_srp.c
  record/dtls1_bitmap.c record/rec_layer_d1.c record/rec_layer_s3.c record/ssl3_buffer.c
  record/ssl3_record.c record/ssl3_record_tls13.c
  statem/extensions.c statem/extensions_clnt.c statem/extensions_cust.c
  statem/extensions_srvr.c statem/statem.c statem/statem_clnt.c statem/statem_dtls.c
  statem/statem_lib.c statem/statem_srvr.c
  )

include_directories( BEFORE SYSTEM
  ${PROJECT_SOURCE_DIR}/ # e_os.h
)

add_library( ssl ${LIBSRC} )

target_include_directories( ssl PUBLIC ${PROJECT_BINARY_DIR}/include )

target_link_libraries( ssl crypto )

set_target_properties( ssl PROPERTIES
  VERSION "${LIB_VERSION}" SOVERSION "${LIB_SOVERSION}" )

if( ANDROID )
  set( OPENSSL_LIBRARY_NAME  "ssl_1_1" )
  set_target_properties( ssl PROPERTIES OUTPUT_NAME ${OPENSSL_LIBRARY_NAME} )
endif()

if( MSVC )
  if( CMAKE_SIZEOF_VOID_P EQUAL 8 )
    set( OPENSSL_LIBRARY_NAME "ssl-1_1-x64" )
  elseif( CMAKE_SIZEOF_VOID_P EQUAL 4 )
    set( OPENSSL_LIBRARY_NAME "ssl-1_1" )
  endif()
  set_target_properties( ssl PROPERTIES OUTPUT_NAME ${OPENSSL_LIBRARY_NAME} )
  set_target_properties( ssl
    PROPERTIES
    COMPILE_PDB_NAME_DEBUG ${OPENSSL_LIBRARY_NAME}${CMAKE_DEBUG_POSTFIX}
    COMPILE_PDB_NAME_RELEASE ${OPENSSL_LIBRARY_NAME}
    COMPILE_PDB_NAME_MINSIZEREL ${OPENSSL_LIBRARY_NAME}
    COMPILE_PDB_NAME_RELWITHDEBINFO ${OPENSSL_LIBRARY_NAME} )
  if( OPENSSL_BINARYDIR_POSTFIX )
    set( OPENSSL_BINARY_DIR  "bin/${OPENSSL_BINARYDIR_POSTFIX}" )
  else()
    set( OPENSSL_BINARY_DIR  "bin" )
  endif()
  if( OPENSSL_LIBRARYDIR_POSTFIX )
    set( OPENSSL_LIBRARY_DIR "lib/${OPENSSL_LIBRARYDIR_POSTFIX}" )
    set( OPENSSL_ARCHIVE_DIR "lib/${OPENSSL_LIBRARYDIR_POSTFIX}" )
  else()
    set( OPENSSL_LIBRARY_DIR "lib" )
    set( OPENSSL_ARCHIVE_DIR "lib" )
  endif()
else()
  set( OPENSSL_BINARY_DIR  "bin" )
  set( OPENSSL_LIBRARY_DIR "lib" )
  set( OPENSSL_ARCHIVE_DIR "lib" )
endif()

install( TARGETS ssl
    RUNTIME DESTINATION ${OPENSSL_BINARY_DIR}
    LIBRARY DESTINATION ${OPENSSL_LIBRARY_DIR}
    ARCHIVE DESTINATION ${OPENSSL_ARCHIVE_DIR} )

if( MSVC )
  if ( BUILD_SHARED_LIBS )
    install(FILES $<TARGET_PDB_FILE:ssl>
        DESTINATION ${OPENSSL_BINARY_DIR} OPTIONAL)
  else()
    install( DIRECTORY $<TARGET_FILE_DIR:ssl>/
        DESTINATION ${OPENSSL_LIBRARY_DIR}
        FILES_MATCHING PATTERN "*.pdb" )
  endif()
endif()
