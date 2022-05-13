cmake_minimum_required(VERSION 3.1)
project(lua LANGUAGES C VERSION 5.4.4)
enable_testing()

option(LUA_SUPPORT_DL "Support dynamic loading of compiled modules" OFF)
option(LUA_BUILD_AS_CXX "Build lua as C++" OFF)
option(LUA_ENABLE_SHARED "Build dynamic liblua" ON)
enable_language(CXX)

if(${PROJECT_NAME} STREQUAL ${CMAKE_PROJECT_NAME})
    set(TOP_LEVEL TRUE)
else()
    set(TOP_LEVEL FALSE)
endif()

if(TOP_LEVEL)
    option(LUA_BUILD_BINARY "Build lua binary" ON)
    option(LUA_BUILD_COMPILER "Build luac compiler" ON)
else()
    option(LUA_BUILD_BINARY "Build lua binary" OFF)
    option(LUA_BUILD_COMPILER "Build luac compiler" ON)
endif()


add_subdirectory(lua-5.4.4)

add_test(NAME lua-testsuite COMMAND lua -e "_U=true" all.lua WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/lua-5.4.4-tests)