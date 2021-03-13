# Lua
CMake based build of Lua 5.4.2  
![Build Linux](https://github.com/walterschell/Lua/workflows/Build%20Linux/badge.svg)  
![Build Windows](https://github.com/walterschell/Lua/workflows/Build%20Windows/badge.svg)  
![Build OSX](https://github.com/walterschell/Lua/workflows/Build%20OSX/badge.svg)  
![Build Emscripten](https://github.com/walterschell/Lua/workflows/Build%20Emscripten/badge.svg)  
# Usage
Inside of your project's CMakeLists.txt
```cmake
add_subdirectory(lua)
...
target_link_libraries(<YOURTARGET> lua_static)
```
