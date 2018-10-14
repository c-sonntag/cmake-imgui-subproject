#
# ImGui project
# CMake ImGui library components maker

# --- PROJECT ---
project(imgui)
cmake_minimum_required(VERSION 3.6)
#
set(IMGUI_VERSION_MAJOR 1)
set(IMGUI_VERSION_MINOR 0)
set(PROJECT_IMGUI_STATIC "${PROJECT_NAME}_static")
set(PROJECT_IMGUI_SHARED "${PROJECT_NAME}_shared")

#
##
#

# --- PROFILE ENVIRONNEMENT ---
if(CMAKE_COMPILER_IS_GNUCXX)            # (GNU COMMAND)
  add_definitions("-Wall" "-pedantic")
  add_definitions("-Wno-unused-function" "-Wno-unknown-pragmas")
  if(NOT CMAKE_BUILD_TYPE STREQUAL "Debug")   ## (not debug)
    add_compile_options("-fno-builtin")         ## Break __buildin function (so slowest)
    add_compile_options("-O3")                  ### Optimize
    set(CMAKE_EXE_LINKER_FLAGS "-s")            ### Strip binary (no work on debug)
  endif()
endif()

set(CMAKE_CXX_STANDARD 14)

#
##
#

# --- BUILD OPTIONS : STATIC/SHARED ---
SET(IMGUI_SHARED ON CACHE BOOL "Build shared IMGUI library")
SET(IMGUI_STATIC ON CACHE BOOL "Build static IMGUI library")

# --- CHECK BUILD OPTIONS ---
if( (NOT ${IMGUI_SHARED}) AND (NOT ${IMGUI_STATIC}) )
  message(FATAL_ERROR, "\n\n   !! Require one of SHARED or STATIC for IMGUI !! \n\n")
  set(IMGUI_FOUND OFF)
  return()
endif()

#
##
#

set(IMGUI_DIR
  "${CMAKE_CURRENT_SOURCE_DIR}/imgui"
)

set(IMGUI_ADDON_DIRS
  "${CMAKE_CURRENT_SOURCE_DIR}/imgui-flix01/addons"
)

set(IMGUI_IMPL_DIRS
  "${CMAKE_CURRENT_SOURCE_DIR}/imgui/examples"
  "${CMAKE_CURRENT_SOURCE_DIR}/impl-repo/impl-qt"
)


#
##
#

set(IMGUI_INCLUDE_DIRS ${IMGUI_DIR})

#
## Components by "subdirectory" parent
#

macro(imgui_component_finder component_type basename paternnames_varname paths_varname)

  set(paternnames ${${paternnames_varname}})
  set(paths       ${${paths_varname}})

  find_file("component_file_found_${component_type}_${basename}"
    NAMES ${paternnames}
    PATHS ${paths}
  )
  set(component_file_found "${component_file_found_${component_type}_${basename}}")

  if(NOT EXISTS "${component_file_found}")
    message(WARNING " - ${component_type} \"${basename}\" not found into \"${paths}\" with this patern \"${paternnames}\"")
  else()
    get_filename_component(component_filename ${component_file_found} NAME_WE)
    get_filename_component(component_directory ${component_file_found} DIRECTORY)

    file(GLOB component_sources
      "${component_directory}/${component_filename}*.c"
      "${component_directory}/${component_filename}*.cpp"
    )

    set(IMGUI_IMPL_SOURCES ${IMGUI_IMPL_SOURCES} "${component_sources}")
    set(IMGUI_IMPL_INCLUDE_DIRS ${IMGUI_IMPL_INCLUDE_DIRS} "${component_directory}")

  endif()

endmacro()

#
##
#

