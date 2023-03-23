include(FindProtobuf)
find_package(Protobuf REQUIRED)

find_package(gRPC QUIET)
if(gRPC_FOUND)
    set(GRPC_LIBRARIES gRPC::gpr gRPC::grpc gRPC::grpc++)
    #if cross-compiling or nativesdk, find host plugin
    if(CMAKE_CROSSCOMPILING)
        find_program(GRPC_CPP_PLUGIN grpc_cpp_plugin)
    else()
        get_target_property(GRPC_CPP_PLUGIN gRPC::grpc_cpp_plugin LOCATION)
    endif()
    get_target_property(GRPC_INCLUDE_DIR gRPC::grpc INTERFACE_INCLUDE_DIRECTORIES)
else()
    include(${CMAKE_CURRENT_SOURCE_DIR}/cmake/FindGRPC.cmake)
    set(GRPC_LIBRARIES ${GPR_LIBRARY} ${GRPC_LIBRARY} ${GRPC_GRPC++_LIBRARY})
endif()

message(STATUS "grpc cpp plugin: ${GRPC_CPP_PLUGIN}")
set(grpc_cpp_plugin_location ${GRPC_CPP_PLUGIN})
message(STATUS "grpc cpp plugin: ${grpc_cpp_plugin_location}")


find_package(Threads)

#
# Protobuf/Grpc source files
#
set(PROTO_FILES
    myproto/address.proto
    myproto/addressbook.proto
)

#
# Add Library target with protobuf sources
#
add_library(myproto ${PROTO_FILES})
target_link_libraries(myproto
    PUBLIC
        protobuf::libprotobuf
        gRPC::grpc
        gRPC::grpc++
)
target_include_directories(myproto PUBLIC ${CMAKE_CURRENT_BINARY_DIR})

#
# Compile protobuf and grpc files in myproto target to cpp
#
protobuf_generate(TARGET myproto LANGUAGE cpp)
protobuf_generate(TARGET myproto LANGUAGE grpc GENERATE_EXTENSIONS .grpc.pb.h .grpc.pb.cc PLUGIN "protoc-gen-grpc=${grpc_cpp_plugin_location}")
