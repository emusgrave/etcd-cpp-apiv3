cmake_minimum_required (VERSION 3.1.3 FATAL_ERROR)
project (etcd-cpp-api)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(etcd-cpp-api_VERSION_MAJOR 0)
set(etcd-cpp-api_VERSION_MINOR 1)

option(BUILD_ETCD_TESTS "Build test cases" OFF)

#find_library(CPPREST_LIB NAMES cpprest)
find_package(cpprestsdk CONFIG REQUIRED)
set(CPPREST_LIB cpprestsdk::cpprest)
find_path(CPPREST_INCLUDE_DIR NAMES cpprest/http_client.h)

find_package(Boost REQUIRED COMPONENTS system thread locale random)
if (APPLE)
    # If we're on OS X check for Homebrew's copy of OpenSSL instead of Apple's
    if (NOT OpenSSL_DIR)
        find_program(HOMEBREW brew)
        if (HOMEBREW STREQUAL "HOMEBREW-NOTFOUND")
            message(WARNING "Homebrew not found: not using Homebrew's OpenSSL")
            if (NOT OPENSSL_ROOT_DIR)
                message(WARNING "Use -DOPENSSL_ROOT_DIR for non-Apple OpenSSL")
            endif()
        else()
            execute_process(COMMAND brew --prefix openssl
                OUTPUT_VARIABLE OPENSSL_ROOT_DIR
                OUTPUT_STRIP_TRAILING_WHITESPACE)
        endif()
    endif()
endif()
find_package(OpenSSL REQUIRED)

message(STATUS "CMAKE_PREFIX_PATH: ${CMAKE_PREFIX_PATH}")
set(protobuf_MODULE_COMPATIBLE TRUE)
find_package(protobuf CONFIG REQUIRED)


message(STATUS "Using protobuf: ${PROTOBUF_VERSION}")
message(STATUS "Using protobuf: ${Protobuf_VERSION}")
message(STATUS "PROTOBUF_INCLUDE_DIR ${PROTOBUF_INCLUDE_DIR}")
message(STATUS "PROTOBUF_INCLUDE_DIRS ${PROTOBUF_INCLUDE_DIRS}")
message(STATUS "PROTOBUF_LIBRARY ${PROTOBUF_LIBRARY}")
message(STATUS "PROTOBUF_LIBRARIES ${PROTOBUF_LIBRARIES}")
message(STATUS "CMAKE_CURRENT_SOURCE_DIR: ${CMAKE_CURRENT_SOURCE_DIR}")

find_package(gRPC CONFIG REQUIRED)
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindGRPC.cmake)
set(GRPC_LIBRARIES gRPC::gpr gRPC::grpc gRPC::grpc++ gRPC::grpc++_alts)
#set(GRPC_LIBRARIES ${GPR_LIBRARY} ${GRPC_LIBRARY} ${GRPC_GRPC++_LIBRARY})


find_package(absl CONFIG REQUIRED)
set(GRPC_LIBRARIES ${GRPC_LIBRARIES} absl::any absl::base)
message(STATUS "GRPC_LIBRARIES: ${GRPC_LIBRARIES}")

# will set `PROTOBUF_GENERATES`, indicates all generated .cc files, and a target `protobuf_generates`.
include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/GenerateProtobuf.cmake)
add_subdirectory(${CMAKE_CURRENT_SOURCE_DIR}/proto)

include_directories(SYSTEM ${CPPREST_INCLUDE_DIR}
                           ${Boost_INCLUDE_DIR}
                           ${PROTOBUF_INCLUDE_DIRS}
                           ${OPENSSL_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_SOURCE_DIR})
#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)

add_subdirectory(src)

if (BUILD_ETCD_TESTS)
    enable_testing()
    add_subdirectory(tst)
endif ()

install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/etcd/Client.hpp
               ${CMAKE_CURRENT_SOURCE_DIR}/etcd/SyncClient.hpp
               ${CMAKE_CURRENT_SOURCE_DIR}/etcd/Response.hpp
               ${CMAKE_CURRENT_SOURCE_DIR}/etcd/Value.hpp
               ${CMAKE_CURRENT_SOURCE_DIR}/etcd/Watcher.hpp
               DESTINATION include/etcd)
install (FILES ${CMAKE_CURRENT_SOURCE_DIR}/etcd/v3/Transaction.hpp
               DESTINATION include/etcd/v3)
install (FILES ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/auth.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/kv.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/rpc.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/rpc.grpc.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/v3lock.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/v3lock.grpc.pb.h
               DESTINATION include/etcd/proto)
install (FILES ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/gogoproto/gogo.pb.h
               DESTINATION include/etcd/proto/gogoproto)
install (FILES ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/google/api/annotations.pb.h
               ${CMAKE_CURRENT_BINARY_DIR}/proto/gen/proto/google/api/http.pb.h
               DESTINATION include/etcd/proto/google/api)
