set(LUA_LIB_SRCS 
    "src/lapi.c"
    "src/lcode.c"
    "src/lctype.c"
    "src/ldebug.c"
    "src/ldo.c"
    "src/ldump.c"
    "src/lfunc.c"
    "src/lgc.c"
    "src/llex.c"
    "src/lmem.c"
    "src/lobject.c"
    "src/lopcodes.c"
    "src/lparser.c"
    "src/lstate.c"
    "src/lstring.c"
    "src/ltable.c"
    "src/ltm.c"
    "src/lundump.c"
    "src/lvm.c"
    "src/lzio.c"
    "src/lauxlib.c"
    "src/lbaselib.c"
    "src/lcorolib.c"
    "src/ldblib.c"
    "src/liolib.c"
    "src/lmathlib.c"
    "src/loadlib.c"
    "src/loslib.c"
    "src/lstrlib.c"
    "src/ltablib.c"
    "src/lutf8lib.c"
    "src/linit.c"
)

set(TARGETS_TO_INSTALL lua_obj)

if(LUA_BUILD_AS_CXX)
	set_source_files_properties(${LUA_LIB_SRCS} "src/lua.c" "src/luac.c" PROPERTIES LANGUAGE CXX )
endif()

add_library(lua_obj OBJECT ${LUA_LIB_SRCS})

if(LUA_ENABLE_SHARED)
    add_library(lua_dynamic SHARED)
    target_link_libraries(lua_dynamic PUBLIC lua_obj)
    set_target_properties(lua_dynamic PROPERTIES
        VERSION "${PACKAGE_VERSION}"
    )
    list(APPEND TARGETS_TO_INSTALL lua_dynamic)
    if(BUILD_SHARED_LIBS)
        set(LUA_EXPORT_LIBRARY lua_dynamic)
        add_library(Lua::Lib ALIAS lua_dynamic)
    elseif(NOT TOP_LEVEL)
        set_target_properties(lua_dynamic PROPERTIES
            EXCLUDE_FROM_ALL ON
        )
    endif()
endif()

if(LUA_ENABLE_STATIC)
    add_library(lua_static STATIC)
    target_link_libraries(lua_static PUBLIC lua_obj)
    set_target_properties(lua_static PROPERTIES
        VERSION "${PACKAGE_VERSION}"
    )
    list(APPEND TARGETS_TO_INSTALL lua_static)
    if(NOT BUILD_SHARED_LIBS)
        set(LUA_EXPORT_LIBRARY lua_static)
        add_library(Lua::Lib ALIAS lua_static)
    elseif(NOT TOP_LEVEL)
        set_target_properties(lua_static PROPERTIES
            EXCLUDE_FROM_ALL ON
        )
    endif()
endif()


target_include_directories(lua_obj PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)
if(UNIX)
	set(LUA_DEFINITIONS)
    
    if(NOT EMSCRIPTEN)
        find_library(LIBM m)
        #TODO: Redo this with find_package
        if(NOT LIBM)
            message(FATAL_ERROR "libm not found and is required by lua")
        endif()
        target_link_libraries(lua_obj INTERFACE m)

        list(APPEND LUA_DEFINITIONS LUA_USE_POSIX)
        if(LUA_SUPPORT_DL)
            target_compile_definitions(lua_obj PRIVATE "LUA_USE_DLOPEN")
            target_link_libraries(lua_obj INTERFACE dl)
        endif()
    endif()

	target_compile_definitions(lua_obj 
		PUBLIC ${LUA_DEFINITIONS}
	)
	target_compile_options(lua_obj
		PRIVATE "-Wall" "-Wextra"
	)
endif()

if (WIN32)
    target_compile_definitions(lua_obj 
		PRIVATE LUA_BUILD_AS_DLL
	)
endif()

if(LUA_BUILD_BINARY)
    include(CheckIncludeFile)
    CHECK_INCLUDE_FILE("readline/readline.h" HAVE_READLINE_READLINE_H)


    add_executable(lua "src/lua.c")
    # Can not use lua_dynamic because some symbols are not exported
    target_link_libraries(lua PRIVATE lua_obj)
    set_target_properties(lua PROPERTIES 
        RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    )
    if (HAVE_READLINE_READLINE_H)
        target_compile_definitions(lua PRIVATE "LUA_USE_READLINE")
        target_link_libraries(lua PUBLIC readline)
    endif()
    list(APPEND TARGETS_TO_INSTALL lua)
endif()

if(LUA_BUILD_COMPILER)
    add_executable(luac "src/luac.c")
    target_link_libraries(luac PRIVATE lua_obj)
    set_target_properties(luac PROPERTIES 
        RUNTIME_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}
    )
    list(APPEND TARGETS_TO_INSTALL luac)
endif()

install(TARGETS ${TARGETS_TO_INSTALL}
        EXPORT LuaTargets
)

install(DIRECTORY include/ TYPE INCLUDE)

include(CMakePackageConfigHelpers)

write_basic_package_version_file(
    LuaConfigVersion.cmake
    VERSION ${PACKAGE_VERSION}
    COMPATIBILITY SameMajorVersion
)

install(EXPORT LuaTargets
        FILE LuaTargets.cmake
        DESTINATION "lib/cmake"
        NAMESPACE Lua::
)

configure_package_config_file(
    LuaConfig.cmake.in
    "${CMAKE_CURRENT_BINARY_DIR}/LuaConfig.cmake"
    INSTALL_DESTINATION "lib/cmake"
)

install(FILES "${CMAKE_CURRENT_BINARY_DIR}/LuaConfig.cmake"
              "${CMAKE_CURRENT_BINARY_DIR}/LuaConfigVersion.cmake"
        DESTINATION "lib/cmake"
)
