cmake_minimum_required(VERSION 3.15)
project(cinatra)

include(FetchContent)

FetchContent_Declare(
    asio
    GIT_REPOSITORY https://github.com/chriskohlhoff/asio.git
    GIT_TAG asio-1-36-0
)
FetchContent_Declare(
    async_simple
    GIT_REPOSITORY https://github.com/lizhizhuanshu/async_simple.git
    GIT_TAG main
    GIT_SUBMODULES ""
    GIT_SUBMODULES_RECURSE OFF
)

FetchContent_MakeAvailable(asio)

set(ASYNC_SIMPLE_BUILD_MODULES OFF CACHE BOOL "" FORCE)
set(ASYNC_SIMPLE_ENABLE_TESTS OFF CACHE BOOL "" FORCE)
set(ASYNC_SIMPLE_BUILD_DEMO_EXAMPLE OFF CACHE BOOL "" FORCE)
FetchContent_MakeAvailable(async_simple)

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