if(IMGUI_COMPONENTS)

  message("IMGUI Component(s) by subdirectory : ${IMGUI_COMPONENTS}")

  foreach(component_name ${IMGUI_COMPONENTS})

    if(component_name MATCHES "^addon_")

      string(REPLACE "addon_" "" addon_name "${component_name}")
      set(addons_possible_names
        "imgui${addon_name}.h"
        "imgui${addon_name}/imgui${addon_name}.h"
      )

      imgui_component_finder("Addon" "${addon_name}" addons_possible_names IMGUI_ADDON_DIRS)

    else()

      #
      set(impl_name ${component_name})
      set(impls_possible_names
        "imgui_impl_${impl_name}.h"
        "${impl_name}.h"
      )

      imgui_component_finder("Impl" "${impl_name}" impls_possible_names IMGUI_IMPL_DIRS)


    endif()
  endforeach()


  #
  ##
  if(IMGUI_ADDON_SOURCES)
    set(IMGUI_ADDON_INCLUDE_DIRS ${IMGUI_INTERNAL_INCLUDE_DIR} ${IMGUI_ADDON_INCLUDE_DIRS})
  endif()

  #
  ##
  set(IMGUI_COMPONENTS_SOURCES ${IMGUI_IMPL_SOURCES} ${IMGUI_ADDON_SOURCES})
  set(IMGUI_COMPONENTS_INCLUDE_DIRS ${IMGUI_IMPL_INCLUDE_DIRS} ${IMGUI_ADDON_INCLUDE_DIRS})

  #
  ##

  set(IMGUI_INCLUDE_DIRS ${IMGUI_INCLUDE_DIRS} ${IMGUI_COMPONENTS_INCLUDE_DIRS})


  ## #
  ## ##
  ## if(IMGUI_COMPONENTS_SOURCES)
  ##
  ##   message(STATUS "IMGUI_COMPONENTS_SOURCES : ${IMGUI_COMPONENTS_SOURCES}")
  ##
  ##   #if(NOT TARGET ${IMGUI_COMPONENTS_LIBRARIES} )
  ##   add_library(${IMGUI_COMPONENTS_LIBRARIES} SHARED ${IMGUI_COMPONENTS_SOURCES})
  ##   #endif()
  ##
  ##   #
  ##   target_include_directories(${IMGUI_COMPONENTS_LIBRARIES} PUBLIC ${IMGUI_INCLUDE_DIRS})
  ##   target_link_libraries(${IMGUI_COMPONENTS_LIBRARIES} PUBLIC ${PROJECT_SUBPROJECT_LINKING})
  ##
  ##   message(STATUS "IMGUI_COMPONENTS_LIBRARIES : ${IMGUI_COMPONENTS_LIBRARIES}")
  ##   message(STATUS "PROJECT_SUBPROJECT_LINKING : ${PROJECT_SUBPROJECT_LINKING}")
  ##
  ##   ## #
  ##   ## target_include_directories(${IMGUI_COMPONENTS_LIBRARIES} PUBLIC ${IMGUI_COMPONENTS_LIBRARIES_DEPENDANCES_INCLUDE_DIRS})
  ##   ## target_link_libraries(${IMGUI_COMPONENTS_LIBRARIES} PUBLIC ${IMGUI_COMPONENTS_LIBRARIES_DEPENDANCES_LIBRARIES})
  ##
  ## endif()

endif()

#
## LIB
#

set(LIB_NAME "imgui")

#
file(GLOB LIB_SOURCES_FILES "${IMGUI_DIR}/imgui*.cpp")
file(GLOB LIB_ALL_HEADERS_FILES "${IMGUI_DIR}/*.h")

#
set(LIB_HEADERS_FILE_NAMES "imgui.h" "imconfig.h")
set(LIB_HEADERS_FILES)
set(LIB_INTERNAL_HEADERS_FILES)
foreach(header_filepath ${LIB_ALL_HEADERS_FILES})
  get_filename_component(header_filename ${header_filepath} NAME)
  if(header_filename IN_LIST LIB_HEADERS_FILE_NAMES)
    set(LIB_HEADERS_FILES ${LIB_HEADERS_FILES} "${header_filepath}")
  else()
    set(LIB_INTERNAL_HEADERS_FILES ${LIB_INTERNAL_HEADERS_FILES} "${header_filepath}")
  endif()
endforeach()

#
file(GLOB ADDON_INTERNAL_HEADERS_FILES "${IMGUI_ADDONS_DIRS}/*.h" "${IMGUI_ADDONS_DIRS}/*.inl")
set(LIB_INTERNAL_HEADERS_FILES ${LIB_INTERNAL_HEADERS_FILES} ${ADDON_INTERNAL_HEADERS_FILES})

#
##
#

message(STATUS "IMGUI_COMPONENTS_SOURCES : ${IMGUI_COMPONENTS_SOURCES}")

# --- BUILDS LIB TARGET STATIC ---
if(${IMGUI_STATIC})
  add_library(${PROJECT_IMGUI_STATIC} STATIC ${LIB_SOURCES_FILES} ${IMGUI_COMPONENTS_SOURCES})
  set_target_properties(${PROJECT_IMGUI_STATIC} PROPERTIES OUTPUT_NAME ${LIB_NAME})
  target_include_directories(${PROJECT_IMGUI_STATIC} PUBLIC ${IMGUI_INCLUDE_DIRS})
endif()

# --- BUILDS LIB TARGET SHARED ---
if(${IMGUI_SHARED})
  add_library(${PROJECT_IMGUI_SHARED} SHARED ${LIB_SOURCES_FILES} ${IMGUI_COMPONENTS_SOURCES})
  set_target_properties(${PROJECT_IMGUI_SHARED} PROPERTIES OUTPUT_NAME ${LIB_NAME})
  target_include_directories(${PROJECT_IMGUI_SHARED} PUBLIC ${IMGUI_INCLUDE_DIRS})
endif()

#
## FOR PARRENT DIRECTORY
#

get_directory_property(hasParent PARENT_DIRECTORY)
if(hasParent)
    set(IMGUI_INCLUDE_DIRS ${IMGUI_INCLUDE_DIRS} PARENT_SCOPE)
    message(STATUS "IMGUI_INCLUDE_DIRS : ${IMGUI_INCLUDE_DIRS}")
endif()

