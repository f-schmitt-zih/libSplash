# Projekt name
cmake_minimum_required(VERSION 2.8.5)

set(CMAKE_PREFIX_PATH "/usr/lib/x86_64-linux-gnu/" "$ENV{MPI_ROOT}")

SET(CMAKE_BUILD_TYPE Debug)
SET(CMAKE_CXX_FLAGS_DEBUG "-g")
OPTION(WITH_MPI "use MPI-based tests" OFF)

#-------------------------------------------------------------------------------

# HDF5 is required package
FIND_PACKAGE(HDF5 REQUIRED)
INCLUDE_DIRECTORIES(${HDF5_INCLUDE_DIR})
LINK_DIRECTORIES(${HDF5_LIBRARY_DIRS})

IF(WITH_MPI)
    # MPI is required package
    FIND_PACKAGE(MPI REQUIRED)
    INCLUDE_DIRECTORIES(${MPI_INCLUDE_PATH})
ENDIF(WITH_MPI)

# CPPUNIT is required package (not available yet)
#FIND_PACKAGE(CppUnit REQUIRED)
#INCLUDE_DIRECTORIES(${CPPUNIT_INCLUDE_DIR})
#LINK_DIRECTORIES(${CPPUNIT_LIBRARIES})

#-------------------------------------------------------------------------------

OPTION(TESTS_DEBUG "enable debug output for tests" OFF)
IF(TESTS_DEBUG)
    ADD_DEFINITIONS(-DTESTS_DEBUG)
ENDIF(TESTS_DEBUG)

INCLUDE_DIRECTORIES(include ../src/include $ENV{MPI_INC} )

#-------------------------------------------------------------------------------

FILE(GLOB SRCFILESOTHER "dependencies/*.cpp")
SET(TESTS Append Attributes FileAccess References Remove SimpleData Striding)

IF(WITH_MPI)
    SET(TESTS ${TESTS} Benchmark Domains MergedRead)
ENDIF(WITH_MPI)

OPTION(PARALLEL "enable tests for parallel libSplash" @HDF5_IS_PARALLEL@)
IF(PARALLEL)
    SET(TESTS ${TESTS} Parallel_Attributes Parallel_Domains Parallel_References Parallel_Remove Parallel_SimpleData)
ELSE(PARALLEL)
    SET(TESTS ${TESTS})
ENDIF(PARALLEL)

FOREACH(TEST_NAME ${TESTS})
    SET(TEST_FILES "${TEST_FILES};${TEST_NAME}Test.cpp")
ENDFOREACH()

#-------------------------------------------------------------------------------

SET(SPLASH_RELEASE CACHE BOOL OFF)
SET(WITH_TOOLS CACHE BOOL OFF)
ADD_SUBDIRECTORY(.. "${CMAKE_CURRENT_BINARY_DIR}/build_libsplash")
INCLUDE_DIRECTORIES("${CMAKE_CURRENT_BINARY_DIR}/build_libsplash")

#-------------------------------------------------------------------------------

FOREACH(TEST_FILE ${TEST_FILES})
    GET_FILENAME_COMPONENT(FILE ${TEST_FILE} NAME)
    MESSAGE(STATUS ${FILE})
    ADD_EXECUTABLE(${FILE}.out
        ${TEST_FILE}
        ${SRCFILESOTHER}
    )

    IF(${vampir})
        TARGET_LINK_LIBRARIES(${FILE}.out vt-hyb splash m cppunit ${MPI_LIBRARIES} hdf5)
    ELSE()
        TARGET_LINK_LIBRARIES(${FILE}.out splash m cppunit ${MPI_LIBRARIES} hdf5)
    ENDIF()
ENDFOREACH()
