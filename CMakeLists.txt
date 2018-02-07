cmake_minimum_required(VERSION 2.8.3)
project(bayes_tracking)
set(CPACK_PACKAGE_NAME "bayes_tracking")
set(VERSION "1.0.3")

## Find catkin macros and libraries if installed
find_package(catkin QUIET)
## Use catkin macros and include dirs
if(catkin_FOUND)
  message(STATUS "Found catkin")
  catkin_package(
    INCLUDE_DIRS include 
    LIBRARIES ${PROJECT_NAME}
  )
  include_directories(
    ${catkin_INCLUDE_DIRS}
  )
else(catkin_FOUND)
    message(STATUS "NOT found catkin")
endif(catkin_FOUND)

## Headers
include_directories(
    include
)
## Source files
add_library(${PROJECT_NAME} STATIC 
    src/bayes_tracking/associationmatrix.cpp 
    src/bayes_tracking/ekfilter.cpp 
    src/bayes_tracking/ukfilter.cpp 
    src/bayes_tracking/pfilter.cpp 
    src/bayes_tracking/trackwin.cpp 
    src/bayes_tracking/models.cpp
    src/bayes_tracking/BayesFilter/bayesFltAlg.cpp
    src/bayes_tracking/BayesFilter/bayesFlt.cpp
    src/bayes_tracking/BayesFilter/CIFlt.cpp
    src/bayes_tracking/BayesFilter/covFlt.cpp
    src/bayes_tracking/BayesFilter/infFlt.cpp
    src/bayes_tracking/BayesFilter/infRtFlt.cpp
    src/bayes_tracking/BayesFilter/itrFlt.cpp
    src/bayes_tracking/BayesFilter/matSup.cpp
    src/bayes_tracking/BayesFilter/SIRFlt.cpp
    src/bayes_tracking/BayesFilter/UDFlt.cpp
    src/bayes_tracking/BayesFilter/UdU.cpp
    src/bayes_tracking/BayesFilter/unsFlt.cpp
)

## Link catkin libraries and set install targets
if(catkin_FOUND)
  target_link_libraries(${PROJECT_NAME}
    ${catkin_LIBRARIES}
  )
  set(PACKAGE_LIB_DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION})
  set(PACKAGE_BIN_DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})
  set(PACKAGE_INCLUDE_DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})
else(catkin_FOUND)
  find_package(Boost 1.54.0 REQUIRED)
  set(INCLUDE_DIRS_EXTRA ${Boost_INCLUDE_DIRS})
  include_directories(${INCLUDE_DIRS_EXTRA})
  link_directories(${Boost_LIBRARY_DIRS})
  target_link_libraries(${PROJECT_NAME}
    ${Boost_LIBRARIES}
  )
  set(PACKAGE_LIB_DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
  set(PACKAGE_BIN_DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)
  set(PACKAGE_INCLUDE_DESTINATION ${CMAKE_INSTALL_PREFIX}/include/${PROJECT_NAME}/)
  set(PACKAGE_INCLUDE_DIR ${CMAKE_INSTALL_PREFIX}/include)
  set(PACKAGE_CONFIG_DESTINATION ${CMAKE_INSTALL_PREFIX}/share/${PROJECT_NAME}/cmake/)
  configure_file(bayes_trackingConfig.cmake.in "${PROJECT_BINARY_DIR}/bayes_trackingConfig.cmake" @ONLY)
  configure_file(bayes_trackingConfigVersion.cmake.in "${PROJECT_BINARY_DIR}/bayes_trackingConfigVersion.cmake" @ONLY)
  install(FILES "${PROJECT_BINARY_DIR}/bayes_trackingConfig.cmake" "${PROJECT_BINARY_DIR}/bayes_trackingConfigVersion.cmake" 
    DESTINATION "${PACKAGE_CONFIG_DESTINATION}"
  )
endif(catkin_FOUND)

#add opencv
find_package(OpenCV REQUIRED)
include_directories(${OpenCV_INCLUDE_DIRS})
link_directories(${OpenCV_INCLUDE_DIRS})
target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})

install(TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION ${PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${PACKAGE_BIN_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
  DESTINATION ${PACKAGE_INCLUDE_DESTINATION}
  PATTERN ".svn" EXCLUDE
)

## Optional builds: examples
OPTION (BAYESTRACKING_BUILD_EXAMPLES "Generates example programs" ON)
if(BAYESTRACKING_BUILD_EXAMPLES)
  add_subdirectory(examples)
endif(BAYESTRACKING_BUILD_EXAMPLES)

## Optional builds: documentation
OPTION (BAYESTRACKING_BUILD_DOC "Generates API documentation" OFF)
if(BAYESTRACKING_BUILD_DOC)
  ## Check if doxygen is even installed
  find_package(Doxygen)
  if (DOXYGEN_FOUND STREQUAL "NO")
      message(FATAL_ERROR "Doxygen not found. Please get a copy http://www.doxygen.org")
  endif (DOXYGEN_FOUND STREQUAL "NO")

  ## Prepare doxygen configuration file
  configure_file(${CMAKE_SOURCE_DIR}/Doxyfile.in ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile)

  ## Add doxygen as target
  add_custom_target(doc ALL ${DOXYGEN_EXECUTABLE} ${CMAKE_CURRENT_BINARY_DIR}/Doxyfile
  COMMENT "Generating API documentation with Doxygen" VERBATIM)

  ## Cleanup $build/doc on "make clean"
  set_property(DIRECTORY APPEND PROPERTY
        ADDITIONAL_MAKE_CLEAN_FILES doc)

  ## Install HTML API documentation and manual pages
  set(DOC_PATH "share/doc/${CPACK_PACKAGE_NAME}-${VERSION}")

  install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/doc/html
    DESTINATION ${DOC_PATH}
  )

endif(BAYESTRACKING_BUILD_DOC)
