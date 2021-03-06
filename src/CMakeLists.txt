file(GLOB_RECURSE FCL_SOURCE_CODE ${CMAKE_CURRENT_SOURCE_DIR}/*.cpp)
if(FCL_STATIC_LIBRARY)
  add_library(${PROJECT_NAME} STATIC ${FCL_HEADERS} ${FCL_SOURCE_CODE})
else()
  add_library(${PROJECT_NAME} SHARED ${FCL_HEADERS} ${FCL_SOURCE_CODE})
endif()

# Be sure to pass to the consumer the set of SIMD used in the compilation
target_compile_options(${PROJECT_NAME} PUBLIC ${SSE_FLAGS})

set_target_properties(${PROJECT_NAME} PROPERTIES
  VERSION ${FCL_VERSION}
  SOVERSION ${FCL_ABI_VERSION})

target_link_libraries(${PROJECT_NAME}
  PUBLIC ${OCTOMAP_LIBRARIES}
  PUBLIC ${CCD_LIBRARIES})

target_include_directories(${PROJECT_NAME} INTERFACE
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

target_include_directories(${PROJECT_NAME} PUBLIC
  ${EIGEN3_INCLUDE_DIR})

export(TARGETS ${PROJECT_NAME}
  FILE "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake")

install(TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}Config
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

if(WIN32 AND NOT CYGWIN)
  install(EXPORT ${PROJECT_NAME}Config DESTINATION CMake)
else()
  install(EXPORT ${PROJECT_NAME}Config
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
endif()

# Setup the coveralls target and tell it to gather coverage data for all the lib sources.
if(FCL_COVERALLS)
  coveralls_setup("${FCL_HEADERS};${FCL_SOURCE_CODE};" ${FCL_COVERALLS_UPLOAD})
endif()
