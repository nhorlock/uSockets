cmake_minimum_required(VERSION 3.2)
project(uSockets C CXX)

option(INSTALL_HEADERS "Install header files" ON)

# To enable this feature, OpenSSL 1.1+ is required and is currently disabled due to issue 4267
if (CMAKE_USE_OPENSSL)
    find_package(OpenSSL REQUIRED)
    set(USE_OPENSSL "-DUSE_OPENSSL -DLIBUS_USE_OPENSSL")
    #set(OPENSSL_LIB "OpenSSL::SSL OpenSSL::Crypto")
    list(APPEND CMAKE_REQUIRED_INCLUDES ${OPENSSL_INCLUDE_DIR})
else()
    set(NOT_USE_OPENSSL "-DLIBUS_NO_SSL")
endif()

if(${LIBUS_USE_LIBUV})
    find_package(unofficial-libuv CONFIG REQUIRED)
    find_path(LIBUV_INCLUDE_DIR uv.h)
    find_library(LIBUV_LIBRARY NAMES libuv)
endif()

include_directories(APPEND "${CMAKE_CURRENT_LIST_DIR}/src")
set(US_INCLUDES ${CMAKE_CURRENT_LIST_DIR}/src)
file(GLOB US_SOURCES src/*.c src/eventing/*.c src/interfaces/*.c src/internal/*.c)

add_library(uSockets ${US_SOURCES})

if (${LIBUS_USE_LIBUV})
    message("Using LIBUV")
  target_compile_definitions(uSockets PRIVATE -DLIBUS_USE_LIBUV)
  set(UV_INC ${LIBUV_INCLUDE_DIR})
  set(UV_LIB ${LIBUV_LIBRARY})
else()
    message("Not using LIBUV")
    set(UV_INC "")
    set(UV_LIB "")
endif()

target_compile_definitions(uSockets PRIVATE ${NOT_USE_OPENSSL} ${USE_OPENSSL})
target_include_directories(uSockets PUBLIC ${US_INCLUDES} ${OPENSSL_INCLUDE_DIR} ${UV_INC} )
target_link_libraries(uSockets PUBLIC ${OPENSSL_LIBRARIES} ${UV_LIB})

install(TARGETS uSockets
    RUNTIME DESTINATION bin
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
)

if(INSTALL_HEADERS)
    file(GLOB HEADERS src/*.h)
    install(FILES ${HEADERS} DESTINATION include)
    file(GLOB HEADERS src/interfaces/*.h)
    install(FILES ${HEADERS} DESTINATION include/interfaces)
endif()

#add_subdirectory(examples)
