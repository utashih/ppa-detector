
configure_file("${CMAKE_CURRENT_SOURCE_DIR}/config.h.cmake" 
               "${CMAKE_CURRENT_BINARY_DIR}/config.h" @ONLY
)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

add_executable(ppa-detector
  Compiler.cpp
  main.cpp
  AllFilesLoader.cpp
)

llvm_map_components_to_libnames(REQ_LLVM_LIBRARIES ${LLVM_TARGETS_TO_BUILD}
        asmparser core linker bitreader bitwriter irreader ipo scalaropts
        analysis target mc support
)

target_link_libraries(ppa-detector ppa-comparator ppa-inst ${REQ_LLVM_LIBRARIES})

# Platform dependencies.
if( WIN32 )
  message(WARNING "Compatibility with Windows is not tested.")
  find_library(SHLWAPI_LIBRARY shlwapi)
  target_link_libraries(ppa-detector
    ${SHLWAPI_LIBRARY}
  )
else()
  find_package(Threads REQUIRED)
  find_package(Curses REQUIRED)
  target_link_libraries(ppa-detector
    ${CMAKE_THREAD_LIBS_INIT}
    ${CMAKE_DL_LIBS}
    ${CURSES_LIBRARIES}
  )
endif()

set_target_properties(ppa-detector
                      PROPERTIES
                      LINKER_LANGUAGE CXX
                      PREFIX ""
)

install(TARGETS ppa-detector
  RUNTIME DESTINATION bin
)