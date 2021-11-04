cmake_minimum_required(VERSION 3.9.6)

project(qterrainview)

find_package(Qt5 REQUIRED COMPONENTS Widgets)

find_package(OpenMP)

find_program(GLSLANG_VALIDATOR glslangValidator)

if(GLSLANG_VALIDATOR)
  add_custom_target(qterrainview_shader_validation ALL
    COMMAND ${GLSLANG_VALIDATOR} "render_terrain.vert"
    COMMAND ${GLSLANG_VALIDATOR} "render_terrain.frag"
    WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/private/shaders"
    COMMENT "Validating shaders.")
endif(GLSLANG_VALIDATOR)

add_library(qterrainview
  qterrainview/qterrainview.h
  private/qterrainview.cpp
  private/open_gl_widget.hpp
  private/open_gl_widget.cpp
  private/terrain.h
  private/terrain.cpp
  private/terrain_render_program.h
  private/terrain_render_program.cpp
  private/pbr_texture.h
  private/pbr_texture.cpp
  private/pbr_texture_buffer.h
  private/pbr_texture_buffer.cpp
  private/splat_map_buffer.h
  private/splat_map_buffer.cpp
  private/qterrainsurface.cpp
  private/qterrainsurface_p.h
  private/qterrainsurface_p.cpp
  private/qterrainsurfacebuilder.cpp
  private/qterrainsurfacebuilder_p.h
  private/qterrainsurfacebuilder_p.cpp
  private/elevation_converter.h
  private/elevation_converter.cpp
  private/shaders/shaders.qrc)

if(TARGET OpenMP::OpenMP_CXX)
  target_link_libraries(qterrainview PRIVATE OpenMP::OpenMP_CXX)
endif(TARGET OpenMP::OpenMP_CXX)

if(NOT MSVC)
  target_compile_options(qterrainview PRIVATE -Wall -Wextra -Werror -Wfatal-errors)
endif(NOT MSVC)

target_link_libraries(qterrainview
  PUBLIC
    Qt5::Widgets)

set_target_properties(qterrainview
  PROPERTIES
    AUTOMOC ON
    AUTORCC ON)

target_include_directories(qterrainview
  PUBLIC
    "${CMAKE_CURRENT_SOURCE_DIR}")

add_library(terrable::terrain_view ALIAS qterrainview)

if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)

  add_executable(main
    private/main.cpp)

  target_link_libraries(main terrable::terrain_view)

endif(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
