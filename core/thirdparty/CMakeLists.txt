#-------------------------------------------------------------------------------
# Copyright (C) 2019-2020 Zilliz. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance
# with the License. You may obtain a copy of the License at
#
# http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software distributed under the License
# is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express
# or implied. See the License for the specific language governing permissions and limitations under the License.
#-------------------------------------------------------------------------------

string(TOUPPER ${CMAKE_BUILD_TYPE} UPPERCASE_BUILD_TYPE)
set(EP_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_CXX_FLAGS_${UPPERCASE_BUILD_TYPE}}")
set(EP_C_FLAGS "${CMAKE_C_FLAGS} ${CMAKE_C_FLAGS_${UPPERCASE_BUILD_TYPE}}")

if (NOT MSVC)
    # Set -fPIC on all external projects
    set(EP_CXX_FLAGS "${EP_CXX_FLAGS} -fPIC")
    set(EP_C_FLAGS "${EP_C_FLAGS} -fPIC")
endif ()

# CC/CXX environment variables are captured on the first invocation of the
# builder (e.g make or ninja) instead of when CMake is invoked into to build
# directory. This leads to issues if the variables are exported in a subshell
# and the invocation of make/ninja is in distinct subshell without the same # environment (CC/CXX).
if ( CCACHE_FOUND )
    set( EP_C_COMPILER "${CCACHE_FOUND} ${CMAKE_C_COMPILER}" )
    set( EP_CXX_COMPILER "${CCACHE_FOUND} ${CMAKE_CXX_COMPILER}" )
else()
    set( EP_C_COMPILER ${CMAKE_C_COMPILER} )
    set( EP_CXX_COMPILER ${CMAKE_CXX_COMPILER} )
endif()

set(EP_COMMON_TOOLCHAIN 
    -DCMAKE_C_COMPILER=${EP_C_COMPILER}
    -DCMAKE_CXX_COMPILER=${EP_CXX_COMPILER})

if (CMAKE_AR)
    set(EP_COMMON_TOOLCHAIN ${EP_COMMON_TOOLCHAIN} -DCMAKE_AR=${CMAKE_AR})
endif ()

if (CMAKE_RANLIB)
    set(EP_COMMON_TOOLCHAIN ${EP_COMMON_TOOLCHAIN} -DCMAKE_RANLIB=${CMAKE_RANLIB})
endif ()

# External projects are still able to override the following declarations.
# cmake command line will favor the last defined variable when a duplicate is
# encountered. This requires that `EP_COMMON_CMAKE_ARGS` is always the first
# argument.
set(EP_COMMON_CMAKE_ARGS
        ${EP_COMMON_TOOLCHAIN}
        -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
        -DCMAKE_C_FLAGS=${EP_C_FLAGS}
        -DCMAKE_C_FLAGS_${UPPERCASE_BUILD_TYPE}=${EP_C_FLAGS}
        -DCMAKE_CXX_FLAGS=${EP_CXX_FLAGS}
        -DCMAKE_CXX_FLAGS_${UPPERCASE_BUILD_TYPE}=${EP_CXX_FLAGS})

if (NOT KNOWHERE_VERBOSE_THIRDPARTY_BUILD)
    set(EP_LOG_OPTIONS LOG_CONFIGURE 1 LOG_BUILD 1 LOG_INSTALL 1 LOG_DOWNLOAD 1)
else ()
    set(EP_LOG_OPTIONS)
endif ()

set( MAKE_BUILD_ARGS "-j6" )

set( FETCHCONTENT_BASE_DIR ${MILVUS_BINARY_DIR}/3rdparty_download )
# ----------------------------------------------------------------------
# Find pthreads

set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

# ****************************** Thirdparty googletest ***************************************
if (MILVUS_BUILD_TESTS)
    if ( DEFINED ENV{MILVUS_GTEST_URL} )
        set( GTEST_SOURCE_URL "$ENV{MILVUS_GTEST_URL}" )
    else ()
        set( GTEST_SOURCE_URL
            "https://gitee.com/quicksilver/googletest/repository/archive/release-${GTEST_VERSION}.zip" )
    endif()
    add_subdirectory( gtest )
endif()

# ****************************** Thirdparty grpc ***************************************
if (MILVUS_WITH_GRPC)
    if (DEFINED ENV{MILVUS_GRPC_URL})
        set(GRPC_SOURCE_URL "$ENV{MILVUS_GRPC_URL}")
    else ()
        set(GRPC_SOURCE_URL
                "https://github.com/milvus-io/grpc-milvus/archive/${GRPC_VERSION}.zip"
                )
    endif ()
    add_subdirectory( grpc )
endif ()
