# Lua
CMake based build of Lua 5.5.0
# Usage
Inside of your project's CMakeLists.txt
```cmake
add_subdirectory(lua)
...
target_link_libraries(<YOURTARGET> Lua::Library)
```