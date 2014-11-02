########################################################################
# Project setup
########################################################################
cmake_minimum_required(VERSION 2.8)
project(SoapySDRJava CXX)
enable_testing()

find_package(SoapySDR CONFIG REQUIRED)

########################################################################
# Find SWIG
########################################################################
include(FindSWIG)

if (NOT SWIG_FOUND)
    message(WARNING "Java bindings require SWIG, skipping...")
    return()
endif ()
message(STATUS "SWIG_VERSION: ${SWIG_VERSION}")

include(UseSWIG)

########################################################################
## Find JNI
########################################################################
find_package(JNI)

if(NOT JNI_FOUND)
    message(WARNING "Java bindings require JNI, skipping...")
    return()
endif()

message(STATUS "JNI_INCLUDE_DIRS: ${JNI_INCLUDE_DIRS}")
message(STATUS "JNI_LIBRARIES: ${JNI_LIBRARIES}")

########################################################################
## Find Java
########################################################################
find_package(Java)

if(NOT Java_JAVA_EXECUTABLE)
    message(WARNING "Java bindings require Java, skipping...")
    return()
endif()
message(STATUS "Java_VERSION: ${Java_VERSION}")

include(UseJava)

########################################################################
# Build Module
########################################################################
include_directories(${SoapySDR_INCLUDE_DIRS})
include_directories(${JNI_INCLUDE_DIRS})

set(CMAKE_SWIG_FLAGS -c++ -package SoapySDR)
set_source_files_properties(SoapySDR.i PROPERTIES CPLUSPLUS ON)

SWIG_ADD_MODULE(SoapySDRJava java SoapySDR.i)
SWIG_LINK_LIBRARIES(SoapySDRJava ${SoapySDR_LIBRARIES} ${JNI_LIBRARIES})

#known set of build products
set(java_out_files
    Device.java
    Range.java
    SoapySDRConstants.java
    SoapySDR.java
    SoapySDRJNI.java
    SoapySDRKwargs.java
    SoapySDRKwargsList.java
    SoapySDRRangeList.java
    SoapySDRSizeList.java
    SoapySDRStringList.java
    SWIGTYPE_p_int.java
    SWIGTYPE_p_long_long.java
    SWIGTYPE_p_p_void.java
    SWIGTYPE_p_SoapySDR__Stream.java
    SWIGTYPE_p_std__complexT_double_t.java
    SWIGTYPE_p_std__vectorT_double_t.java
)

foreach(java_out_file ${java_out_files})
    list(APPEND java_out_files_fp ${CMAKE_CURRENT_BINARY_DIR}/${java_out_file})
endforeach(java_out_file)

set(CMAKE_JNI_TARGET TRUE)
set(CMAKE_JAVA_JAR_ENTRY_POINT SoapySDR)
add_jar(SoapySDRJar ${java_out_files_fp})
add_dependencies(SoapySDRJar SoapySDRJava)

#install stuff...
#install_jar(SoapySDRJar lib)
#install_jni_symlink(SoapySDRJar .)


