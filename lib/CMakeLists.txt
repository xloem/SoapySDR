########################################################################
# Soapy SDR sources and dependencies
########################################################################
list(APPEND SOAPY_SDR_SOURCES
    Device.cpp
    Factory.cpp
    Registry.cpp
    Types.cpp
    NullDevice.cpp
    Logger.cpp
    Errors.cpp
)

#dl libs used by dlopen in unix
list(APPEND SOAPY_SDR_LIBRARIES
    ${CMAKE_DL_LIBS}
)

#support threaded client code
#notice that -pthread is not the same as -lpthread
if(CMAKE_COMPILER_IS_GNUCXX)
    list(APPEND SOAPY_SDR_LIBRARIES -pthread)
endif()

include_directories(${PROJECT_SOURCE_DIR}/include)

list(APPEND SOAPY_SDR_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/Modules.cpp)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/Modules.in.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/Modules.cpp
@ONLY)

list(APPEND SOAPY_SDR_SOURCES ${CMAKE_CURRENT_BINARY_DIR}/Version.cpp)
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/Version.in.cpp
    ${CMAKE_CURRENT_BINARY_DIR}/Version.cpp
@ONLY)

#C API support sources
list(APPEND SOAPY_SDR_SOURCES
    TypesC.cpp
    ModulesC.cpp
    VersionC.cpp
    DeviceC.cpp
    FactoryC.cpp
    LoggerC.cpp
    TimeC.cpp
    ErrorsC.cpp
)

#disable MSVC warnings
if (MSVC)
    add_definitions(-D_CRT_NONSTDC_NO_DEPRECATE)
endif ()

########################################################################
# Extract SOVERSION
########################################################################
file(READ "${PROJECT_SOURCE_DIR}/include/SoapySDR/Version.h" version_h)
string(REGEX MATCH "\\#define SOAPY_SDR_ABI_VERSION \"([0-9]+\\.[0-9]+-[0-9]+)\"" SOAPY_SDR_ABI_VERSION_MATCHES "${version_h}")
if(NOT SOAPY_SDR_ABI_VERSION_MATCHES)
    message(FATAL_ERROR "Failed to extract version number from Version.h")
endif(NOT SOAPY_SDR_ABI_VERSION_MATCHES)
set(SOAPY_SDR_SOVER "${CMAKE_MATCH_1}")

########################################################################
# Build the library
########################################################################
add_library(SoapySDR SHARED ${SOAPY_SDR_SOURCES})
if (SOAPY_SDR_LIBRARIES)
    target_link_libraries(SoapySDR ${SOAPY_SDR_LIBRARIES})
endif()
set_target_properties(SoapySDR PROPERTIES SOVERSION ${SOAPY_SDR_SOVER})
set_target_properties(SoapySDR PROPERTIES VERSION ${SOAPY_SDR_LIBVER})
set_target_properties(SoapySDR PROPERTIES DEFINE_SYMBOL "SOAPY_SDR_DLL_EXPORTS")

########################################################################
# Install the library
########################################################################
install(TARGETS SoapySDR
    LIBRARY DESTINATION lib${LIB_SUFFIX} # .so file
    ARCHIVE DESTINATION lib${LIB_SUFFIX} # .lib file
    RUNTIME DESTINATION bin              # .dll file
)

########################################################################
# Build pkg config file
########################################################################
configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/SoapySDR.in.pc
    ${CMAKE_CURRENT_BINARY_DIR}/SoapySDR.pc
@ONLY)

install(
    FILES ${CMAKE_CURRENT_BINARY_DIR}/SoapySDR.pc
    DESTINATION lib${LIB_SUFFIX}/pkgconfig
)
