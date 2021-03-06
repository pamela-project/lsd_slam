cmake_minimum_required(VERSION 2.8)



find_package(OpenGL )
find_package(Pangolin )
find_package(OpenCV REQUIRED)
find_package(SuiteSparse REQUIRED) # Apparently needed by g2o
find_package(G2O REQUIRED)
#find_package(Boost COMPONENTS thread filesystem system REQUIRED)
find_package(Eigen3 REQUIRED)

# --- common includes

include_directories(
  ${EIGEN3_INCLUDE_DIR}
  ${Pangolin_INCLUDE_DIR}
  ${G2O_INCLUDE_DIR}
  ./include
  ${CSPARSE_INCLUDE_DIR} #Has been set by SuiteParse
  ${CHOLMOD_INCLUDE_DIR} #Has been set by SuiteParse
  ${OpenCV_INCLUDE_DIRS}
)


set(lsd_VERSION original_mp)

IF(NOT ANDROID) ## This version of LSDSLAM HAS BOOST DEPENDENCIES FOR MUTEX HARD TIME FOR ANDROID COMPILATION

find_package(Boost COMPONENTS thread filesystem system REQUIRED)
FIND_PACKAGE(GLUT)
include_directories(${Boost_INCLUDE_DIR})
  
# ------------ FABMAP LIB ---------

add_subdirectory(./lsd_slam_core/src//thirdparty/openFabMap)
include_directories(./lsd_slam_core/src//thirdparty/openFabMap/include)
include_directories(./lsd_slam_core/src//thirdparty/Sophus/)
add_definitions("-DHAVE_FABMAP")
set(FABMAP_LIB openFABMAP)

# ------------ COMPILER FLAGS ------------ 
# Also add some useful compiler flag
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS}  -O3 -std=c++11 -g   -fopenmp -Wno-error=deprecated-declarations") 

IF(ANDROID)
ELSE(ANDROID)
# SSE flags
#add_definitions("-DENABLE_SSE")

# Also add some useful compiler flag
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -march=native") 

ENDIF(ANDROID)

# ------------ COMMON  ------------ 
set(common_libraries     ${Boost_LIBRARIES} ${OpenCV_LIBS}  ${G2O_TYPES_SIM3} ${G2O_TYPES_SBA} ${CSPARSE_LIBRARY} ${G2O_STUFF_LIBRARY}  ${G2O_CORE_LIBRARY}  ${G2O_SOLVER_CSPARSE} ${G2O_SOLVER_CSPARSE_EXTENSION} boost_program_options boost_regex)
set(CMAKE_CXX_FLAGS  "${CMAKE_CXX_FLAGS} -Wno-unused-variable -Wno-unused-parameter -Wno-maybe-uninitialized -Wno-unused-but-set-variable") 


include_directories(./lsd_slam_core/src/)

# Set source files
set(lsd_SOURCE_FILES
  ./lsd_slam_core/src/DataStructures/Frame.cpp
  ./lsd_slam_core/src/DataStructures/FramePoseStruct.cpp
  ./lsd_slam_core/src/DataStructures/FrameMemory.cpp
  ./lsd_slam_core/src/SlamSystem.cpp
  ./lsd_slam_core/src/LiveSLAMWrapper.cpp
  ./lsd_slam_core/src/front-end-shared.cpp
  ./lsd_slam_core/src/lsdslam_library.cpp
  ./lsd_slam_core/src/DepthEstimation/DepthMap.cpp
  ./lsd_slam_core/src/DepthEstimation/DepthMapPixelHypothesis.cpp
  ./lsd_slam_core/src/util/globalFuncs.cpp
  ./lsd_slam_core/src/util/SophusUtil.cpp
  ./lsd_slam_core/src/util/settings.cpp
  ./lsd_slam_core/src/util/Undistorter.cpp
  ./lsd_slam_core/src/util/RawLogReader.cpp
  ./lsd_slam_core/src/Tracking/Sim3Tracker.cpp
  ./lsd_slam_core/src/Tracking/Relocalizer.cpp
  ./lsd_slam_core/src/Tracking/SE3Tracker.cpp
  ./lsd_slam_core/src/Tracking/least_squares.cpp
  ./lsd_slam_core/src/Tracking/TrackingReference.cpp
  ./lsd_slam_core/src/IOWrapper/Timestamp.cpp
  ./lsd_slam_core/src/GlobalMapping/FabMap.cpp
  ./lsd_slam_core/src/GlobalMapping/KeyFrameGraph.cpp
  ./lsd_slam_core/src/GlobalMapping/g2oTypeSim3Sophus.cpp
  ./lsd_slam_core/src/GlobalMapping/TrackableKeyFrameSearch.cpp
)

set(SOURCE_FILES
  ${lsd_SOURCE_FILES}
  ./lsd_slam_core/src/IOWrapper/OpenCV/ImageDisplay_OpenCV.cpp
)

# build shared library.
add_library(${appname}-${lsd_VERSION} ${SOURCE_FILES})
target_link_libraries(${appname}-${lsd_VERSION} ${common_libraries} ${FABMAP_LIB})
SET_TARGET_PROPERTIES(${appname}-${lsd_VERSION} PROPERTIES COMPILE_FLAGS "-fPIC")


 install(TARGETS ${appname}-original_mp  DESTINATION  lib/)

generate_slam_library(${appname}-original_mp)

ENDIF(NOT ANDROID)
