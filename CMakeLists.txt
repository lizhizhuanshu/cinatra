cmake_minimum_required(VERSION 3.15)
project(cinatra)

# ── Dependencies provided by parent project ──────────────────────────────────
# Parent must set asio_SOURCE_DIR and async_simple_SOURCE_DIR before
# add_subdirectory(this dir), or set CINATRA_3RD_DIR to a directory
# containing asio/ and async_simple/ subdirectories.

if(NOT asio_SOURCE_DIR)
    if(CINATRA_3RD_DIR)
        set(asio_SOURCE_DIR "${CINATRA_3RD_DIR}/asio" CACHE PATH "asio source dir")
    else()
        message(FATAL_ERROR "cinatra requires asio_SOURCE_DIR or CINATRA_3RD_DIR to be set")
    endif()
endif()

if(NOT async_simple_SOURCE_DIR)
    if(CINATRA_3RD_DIR)
        set(async_simple_SOURCE_DIR "${CINATRA_3RD_DIR}/async_simple" CACHE PATH "async_simple source dir")
    else()
        message(FATAL_ERROR "cinatra requires async_simple_SOURCE_DIR or CINATRA_3RD_DIR to be set")
    endif()
endif()

# Build async_simple if not already provided by parent
if(NOT TARGET async_simple)
    set(ASYNC_SIMPLE_BUILD_MODULES OFF CACHE BOOL "" FORCE)
    set(ASYNC_SIMPLE_ENABLE_TESTS OFF CACHE BOOL "" FORCE)
    set(ASYNC_SIMPLE_BUILD_DEMO_EXAMPLE OFF CACHE BOOL "" FORCE)
    add_subdirectory(${async_simple_SOURCE_DIR})
endif()

function(cinatra_use_third_party_headers target_name)
    if (TARGET ${target_name})
        target_include_directories(${target_name} SYSTEM PRIVATE
            ${asio_SOURCE_DIR}/asio/include
            ${async_simple_SOURCE_DIR}
        )
    endif ()
endfunction()

include_directories(include)

#the thread library of the system.
find_package(Threads REQUIRED)

include(cmake/find_openssl.cmake)
include(cmake/build.cmake)
include(cmake/develop.cmake)

if (BUILD_UNIT_TESTS)
    add_subdirectory(${cinatra_SOURCE_DIR}/tests)
endif ()
if (BUILD_EXAMPLES)
    add_subdirectory(${cinatra_SOURCE_DIR}/example)
endif ()
if (BUILD_PRESS_TOOL)
    add_subdirectory(${cinatra_SOURCE_DIR}/press_tool)
endif ()
