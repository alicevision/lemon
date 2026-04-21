# ---------------------------------------------------------------------------
# demo/CMakeLists.txt
# ---------------------------------------------------------------------------

option(LEMON_BUILD_DEMOS "Build LEMON demo executables." OFF)

if(NOT LEMON_BUILD_DEMOS)
  return()
endif()

set(LEMON_DEMOS
  arg_parser_demo
  graph_to_eps_demo
  lgf_demo
)

foreach(demo IN LISTS LEMON_DEMOS)
  add_executable(${demo} ${demo}.cc)
  target_link_libraries(${demo} PRIVATE LEMON::lemon)
endforeach()
