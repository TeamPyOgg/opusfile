# Opusfile CMakeLists.txt
cmake_minimum_required(VERSION 3.15)
project(Opusfile LANGUAGES C)

# Set options to disable documentation and examples
option(OP_ENABLE_HTTP "Enable HTTP support" OFF)
option(OP_ENABLE_EXAMPLES "Enable examples" OFF)

# Library sources
set(OPUSFILE_SOURCES
    src/info.c
    src/internal.c
    src/opusfile.c
    src/stream.c
)

# Create the opusfile library
add_library(opusfile STATIC ${OPUSFILE_SOURCES})
target_include_directories(opusfile PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)

# Link with required libraries
target_link_libraries(opusfile PRIVATE opus ogg)

# Optional HTTP support for libopusurl
if(OP_ENABLE_HTTP)
    set(OPUSURL_SOURCES src/http.c)
    add_library(opusurl STATIC ${OPUSURL_SOURCES} src/internal.c)
    target_link_libraries(opusurl PRIVATE opusfile -lws2_32 -lcrypt32)
    target_include_directories(opusurl PRIVATE ${CMAKE_CURRENT_SOURCE_DIR}/include)
endif()

# Optional examples
if(OP_ENABLE_EXAMPLES)
    add_executable(opusfile_example examples/opusfile_example.c)
    target_link_libraries(opusfile_example PRIVATE opusfile opusurl)

    add_executable(seeking_example examples/seeking_example.c)
    target_link_libraries(seeking_example PRIVATE opusfile opusurl)

    # Include additional headers for Windows UTF-8 support if on Windows
    if(WIN32)
        target_sources(opusfile_example PRIVATE examples/win32utf8.c)
        target_sources(seeking_example PRIVATE examples/win32utf8.c)
    endif()
endif()
