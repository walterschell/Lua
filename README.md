# Lua
CMake based build of Lua 5.4.0
# Usage
Inside of CMakeLists.txt
```cmake
add_subdirectory(lua)
...
target_link_libraries(<YOURTARGET> lua_static)
```
