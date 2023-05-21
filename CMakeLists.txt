cmake_minimum_required(VERSION 2.8)
project(ORB_SLAM3)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall   -O3")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall   -O3")
set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -march=native")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -march=native")

# set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wno-deprecated -O3 -march=native ")
# set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wno-deprecated -O3 -march=native")

# Check C++11 or C++0x support
include(CheckCXXCompilerFlag)
CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
   add_definitions(-DCOMPILEDWITHC11)
   message(STATUS "Using flag -std=c++11.")
elseif(COMPILER_SUPPORTS_CXX0X)
   set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
   add_definitions(-DCOMPILEDWITHC0X)
   message(STATUS "Using flag -std=c++0x.")
else()
   message(FATAL_ERROR "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support. Please use a different C++ compiler.")
endif()

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(OpenCV 4.0)
if(NOT OpenCV_FOUND)
  find_package(OpenCV 3.0)
  if(NOT OpenCV_FOUND)
    message(FATAL_ERROR "OpenCV > 3.0 not found.")
  endif()
endif()

MESSAGE("OPENCV VERSION:")
MESSAGE(${OpenCV_VERSION})

find_package(Eigen3 3.1.0 REQUIRED)
find_package(Pangolin REQUIRED)
find_package(realsense2)

include_directories(
${PROJECT_SOURCE_DIR}
${PROJECT_SOURCE_DIR}/include
${PROJECT_SOURCE_DIR}/include/CameraModels
${EIGEN3_INCLUDE_DIR}
${Pangolin_INCLUDE_DIRS}
)

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)

add_library(${PROJECT_NAME} SHARED
orb_slam3/src/System.cc
orb_slam3/src/Tracking.cc
orb_slam3/src/LocalMapping.cc
orb_slam3/src/LoopClosing.cc
orb_slam3/src/ORBextractor.cc
orb_slam3/src/ORBmatcher.cc
orb_slam3/src/FrameDrawer.cc
orb_slam3/src/Converter.cc
orb_slam3/src/MapPoint.cc
orb_slam3/src/KeyFrame.cc
orb_slam3/src/Atlas.cc
orb_slam3/src/Map.cc
orb_slam3/src/MapDrawer.cc
orb_slam3/src/Optimizer.cc
orb_slam3/src/Frame.cc
orb_slam3/src/KeyFrameDatabase.cc
orb_slam3/src/Sim3Solver.cc
orb_slam3/src/Initializer.cc
orb_slam3/src/Viewer.cc
orb_slam3/src/ImuTypes.cc
orb_slam3/src/G2oTypes.cc
orb_slam3/src/CameraModels/Pinhole.cpp
orb_slam3/src/CameraModels/KannalaBrandt8.cpp
orb_slam3/src/OptimizableTypes.cpp
orb_slam3/src/MLPnPsolver.cpp
orb_slam3/src/TwoViewReconstruction.cc
orb_slam3/include/System.h
orb_slam3/include/Tracking.h
orb_slam3/include/LocalMapping.h
orb_slam3/include/LoopClosing.h
orb_slam3/include/ORBextractor.h
orb_slam3/include/ORBmatcher.h
orb_slam3/include/FrameDrawer.h
orb_slam3/include/Converter.h
orb_slam3/include/MapPoint.h
orb_slam3/include/KeyFrame.h
orb_slam3/include/Atlas.h
orb_slam3/include/Map.h
orb_slam3/include/MapDrawer.h
orb_slam3/include/Optimizer.h
orb_slam3/include/Frame.h
orb_slam3/include/KeyFrameDatabase.h
orb_slam3/include/Sim3Solver.h
orb_slam3/include/Initializer.h
orb_slam3/include/Viewer.h
orb_slam3/include/ImuTypes.h
orb_slam3/include/G2oTypes.h
orb_slam3/include/CameraModels/GeometricCamera.h
orb_slam3/include/CameraModels/Pinhole.h
orb_slam3/include/CameraModels/KannalaBrandt8.h
orb_slam3/include/OptimizableTypes.h
orb_slam3/include/MLPnPsolver.h
orb_slam3/include/TwoViewReconstruction.h
orb_slam3/include/Config.h
)

add_subdirectory(orb_slam3/Thirdparty/g2o)

target_link_libraries(${PROJECT_NAME}
${OpenCV_LIBS}
${EIGEN3_LIBS}
${Pangolin_LIBRARIES}
${PROJECT_SOURCE_DIR}/orb_slam3/Thirdparty/DBoW2/lib/libDBoW2.so
${PROJECT_SOURCE_DIR}/orb_slam3/Thirdparty/g2o/lib/libg2o.so
-lboost_serialization
-lcrypto
)


### Build examples


# Monocular examples
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/orb_slam3/Examples/Monocular)

add_executable(ros_mono
  src/ros_mono.cc
  src/common.cc
)
target_link_libraries(ros_mono
  ${PROJECT_NAME}
)


add_executable(mono_tum
Examples/Monocular/mono_tum.cc)
target_link_libraries(mono_tum ${PROJECT_NAME})

add_executable(mono_kitti
Examples/Monocular/mono_kitti.cc)
target_link_libraries(mono_kitti ${PROJECT_NAME})

add_executable(mono_euroc
Examples/Monocular/mono_euroc.cc)
target_link_libraries(mono_euroc ${PROJECT_NAME})

add_executable(mono_tum_vi
Examples/Monocular/mono_tum_vi.cc)
target_link_libraries(mono_tum_vi ${PROJECT_NAME})
