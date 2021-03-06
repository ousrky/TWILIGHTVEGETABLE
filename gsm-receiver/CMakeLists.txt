########################################################################
# Project setup
########################################################################
cmake_minimum_required(VERSION 2.8)
project(gr-npgsm CXX C)
set(PROJECT_VERSION "0.0.4")
enable_testing()

#select the release build type by default to get optimization flags
if(NOT CMAKE_BUILD_TYPE)
   set(CMAKE_BUILD_TYPE "Debug")
   message(STATUS "Build type not specified: defaulting to debug.")
endif(NOT CMAKE_BUILD_TYPE)
set(CMAKE_BUILD_TYPE ${CMAKE_BUILD_TYPE} CACHE STRING "")

list(APPEND CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake/Modules)
########################################################################
# Compiler specific setup
########################################################################
add_definitions(-Wl,--no-undefined)
set(CMAKE_SHARED_LINKER_FLAGS "-Wl,--no-undefined")
set(CMAKE_MODULE_LINKER_FLAGS "-Wl,--no-undefined")
set(CMAKE_STATIC_LINKER_FLAGS "-Wl,--no-undefined")

if(CMAKE_COMPILER_IS_GNUCXX AND NOT WIN32)
    #http://gcc.gnu.org/wiki/Visibility
    add_definitions(-fvisibility=hidden)
endif()

if(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
    add_definitions(-fPIC)
endif(CMAKE_SYSTEM_PROCESSOR MATCHES "x86_64")
########################################################################
# Find boost
########################################################################
if(UNIX AND EXISTS "/usr/lib64")
    list(APPEND BOOST_LIBRARYDIR "/usr/lib64") #fedora 64-bit fix
endif(UNIX AND EXISTS "/usr/lib64")
set(Boost_ADDITIONAL_VERSIONS
    "1.35.0" "1.35" "1.36.0" "1.36" "1.37.0" "1.37" "1.38.0" "1.38" "1.39.0" "1.39"
    "1.40.0" "1.40" "1.41.0" "1.41" "1.42.0" "1.42" "1.43.0" "1.43" "1.44.0" "1.44"
    "1.45.0" "1.45" "1.46.0" "1.46" "1.47.0" "1.47" "1.48.0" "1.48" "1.49.0" "1.49"
    "1.50.0" "1.50" "1.51.0" "1.51" "1.52.0" "1.52" "1.53.0" "1.53" "1.54.0" "1.54"
    "1.55.0" "1.55" "1.56.0" "1.56" "1.57.0" "1.57" "1.58.0" "1.58" "1.59.0" "1.59"
    "1.60.0" "1.60" "1.61.0" "1.61" "1.62.0" "1.62" "1.63.0" "1.63" "1.64.0" "1.64"
    "1.65.0" "1.65" "1.66.0" "1.66" "1.67.0" "1.67" "1.68.0" "1.68" "1.69.0" "1.69"
)
find_package(Boost "1.35" COMPONENTS filesystem system thread)

if(NOT Boost_FOUND)
    message(FATAL_ERROR "Boost required to compile acars")
endif()

########################################################################
# Install directories
########################################################################
include(GrPlatform) #define LIB_SUFFIX
set(GR_RUNTIME_DIR      bin)
set(GR_LIBRARY_DIR      lib${LIB_SUFFIX})
set(GR_INCLUDE_DIR      include/npgsm)
set(GR_DATA_DIR         share)
set(GR_PKG_DATA_DIR     ${GR_DATA_DIR}/${CMAKE_PROJECT_NAME})
set(GR_DOC_DIR          ${GR_DATA_DIR}/doc)
set(GR_PKG_DOC_DIR      ${GR_DOC_DIR}/${CMAKE_PROJECT_NAME})
set(GR_CONF_DIR         etc)
set(GR_PKG_CONF_DIR     ${GR_CONF_DIR}/${CMAKE_PROJECT_NAME}/conf.d)
set(GR_LIBEXEC_DIR      libexec)
set(GR_PKG_LIBEXEC_DIR  ${GR_LIBEXEC_DIR}/${CMAKE_PROJECT_NAME})
set(GRC_BLOCKS_DIR      ${GR_PKG_DATA_DIR}/grc/blocks)

########################################################################
# Find gnuradio build dependencies
########################################################################
find_package(CppUnit REQUIRED)
find_package(Osmocore REQUIRED)
find_package(Threads REQUIRED)
find_package(Protobuf REQUIRED)
# To run a more advanced search for GNU Radio and it's components and
# versions, use the following. Add any components required to the list
# of GR_REQUIRED_COMPONENTS (in all caps) and change "version" to the
# minimum API compatible version required.
#
set(GR_REQUIRED_COMPONENTS RUNTIME BLOCKS FILTER UHD)
find_package(Gnuradio "3.7")

if(NOT GNURADIO_RUNTIME_FOUND)
    message(FATAL_ERROR "GnuRadio Runtime required to compile np-gsm")
endif()
if(NOT PROTOBUF_FOUND)
    message(FATAL_ERROR "Protobuf required to compile np-gsm")
endif()
if(NOT CPPUNIT_FOUND)
    message(FATAL_ERROR "CppUnit required to compile np-gsm")
endif()
if(NOT_OSMOCORE_FOUND)
    message(FATAL_ERROR "osmocore required to compile np-gsm")
endif()

list(APPEND CMAKE_MODULE_PATH ${GNURADIO_LIB_DIR}/cmake/gnuradio)
find_package(UHD REQUIRED)


########################################################################
# Setup the include and linker paths
########################################################################
include_directories(
    ${CMAKE_SOURCE_DIR}/lib
    ${CMAKE_SOURCE_DIR}/include
    ${CMAKE_BINARY_DIR}/lib
    ${CMAKE_BINARY_DIR}/include
    ${Boost_INCLUDE_DIRS}
    ${CPPUNIT_INCLUDE_DIRS}
    ${GNURADIO_RUNTIME_INCLUDE_DIRS}
    ${OSMOCORE_INCLUDE_DIRS}
)

link_directories(
    ${Boost_LIBRARY_DIRS}
    ${CPPUNIT_LIBRARY_DIRS}
    ${GNURADIO_RUNTIME_LIBRARY_DIRS}
    ${OSMOCORE_LIBRARY_DIRS}
)

# Set component parameters
set(GR_NPGSM_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/include CACHE INTERNAL "" FORCE)
set(GR_NPGSM_SWIG_INCLUDE_DIRS ${CMAKE_CURRENT_SOURCE_DIR}/swig CACHE INTERNAL "" FORCE)

#### FFTW3 jmfriedt ####
#find_package(FFTW)

#INCLUDE_DIRECTORIES(${FFTW_INCLUDE_DIR})
#LINK_LIBRARIES(${FFTW_LIBRARIES})

########################################################################
# Create uninstall target
########################################################################
configure_file(
    ${CMAKE_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
@ONLY)

add_custom_target(uninstall
    ${CMAKE_COMMAND} -P ${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake
)

########################################################################
# Add subdirectories
########################################################################
#add_subdirectory(include/npgsm)
add_subdirectory(lib)
add_subdirectory(swig)
#add_subdirectory(python)
#add_subdirectory(grc)
add_subdirectory(apps)
#add_subdirectory(docs)

########################################################################
# Install cmake search helper for this library
########################################################################
install(FILES cmake/Modules/npgsmConfig.cmake
    DESTINATION lib/cmake/npgsm
)

