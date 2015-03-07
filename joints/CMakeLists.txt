# ----------------------
# Configuration options
# ----------------------
INCLUDE(CMakeDependentOption)

OPTION(ENABLE_JOINTS_TESTS "Enable Chrono joint validation tests" ON)

IF(NOT ENABLE_JOINTS_TESTS)
    RETURN()
ENDIF()

#--------------------------------------------------------------
# List here the names of all tests

MESSAGE(STATUS "Adding joint validation tests...")

SET(TEST_PROGRAMS
    test_revolute
    test_spherical
    test_universal
    test_prismatic
    test_cylindrical
    test_distance
    test_rackpinion
    test_revsph
    test_linactuator
    test_transpring
    test_transpringcb
    test_rotspring
)

#--------------------------------------------------------------
# Always use full RPATH (differentiating between the build and install trees)

# use, i.e. don't skip the full RPATH for the build tree
SET(CMAKE_SKIP_BUILD_RPATH  FALSE)

# when building, don't use the install RPATH already
# (but later on when installing)
SET(CMAKE_BUILD_WITH_INSTALL_RPATH FALSE) 

SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")

# add the automatically determined parts of the RPATH
# which point to directories outside the build tree to the install RPATH
SET(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

# the RPATH to be used when installing, but only if it's not a system directory
LIST(FIND CMAKE_PLATFORM_IMPLICIT_LINK_DIRECTORIES "${CMAKE_INSTALL_PREFIX}/lib" isSystemDir)
IF("${isSystemDir}" STREQUAL "-1")
   SET(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/lib")
ENDIF("${isSystemDir}" STREQUAL "-1")

#--------------------------------------------------------------

IF(${CMAKE_SYSTEM_NAME} MATCHES "Darwin")
  SET (CH_LINKERFLAG_EXE  "${CH_LINKERFLAG_EXE} -framework IOKit -framework Cocoa -framework OpenGL")
ENDIF()

#--------------------------------------------------------------

SET(LIBRARIES 
    ${CHRONOENGINE_LIBRARIES}
    ChronoValidation_Utils
)

IF(ENABLE_IRRLICHT AND ${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  SET(CH_BUILDFLAGS "${CH_BUILDFLAGS} /wd4275")
ENDIF()


IF(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
  SET(WORK_DIR ${PROJECT_BINARY_DIR}/bin/$<CONFIGURATION>)
ELSE()
  SET(WORK_DIR ${PROJECT_BINARY_DIR}/bin)
ENDIF()

#--------------------------------------------------------------
# Add executables

FOREACH(PROGRAM ${TEST_PROGRAMS})
  MESSAGE(STATUS "... ${PROGRAM}")
  
  ADD_EXECUTABLE(${PROGRAM}  "${PROGRAM}.cpp")
  SOURCE_GROUP(""  FILES  "${PROGRAM}.cpp")

  SET_TARGET_PROPERTIES(${PROGRAM}  PROPERTIES
    FOLDER tests
    COMPILE_FLAGS "${CH_BUILDFLAGS}"
    LINK_FLAGS "${CH_LINKERFLAG_EXE}"
    )

  TARGET_LINK_LIBRARIES(${PROGRAM} ${LIBRARIES})

  INSTALL(TARGETS ${PROGRAM} DESTINATION bin)

  # Note: this is not intended to work on Windows!
  ADD_TEST(NAME ${PROGRAM}
           WORKING_DIRECTORY ${WORK_DIR}
           COMMAND ${WORK_DIR}/${PROGRAM}
           )

ENDFOREACH()

