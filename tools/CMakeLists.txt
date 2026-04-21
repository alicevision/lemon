# ---------------------------------------------------------------------------
# tools/CMakeLists.txt
# ---------------------------------------------------------------------------

option(LEMON_BUILD_TOOLS "Build LEMON tool executables." OFF)

if(NOT LEMON_BUILD_TOOLS)
  return()
endif()

set(LEMON_TOOLS
  lgf-gen
  dimacs-to-lgf
  dimacs-solver
)

foreach(tool IN LISTS LEMON_TOOLS)
  add_executable(${tool} ${tool}.cc)
  target_link_libraries(${tool} PRIVATE LEMON::lemon)
endforeach()

install(
  TARGETS   ${LEMON_TOOLS}
  RUNTIME   DESTINATION "${CMAKE_INSTALL_BINDIR}"
  COMPONENT bin
)

if(UNIX)
  install(
    PROGRAMS  "${CMAKE_CURRENT_SOURCE_DIR}/lemon-0.x-to-1.x.sh"
    DESTINATION "${CMAKE_INSTALL_BINDIR}"
    COMPONENT bin
  )
endif()
