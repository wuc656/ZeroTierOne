cmake_minimum_required(VERSION 3.2)

project(ZeroTierOneJNI)

find_package(Java COMPONENTS Development)
message("JAVA_HOME: $ENV{JAVA_HOME}")

if(WIN32)
set(Java_INCLUDE_DIRS $ENV{JAVA_HOME}/include)
endif()

if(APPLE)
set(Java_INCLUDE_DIRS "/Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.10.sdk/System/Library/Frameworks/JavaVM.framework/Headers")
endif()

message("Java Include Dirs: ${Java_INCLUDE_DIRS}")

if(WIN32)
    add_definitions(-DNOMINMAX)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /EHsc /W3 /MP")
endif()

if(APPLE)
    set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -arch i386 -arch x86_64 -Wall -O3 -flto -fPIE -fvectorize -fstack-protector -mmacosx-version-min=10.7 -Wno-unused-private-field")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${CMAKE_C_FLAGS} -fno-rtti")
endif()

set(src_files
    # ../ext/lz4/lz4.c
    # ../ext/json-parser/json.c
    ../ext/http-parser/http_parser.c
    # ../node/C25519.cpp
    ../node/CertificateOfMembership.cpp
    # ../node/Defaults.cpp
    # ../node/Dictionary.cpp
    ../node/Identity.cpp
    ../node/IncomingPacket.cpp
    ../node/InetAddress.cpp
    ../node/Multicaster.cpp
    ../node/Network.cpp
    ../node/NetworkConfig.cpp
    ../node/Node.cpp
    ../node/OutboundMulticast.cpp
    ../node/Packet.cpp
    ../node/Peer.cpp
    ../node/Poly1305.cpp
    ../node/Salsa20.cpp
    ../node/SelfAwareness.cpp
    ../node/SHA512.cpp
    ../node/Switch.cpp
    ../node/Topology.cpp
    ../node/Utils.cpp
    ../osdep/Http.cpp
    ../osdep/OSUtils.cpp
    jni/com_zerotierone_sdk_Node.cpp
    jni/ZT_jniutils.cpp
    jni/ZT_jnicache.cpp
    )

set(include_dirs
    ${CMAKE_CURRENT_SOURCE_DIR}/../include/
    ${CMAKE_CURRENT_SOURCE_DIR}/../node/
    ${Java_INCLUDE_DIRS})

if(WIN32)
    set(include_dirs
        ${include_dirs}
        ${Java_INCLUDE_DIRS}/win32)
endif()

include_directories(
    ${include_dirs}
    )
file(GLOB ZT_NODE_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/../node/*.cpp)
file(GLOB ZT_OSDEP_SOURCES ${CMAKE_CURRENT_SOURCE_DIR}/../osdep/*.cpp)
set(src_files
    ../ext/http-parser/http_parser.c
    ${ZT_NODE_SOURCES}
    ${ZT_OSDEP_SOURCES}
    jni/com_zerotierone_sdk_Node.cpp
    jni/ZT_jniutils.cpp
    jni/ZT_jnicache.cpp
)
add_library(${PROJECT_NAME} SHARED ${src_files})

if(APPLE)
    set_target_properties(${PROJECT_NAME} PROPERTIES SUFFIX ".jnilib")
endif()

set(link_libs )

if(WIN32)
    set(link_libs
        wsock32
        ws2_32

        )
endif()

target_link_libraries(${PROJECT_NAME} ${link_libs})

include(FetchContent)
FetchContent_Declare(
  nlohmann_json
  GIT_REPOSITORY https://github.com/nlohmann/json.git
  GIT_TAG v3.12.0
)
FetchContent_MakeAvailable(nlohmann_json)

target_link_libraries(ZeroTierOneJNI PRIVATE nlohmann_json::nlohmann_json)

include_directories(
  ${CMAKE_SOURCE_DIR}/ext/prometheus-cpp-lite-1.0/core/include
  ${CMAKE_SOURCE_DIR}/ext/prometheus-cpp-lite-1.0/simpleapi/include
)
include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/../../ext/prometheus-cpp-lite-1.0/core/include
  ${CMAKE_CURRENT_SOURCE_DIR}/../../ext/prometheus-cpp-lite-1.0/simpleapi/include
)

include_directories(
  ${CMAKE_SOURCE_DIR}/../ext/prometheus-cpp-lite-1.0/core/include
  ${CMAKE_SOURCE_DIR}/../ext/prometheus-cpp-lite-1.0/simpleapi/include
)
find_library(ANDROID_LOG_LIB log)
target_link_libraries(${PROJECT_NAME} PRIVATE ${ANDROID_LOG_LIB})





