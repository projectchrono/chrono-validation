#=============================================================================
# CMake configuration file for Chrono-Validation
#
#=============================================================================

cmake_minimum_required(VERSION 2.8)

project(ChronoValidation)

# Location of additional CMAKE modules
SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)

# Enable CTest
ENABLE_TESTING()
INCLUDE(CTest)

# ------------------------------------------------------------------------------
# Specify output locations for libraries and executables
# ------------------------------------------------------------------------------

SET(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/bin)
SET(LIBRARY_OUTPUT_PATH ${PROJECT_BINARY_DIR}/lib)

IF (MSVC OR XCODE_VERSION)
  SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
ENDIF()

# ------------------------------------------------------------------------------
# Always use full RPATH (differentiating between the build and install trees)
# ------------------------------------------------------------------------------

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

#-----------------------------------------------------------------------------
# Disable some warnings
#-----------------------------------------------------------------------------

if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    if(MSVC)
        add_definitions( "-D_CRT_SECURE_NO_DEPRECATE" )  # avoids deprecation warnings
        add_definitions( "-D_SCL_SECURE_NO_DEPRECATE" )  # avoids deprecation warnings
        add_definitions( "-DNOMINMAX" ) # do not use MSVC's min/max macros
        add_definitions( "-MP" ) # perform parallel builds
    endif(MSVC)
endif()

# ------------------------------------------------------------------------------
# Find the Chrono package with optional Irrlicht component
# ------------------------------------------------------------------------------

# Invoke find_package in CONFIG mode
find_package(Chrono
             OPTIONAL_COMPONENTS Irrlicht
             CONFIG
)

# If Chrono was not found, return now.
if(NOT Chrono_FOUND)
  message("Could not find Chrono package")
  return()
endif()

# ------------------------------------------------------------------------------
# Paths to include headers
# ------------------------------------------------------------------------------

# Add paths to Chrono headers
INCLUDE_DIRECTORIES(${CHRONO_INCLUDE_DIRS})

# Add paths to the top of the source directory and the binary directory
INCLUDE_DIRECTORIES(
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_BINARY_DIR}
)

#--------------------------------------------------------------
# Compilation flags
#--------------------------------------------------------------

set(COMPILE_FLAGS ${CHRONO_CXX_FLAGS})

# Disable some warnings triggered by Irrlicht (Windows only)
if(${CMAKE_SYSTEM_NAME} MATCHES "Windows")
    SET(COMPILE_FLAGS "${COMPILE_FLAGS} /wd4275")
endif()

# ------------------------------------------------------------------------------
# Automatically copy DLLs 
# ------------------------------------------------------------------------------

# Add custom command for copying Chrono DLLs to the appropriate binary output
# folder (on Windows only). This CMake function is defined in Chronoconfig.cmake.
# Note that this requires EXECUTABLE_OUTPUT_PATH to be defined.

add_DLL_copy_command("${CHRONO_DLLS}")

# ------------------------------------------------------------------------------
# Copy the data folder to the bin directory
# ------------------------------------------------------------------------------

IF(MSVC)
  FILE( COPY ${CMAKE_SOURCE_DIR}/data/ DESTINATION ${CMAKE_BINARY_DIR}/bin/data/)
ELSEIF(XCODE_VERSION)
  FILE( COPY ${CMAKE_SOURCE_DIR}/data/ DESTINATION ${CMAKE_BINARY_DIR}/bin/data/)
ELSE()
  FILE( COPY ${CMAKE_SOURCE_DIR}/data/ DESTINATION ${CMAKE_BINARY_DIR}/data/)
ENDIF()

# ------------------------------------------------------------------------------
# Create configuration file
# ------------------------------------------------------------------------------

# Prepare replacement variables

IF(CHRONO_IRRLICHT_FOUND)
  SET(IRRLICHT_ENABLED "1")
ELSE()
  SET(IRRLICHT_ENABLED "0")
ENDIF()

SET(CHRONO_DATA_DIR "${CHRONO_DATA_DIR}")

# Generate the configuration header file using substitution variables.
# Place the header file in the library output directory.

CONFIGURE_FILE(
  ${CMAKE_MODULE_PATH}/ChronoValidation_config.h.in
  ${PROJECT_BINARY_DIR}/ChronoValidation_config.h
  )

# ------------------------------------------------------------------------------
# Add subdirectories
# ------------------------------------------------------------------------------

message(STATUS "==== Add tests ====")

ADD_SUBDIRECTORY(joints)
