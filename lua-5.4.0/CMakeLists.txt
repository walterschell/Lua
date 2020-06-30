set(LUA_LIB_SRCS 
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lapi.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lcode.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lctype.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ldebug.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ldo.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ldump.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lfunc.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lgc.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/llex.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lmem.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lobject.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lopcodes.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lparser.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lstate.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lstring.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ltable.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ltm.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lundump.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lvm.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lzio.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lauxlib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lbaselib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lcorolib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ldblib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/liolib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lmathlib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/loadlib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/loslib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lstrlib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/ltablib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/lutf8lib.c
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linit.c
)
set(LUA_LIB_INCLUDE ${CMAKE_CURRENT_SOURCE_DIR}/src)
#TODO: Redo this with find_package
find_library(LIBM m)
if(NOT LIBM)
    message(FATAL_ERROR "libm not found and requred by lua")
endif()
add_library(lua_static STATIC ${LUA_LIB_SRCS})
target_link_libraries(lua_static INTERFACE ${LIBM})
target_include_directories(lua_static PUBLIC ${LUA_LIB_INCLUDE})
set(LUA_DEFINITIONS)
if(NOT EMSCRIPTEN)
    list(APPEND LUA_DEFINITIONS LUA_USE_POSIX)
endif()
target_compile_definitions(lua_static 
    PUBLIC ${LUA_DEFINITIONS}
)
target_compile_options(lua_static
    PRIVATE "-Wall" "-Wextra"
)

if(LUA_BUILD_BINARY)
    add_executable(lua ${CMAKE_CURRENT_SOURCE_DIR}/src/lua.c)
    target_link_libraries(lua PUBLIC lua_static)
    set_target_properties(lua PROPERTIES 
        RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    )
endif()
if(LUA_BUILD_COMPILER)
    add_executable(luac ${CMAKE_CURRENT_SOURCE_DIR}/src/luac.c)
    target_link_libraries(luac PUBLIC lua_static)
    set_target_properties(luac PROPERTIES 
        RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    )
endif()