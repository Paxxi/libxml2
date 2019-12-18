cmake_minimum_required(VERSION 3.2)

project(libxml2 VERSION 2.9.4 LANGUAGES C)

option(BUILD_SHARED_LIBS "Build as shared lib instead of static" ON)
option(WITH_TRIO "Enable TRIO string manipulator" 1)
option(WITH_THREADS:BOOL "Enable thread safety [no|ctls|native|posix" ON)
option(WITH_THREAD_ALLOC:BOOL "Thread specific allocators" ON)
option(WITH_FTP "Enable FTP client" 0)
option(WITH_HTTP "Enable HTTP client" 0)
option(WITH_HTML "Enable HTML processor" 1)
option(WITH_C14N "Enable C14N support" 1)
option(WITH_CATALOG "Enable catalog support" 1)
option(WITH_DOCB "Enable DocBook support" 1)
option(WITH_XPATH "Enable XPath support" 1)
option(WITH_XPTR "Enable XPointer support" 1)
option(WITH_XINCLUDE "Enable XInclude support" 1)
option(WITH_ICONV "Enable iconv support" 1)
option(WITH_ICU "Enable icu support" 0)
option(WITH_ISO8859X "Enable ISO8859X support" 0)
option(WITH_ZLIB "Enable zlib support" 0)
option(WITH_LZMA "Enable lzma support" 0)
option(WITH_DEBUG "Enable XML debugging module" 1)
option(WITH_MEM_DEBUG "Enable memory debugger" 0)
option(WITH_RUN_DEBUG "Enable memory debugger" 0)
option(WITH_SCHEMAS "Enable XML schema support" 1)
option(WITH_SCHEMATRON "Enable schematron support" 1)
option(WITH_REGEXPS "Enable regular expressions" 1)
option(WITH_MODULES "Enable module support" 1)
option(WITH_TREE:BOOL "Enable tree api" 1)
option(WITH_READER "Enable xmlReader api" 1)
option(WITH_WRITER "Enable xmlWriter api" 1)
option(WITH_WALKER "Enable xmlDocWalker api" 1)
option(WITH_PATTERN "Enable xmlPattern api" 1)
option(WITH_PUSH "Enable push api" 1)
option(WITH_VALID "Enable DTD validation support" 1)
option(WITH_SAX1 "Enable SAX1 api" 1)
option(WITH_LEGACY "Enable deprecated api's" 0)
option(WITH_OUTPUT "Enable serialization support" 1)
option(WITH_PYTHON "Enable Python bindings" 0)

set(VERSION 2.9.4)
set(LIBXML_VERSION_NUMBER 20904)

include(CheckIncludeFile)
include(CheckFunctionExists)

CHECK_INCLUDE_FILE(ctype.h HAVE_CTYPE_H)
CHECK_INCLUDE_FILE(dirent.h HAVE_DIRENT_H)
CHECK_INCLUDE_FILE(dlfcn.h HAVEN_DLFCN_H)
CHECK_INCLUDE_FILE(dl.h HAVE_DL_H)
CHECK_INCLUDE_FILE(errno.h HAVE_ERRNO_H)
CHECK_INCLUDE_FILE(fcntl.h HAVE_FCNTL_H)
CHECK_INCLUDE_FILE(ieeefp.h HAVE_IEEEP_H)
CHECK_INCLUDE_FILE(inttypes.h HAVE_INTTYPES_H)
CHECK_INCLUDE_FILE(limits.h HAVE_LIMITS_H)
CHECK_INCLUDE_FILE(malloc.h HAVE_MALLOC_H)
CHECK_INCLUDE_FILE(math.h HAVE_MATH_H)
CHECK_INCLUDE_FILE(memory.h HAVE_MEMORY_H)


if(WIN32)
  set(CMAKE_DEBUG_POSTFIX "d")
endif()

configure_file(
  config.h.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/include/config.h
)
add_library(${PROJECT_NAME}
  xzlib.h
  buf.h
  ${CMAKE_CURRENT_BINARY_DIR}/include/config.h
  elfgcchack.h
  enc.h
  libxml.h
  save.h
  timsort.h
  trio.h
  triodef.h
  trionan.h
  triop.h
  triostr.h
  xzlib.c
  buf.c
  c14n.c
  catalog.c
  chvalid.c
  debugXML.c
  dict.c
  DOCBparser.c
  encoding.c
  entities.c
  error.c
  globals.c
  hash.c
  HTMLparser.c
  HTMLtree.c
  legacy.c
  list.c
  nanoftp.c
  nanohttp.c
  parser.c
  parserInternals.c
  pattern.c
  relaxng.c
  SAX.c
  SAX2.c
  schematron.c
  threads.c
  tree.c
  trio.c
  trionan.c
  triostr.c
  uri.c
  valid.c
  xinclude.c
  xlink.c
  xmlIO.c
  xmlmemory.c
  xmlmodule.c
  xmlreader.c
  xmlregexp.c
  xmlsave.c
  xmlschemas.c
  xmlschemastypes.c
  xmlstring.c
  xmlunicode.c
  xmlwriter.c
  xpath.c
  xpointer.c
)
add_library(${PROJECT_NAME}::${PROJECT_NAME} ALIAS ${PROJECT_NAME})
target_include_directories(
  libxml2 PRIVATE
  $<BUILD_INTERFACE:.;include;include/libxml;${CMAKE_CURRENT_BINARY_DIR}/include>
  INTERFACE
  $<INSTALL_INTERFACE:include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/libxml>
)
target_compile_definitions(libxml2
  PRIVATE
  _CRT_SECURE_NO_WARNINGS
  _CRT_STDC_NO_DEPRECATE)

if(WINDOWS_STORE)
  target_compile_definitions(libxml2
    PRIVATE
      MS_APP
  )
endif()
if(BUILD_SHARED_LIBS AND MSVC)
  target_link_libraries(libxml2 PRIVATE ws2_32.lib)
endif()

if(WITH_ICONV)
  find_package(iconv 1.14 REQUIRED NO_MODULE)
  target_link_libraries(libxml2 PRIVATE iconv::iconv)
endif()

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/libxml2-config-version.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion
)

install(TARGETS libxml2 EXPORT libxml2
  RUNTIME DESTINATION bin
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib)


install(DIRECTORY
  ${CMAKE_CURRENT_SOURCE_DIR}/include/libxml
  DESTINATION include
  FILES_MATCHING PATTERN "*.h")

install(EXPORT libxml2
  FILE
    libxml2.cmake
  NAMESPACE
    libxml2::
  DESTINATION
    lib/cmake/libxml2
)
install(
  FILES
    cmake/libxml2-config.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/libxml2-config-version.cmake
  DESTINATION
    lib/cmake/libxml2
)

export(TARGETS libxml2
  FILE libxml2.cmake
  NAMESPACE libxml2::
)

if(MSVC)
  set_target_properties(libxml2
    PROPERTIES
      COMPILE_PDB_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
      COMPILE_PDB_NAME libxml2
      COMPILE_PDB_NAME_DEBUG libxml2d
  )
  install(FILES
    $<TARGET_PDB_FILE:libxml2>
    DESTINATION lib
  )
endif()