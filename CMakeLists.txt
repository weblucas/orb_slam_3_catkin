cmake_minimum_required(VERSION 2.8.3)
project(orb_slam_3_catkin)

add_definitions(-std=c++11)

find_package(catkin_simple REQUIRED)
catkin_simple()

include_directories(${Eigen_INCLUDE_DIRS})

include(ExternalProject)

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)
file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/CameraModels)
set(ORB_SLAM3_CODE_DIR ${CMAKE_SOURCE_DIR}/ORB_SLAM3) #${CMAKE_SOURCE_DIR}/ORB_SLAM3  
#set(ORB_SLAM3_BUILD_DIR ${CMAKE_SOURCE_DIR}/build)
#set(ORB_SLAM3_CODE_DIR ${CMAKE_SOURCE_DIR}/ORB_SLAM3)



ExternalProject_Add(orb_slam_3_src
  #GIT_REPOSITORY git@github.com:weblucas/ORB_SLAM3_aerial.git
  SOURCE_DIR ${ORB_SLAM3_CODE_DIR} 
  PREFIX ${CMAKE_SOURCE_DIR}/base
  #BINARY_DIR ${ORB_SLAM3_BUILD_DIR}
  UPDATE_COMMAND ""
  #PATCH_COMMAND patch -p1 < ${CMAKE_SOURCE_DIR}/fix_usleep.patch
  CONFIGURE_COMMAND
      # Configuring DBoW2
      cd ${ORB_SLAM3_CODE_DIR}/Thirdparty/DBoW2 &&
      mkdir -p build && 
      cd build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release &&
      # Configuring g2o
      cd ${ORB_SLAM3_CODE_DIR}/Thirdparty/g2o &&
      mkdir -p build &&
      cd build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release &&
      # Uncompressing the vocabulary
      cd ${ORB_SLAM3_CODE_DIR}/Vocabulary &&
      tar -xf ${ORB_SLAM3_CODE_DIR}/Vocabulary/ORBvoc.txt.tar.gz &&
      # Configuring the orb slam 3 build
      mkdir -p ${ORB_SLAM3_CODE_DIR}/build &&
      cd ${ORB_SLAM3_CODE_DIR}/build &&
      cmake .. -DCMAKE_BUILD_TYPE=Release
  BUILD_COMMAND 
      # Building DBoW2
      cd ${ORB_SLAM3_CODE_DIR}/Thirdparty/DBoW2/build/ &&
      make -j &&
      # Building g2o
      cd ${ORB_SLAM3_CODE_DIR}/Thirdparty/g2o/build/ &&
      make -j &&
      # Building orb slam 2
      cd ${ORB_SLAM3_CODE_DIR}/build/ &&
      make -j
  INSTALL_COMMAND
      cp ${ORB_SLAM3_CODE_DIR}/lib/libORB_SLAM3${CMAKE_SHARED_LIBRARY_SUFFIX} ${CATKIN_DEVEL_PREFIX}/lib/ &&
      cp -r ${ORB_SLAM3_CODE_DIR}/include/. ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3 &&
      mkdir -p ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/DBoW2/ &&
      cp -r ${ORB_SLAM3_CODE_DIR}/Thirdparty/DBoW2/DBoW2 ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/DBoW2/ &&
      cp -r ${ORB_SLAM3_CODE_DIR}/Thirdparty/DBoW2/DUtils ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/DBoW2/ &&
      mkdir -p ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/g2o/ &&
      cp -r ${ORB_SLAM3_CODE_DIR}/Thirdparty/g2o/g2o/ ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/g2o/ &&
      cp -r ${ORB_SLAM3_CODE_DIR}/Thirdparty/g2o/config.h ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/Thirdparty/g2o/
)

cs_add_library(${PROJECT_NAME} src/dependency_tracker.cpp)
add_dependencies(${PROJECT_NAME} ${ORB_SLAM3_CODE_DIR})
target_link_libraries(${PROJECT_NAME}
  ${CATKIN_DEVEL_PREFIX}/lib/libORB_SLAM3${CMAKE_SHARED_LIBRARY_SUFFIX} ${catkin_LIBRARIES})

cs_install()

cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/ ${CATKIN_DEVEL_PREFIX}/include/orb_slam_3/CameraModels
          CFG_EXTRAS orb_slam_3-extras.cmake)
