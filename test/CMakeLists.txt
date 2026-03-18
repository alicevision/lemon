# ---------------------------------------------------------------------------
# test/CMakeLists.txt
# ---------------------------------------------------------------------------

option(LEMON_BUILD_TESTS "Build LEMON test executables." OFF)

if(NOT LEMON_BUILD_TESTS)
  return()
endif()

# ---------------------------------------------------------------------------
# Valgrind
# ---------------------------------------------------------------------------
option(LEMON_TEST_WITH_VALGRIND "Run tests under valgrind." OFF)
set(LEMON_VALGRIND_FLAGS "" CACHE STRING "Extra valgrind flags for tests.")

# ---------------------------------------------------------------------------
# Helper : enregistre un test, avec ou sans valgrind
# ---------------------------------------------------------------------------
function(lemon_add_test target)
  if(LEMON_TEST_WITH_VALGRIND)
    add_test(
      NAME    ${target}
      COMMAND valgrind --error-exitcode=1 ${LEMON_VALGRIND_FLAGS}
              $<TARGET_FILE:${target}>
    )
  else()
    add_test(NAME ${target} COMMAND ${target})
  endif()
  add_dependencies(check ${target})
endfunction()

# ---------------------------------------------------------------------------
# Helper : crée un exécutable de test (EXCLUDE_FROM_ALL hors Maintainer)
# ---------------------------------------------------------------------------
function(lemon_add_test_executable target)
  if(CMAKE_BUILD_TYPE STREQUAL "Maintainer")
    add_executable(${target} ${target}.cc)
  else()
    add_executable(${target} EXCLUDE_FROM_ALL ${target}.cc)
  endif()
  target_link_libraries(${target} PRIVATE LEMON::lemon)
endfunction()

# ---------------------------------------------------------------------------
# Tests standards
# ---------------------------------------------------------------------------
set(LEMON_TESTS
  adaptors_test
  arc_look_up_test
  bellman_ford_test
  bfs_test
  bpgraph_test
  circulation_test
  connectivity_test
  counter_test
  dfs_test
  digraph_test
  dijkstra_test
  dim_test
  edge_set_test
  error_test
  euler_test
  fractional_matching_test
  gomory_hu_test
  graph_copy_test
  graph_test
  graph_utils_test
  hao_orlin_test
  heap_test
  kruskal_test
  lgf_reader_writer_test
  lgf_test
  maps_test
  matching_test
  max_cardinality_search_test
  max_clique_test
  max_flow_test
  min_cost_arborescence_test
  min_cost_flow_test
  min_mean_cycle_test
  nagamochi_ibaraki_test
  path_test
  planarity_test
  radix_sort_test
  random_test
  suurballe_test
  time_measure_test
  tsp_test
  unionfind_test
)

foreach(test IN LISTS LEMON_TESTS)
  lemon_add_test_executable(${test})
  lemon_add_test(${test})
endforeach()

# ---------------------------------------------------------------------------
# LP test (nécessite au moins un solver LP)
# ---------------------------------------------------------------------------
if(LEMON_HAVE_LP)
  lemon_add_test_executable(lp_test)

  # Les solvers sont linkés en PRIVATE : détails d'implémentation du test
  target_link_libraries(lp_test PRIVATE
    $<$<BOOL:${LEMON_HAVE_GLPK}>:${GLPK_LIBRARIES}>
    $<$<BOOL:${LEMON_HAVE_CPLEX}>:${ILOG_LIBRARIES}>
    $<$<BOOL:${LEMON_HAVE_CLP}>:${COIN_CLP_LIBRARIES}>
    $<$<BOOL:${LEMON_HAVE_SOPLEX}>:${SOPLEX_LIBRARIES}>
  )

  lemon_add_test(lp_test)

  # Copie des DLL solver au même endroit que l'exécutable (Windows only)
  if(WIN32)
    if(LEMON_HAVE_GLPK)
      add_custom_command(TARGET lp_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
          "${GLPK_BIN_DIR}/glpk.dll"
          "${GLPK_BIN_DIR}/libltdl3.dll"
          "${GLPK_BIN_DIR}/zlib1.dll"
          "$<TARGET_FILE_DIR:lp_test>"
      )
    endif()
    if(LEMON_HAVE_CPLEX)
      add_custom_command(TARGET lp_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
          "${ILOG_CPLEX_DLL}"
          "$<TARGET_FILE_DIR:lp_test>"
      )
    endif()
  endif()
endif()

# ---------------------------------------------------------------------------
# MIP test (nécessite au moins un solver MIP)
# ---------------------------------------------------------------------------
if(LEMON_HAVE_MIP)
  lemon_add_test_executable(mip_test)

  target_link_libraries(mip_test PRIVATE
    $<$<BOOL:${LEMON_HAVE_GLPK}>:${GLPK_LIBRARIES}>
    $<$<BOOL:${LEMON_HAVE_CPLEX}>:${ILOG_LIBRARIES}>
    $<$<BOOL:${LEMON_HAVE_CBC}>:${COIN_CBC_LIBRARIES}>
  )

  lemon_add_test(mip_test)

  if(WIN32)
    if(LEMON_HAVE_GLPK)
      add_custom_command(TARGET mip_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
          "${GLPK_BIN_DIR}/glpk.dll"
          "${GLPK_BIN_DIR}/libltdl3.dll"
          "${GLPK_BIN_DIR}/zlib1.dll"
          "$<TARGET_FILE_DIR:mip_test>"
      )
    endif()
    if(LEMON_HAVE_CPLEX)
      add_custom_command(TARGET mip_test POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy
          "${ILOG_CPLEX_DLL}"
          "$<TARGET_FILE_DIR:mip_test>"
      )
    endif()
  endif()
endif()
