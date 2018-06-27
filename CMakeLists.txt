cmake_minimum_required(VERSION 2.8)
project(replicator)

set(CMAKE_VERBOSE_MAKEFILE on)
set(CMAKE_SKIP_INSTALL_ALL_DEPENDENCY true)
string(TOLOWER ${CMAKE_BUILD_TYPE} CMAKE_BUILD_TYPE_LOWER)

set(CMAKE_MODULE_PATH  "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_MODULE_PATH})
set(CMAKE_INCLUDE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake" ${CMAKE_INCLUDE_PATH})

set(COPTS_FLAGS_RELEASE     "-pipe -fomit-frame-pointer -ffunction-sections -fdata-sections")
set(CMAKE_C_FLAGS_RELEASE   "${CMAKE_C_FLAGS_RELEASE}   ${COPTS_FLAGS_RELEASE}")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} ${COPTS_FLAGS_RELEASE}")

set(COPTS_FLAGS_DEBUG     "-O0 -g3 -ggdb3")
SET(CMAKE_C_FLAGS_DEBUG   "${CMAKE_C_FLAGS_DEBUG}   ${COPTS_FLAGS_DEBUG}")
SET(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${COPTS_FLAGS_DEBUG}")

find_package(MySQL)

set(REPLICATOR_ROOT "${CMAKE_SOURCE_DIR}")
set(REPLICATOR_SRC
	"${REPLICATOR_ROOT}/dbreader.cpp"
	"${REPLICATOR_ROOT}/main.cpp"
	"${REPLICATOR_ROOT}/tpwriter.cpp"
)

set(YAML_CPP_BUILD_TOOLS OFF CACHE BOOL "" FORCE)
set(YAML_CPP_BUILD_CONTRIB OFF CACHE BOOL "" FORCE)

include_directories(
	${REPLICATOR_ROOT}
	"${REPLICATOR_ROOT}/lib/libslave"
	"${REPLICATOR_ROOT}/lib/yaml-cpp/include"
	"${REPLICATOR_ROOT}/lib/tarantool-c/include"
	"${REPLICATOR_ROOT}/lib/tarantool-c/third_party/msgpuck"
)

set(TARANTOOL_C_EMBEDDED ON)
add_subdirectory(lib/libslave    EXCLUDE_FROM_ALL)
add_subdirectory(lib/yaml-cpp    EXCLUDE_FROM_ALL)
add_subdirectory(lib/tarantool-c EXCLUDE_FROM_ALL)

find_library(LPTHREAD pthread)

add_executable(rp ${REPLICATOR_SRC})
set_target_properties(rp PROPERTIES
       COMPILE_FLAGS "-std=c++11"
       OUTPUT_NAME replicatord)
target_link_libraries(rp rt dl tnt slave yaml-cpp ${LPTHREAD} z)

if ("${CMAKE_BUILD_TYPE_LOWER}" STREQUAL "release")
    set_target_properties(rp PROPERTIES
           LINK_FLAGS "-Wl,-s -Wl,--gc-sections")
endif("${CMAKE_BUILD_TYPE_LOWER}" STREQUAL "release")

install(TARGETS rp RUNTIME DESTINATION sbin)
install(FILES replicatord.yml DESTINATION etc)
