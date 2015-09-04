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
    COMPILE_FLAGS "${COMPILE_FLAGS}"
    LINK_FLAGS "${COMPILE_FLAGS} ${CHRONO_LINKER_FLAGS}"
    )

  TARGET_LINK_LIBRARIES(${PROGRAM} ${CHRONO_LIBRARIES})

  # Note: this is not intended to work on Windows!
  ADD_TEST(NAME ${PROGRAM}
           WORKING_DIRECTORY ${WORK_DIR}
           COMMAND ${WORK_DIR}/${PROGRAM}
           )

ENDFOREACH()

