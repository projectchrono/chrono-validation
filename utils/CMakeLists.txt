#=============================================================================
# CMake configuration file for the ChronoValidation_Utils library
#=============================================================================

# ------------------------------------------------------------------------------
# LIST THE FILES in the ChronoValidation_Utils LIBRARY
# ------------------------------------------------------------------------------

SET(CV_UTILS_FILES
    ChApiUtils.h
    ChUtilsGeometry.h
    ChUtilsCreators.h
    ChUtilsCreators.cpp
    ChUtilsInputOutput.h
    ChUtilsInputOutput.cpp
    ChUtilsValidation.h
    ChUtilsValidation.cpp
)

SOURCE_GROUP("utils" FILES ${CV_UTILS_FILES})

# ------------------------------------------------------------------------------
# ADD THE ChronoValidation_Utils LIBRARY
# ------------------------------------------------------------------------------

ADD_LIBRARY(ChronoValidation_Utils SHARED ${CV_UTILS_FILES})

SET_TARGET_PROPERTIES(ChronoValidation_Utils PROPERTIES
    COMPILE_FLAGS "${CH_BUILDFLAGS}"
    LINK_FLAGS "${CH_LINKERFLAG_GPU}"
    COMPILE_DEFINITIONS "CH_API_COMPILE_UTILS"
)

TARGET_LINK_LIBRARIES(ChronoValidation_Utils ${CHRONOENGINE_LIBRARY})

INSTALL(TARGETS ChronoValidation_Utils
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

