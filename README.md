# cmake-imgui-subproject

## How to clone
``` git clone --recursive https://github.com/c-sonntag/cmake-imgui-subproject ```

## Example how to use in CMakeList.txt file :
 
```
# --- LIB : IMGUI ---
add_definitions("-DIMGUI_IMPL_OPENGL_LOADER_GLEW=1")
add_definitions("-DIMGUI_IMPL_API=")
if(NOT TARGET imgui_shared OR NOT TARGET imgui_components)
  set(IMGUI_SHARED ON)
  set(IMGUI_STATIC OFF)
  set(IMGUI_COMPONENTS opengl3 glfw addon_filesystem)
  add_subdirectory(${IMGUI_BASE_DIR} "${CMAKE_BINARY_DIR}/imgui")
endif()
include_directories(${IMGUI_INCLUDE_DIRS})
link_libraries(imgui_shared)
set_target_properties(imgui_shared PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}" 
LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}")
```


