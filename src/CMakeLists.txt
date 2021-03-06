set(CMAKE_INCLUDE_CURRENT_DIR ON)
IF(APPLE)
  set(CMAKE_CXX_FLAGS "-std=c++11 -stdlib=libc++")
ELSE()
  set(CMAKE_CXX_FLAGS "-std=gnu++0x")
ENDIF()

set(CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")
include(public-headers)
include(core)
include(async)
include(objc)
include(LibFindMacros)

IF(APPLE)
  set(additional_includes
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/ctemplate/include
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/icu4c/include
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/libetpan/include
  )
  
  set(additional_lib_searchpath
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/ctemplate/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/icu4c/lib
    ${CMAKE_CURRENT_SOURCE_DIR}/../Externals/libetpan/lib
  )
  
  execute_process(COMMAND "${CMAKE_CURRENT_SOURCE_DIR}/../scripts/prepare.sh"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/../scripts")
ENDIF()

# detect ctemplate

find_path(CTEMPLATE_INCLUDE_DIR
  NAMES ctemplate/template.h
  PATHS ${additional_includes}
)
find_library(CTEMPLATE_LIBRARY
  NAMES ctemplate
  PATHS ${additional_lib_searchpath}
)

if(NOT CTEMPLATE_INCLUDE_DIR OR NOT CTEMPLATE_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find ctemplate")
else()
  message(STATUS "Found ctemplate")
endif()

# detect icu4c

find_path(ICU4C_INCLUDE_DIR
  NAMES unicode/utf8.h
  PATHS ${additional_includes}
)
find_library(ICU4C_LIBRARY
  NAMES icuuc
  PATHS ${additional_lib_searchpath}
)

if(NOT ICU4C_INCLUDE_DIR OR NOT ICU4C_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find icu4c")
else()
  message(STATUS "Found icu4c")
endif()

# detect libetpan

find_path(LIBETPAN_INCLUDE_DIR
  NAMES libetpan/libetpan.h
  PATHS ${additional_includes}
)
find_library(LIBETPAN_LIBRARY
  NAMES etpan
  PATHS ${additional_lib_searchpath}
)

if(NOT LIBETPAN_INCLUDE_DIR OR NOT LIBETPAN_LIBRARY)
  message(FATAL_ERROR "ERROR: Could not find libetpan")
else()
  message(STATUS "Found libetpan")
endif()


file(GLOB_RECURSE
  source_files
  *.h
  *.m
  *.c
)

SET_SOURCE_FILES_PROPERTIES(
  ${source_files}
  PROPERTIES LANGUAGE C
)

file(COPY
  ${public_headers}

  DESTINATION
  ${CMAKE_CURRENT_BINARY_DIR}/include/MailCore
)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include
  ${async_includes}
  ${core_includes}
  ${objc_includes}
  ${additional_includes}
  /usr/include/tidy
  /usr/include/libxml2
)

add_library(MailCore
  ${core_files}
  ${async_files}
  ${objc_files}
)
