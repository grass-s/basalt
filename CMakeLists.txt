cmake_minimum_required(VERSION 3.10...3.18)

project(basalt)

set(CMAKE_BUILD_TYPE "Release")
# set(CMAKE_BUILD_TYPE "Debug")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_REQUIRED_FLAGS "-Wno-error")

option(BASALT_INSTANTIATIONS_DOUBLE "Instatiate templates for Scalar=double." ON)
option(BASALT_INSTANTIATIONS_FLOAT "Instatiate templates for Scalar=float." ON)
if(BASALT_INSTANTIATIONS_DOUBLE)
  list(APPEND BASALT_COMPILE_DEFINITIONS BASALT_INSTANTIATIONS_DOUBLE)
endif()
if(BASALT_INSTANTIATIONS_FLOAT)
  list(APPEND BASALT_COMPILE_DEFINITIONS BASALT_INSTANTIATIONS_FLOAT)
endif()

find_package(Eigen3 REQUIRED)
find_package(Pangolin REQUIRED)

find_package(TBB REQUIRED)

find_package(OpenCV REQUIRED COMPONENTS core imgproc calib3d highgui)
# Add our own custom scoped opencv target since none is provided by OpenCV itself
# add_library(basalt::opencv INTERFACE IMPORTED)
# set_property(TARGET basalt::opencv PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${OpenCV_INCLUDE_DIRS})
# set_property(TARGET basalt::opencv PROPERTY INTERFACE_LINK_LIBRARIES ${OpenCV_LIBS})

find_package(fmt REQUIRED)
message(STATUS "Found {fmt} ${fmt_VERSION} in: ${fmt_DIR}")

include_directories(
  include
  thirdparty/CLI11/include
  thirdparty/magic_enum/include
  thirdparty/json
  ${EIGEN3_INCLUDE_DIR}
  ${OpenCV_INCLUDE_DIRS}
)

add_subdirectory(thirdparty)


# file(GLOB_RECURSE ALL_SRC_FILES "src/*.cpp")
# list(FILTER ALL_SRC_FILES EXCLUDE REGEX "src/[^/]*\\.cpp$") # 只保留在子目录中的文件
file(GLOB calibration_SRCS "src/calibration/*.cpp")
file(GLOB io_SRCS "src/io/*.cpp")
file(GLOB linearization_SRCS "src/linearization/*.cpp")
file(GLOB optical_flow_SRCS "src/optical_flow/*.cpp")
file(GLOB utils_SRCS "src/utils/*.cpp")
file(GLOB vi_estimator_SRCS "src/vi_estimator/*.cpp")

add_library(basalt
  # ${calibration_SRCS}
  ${io_SRCS}
  ${linearization_SRCS}
  ${optical_flow_SRCS}
  ${utils_SRCS}
  ${vi_estimator_SRCS}
)
target_link_libraries(basalt ${OpenCV_LIBS} TBB::tbb opengv fmt rosbag)
# target_include_directories(basalt PRIVATE
#   ${CMAKE_CURRENT_SOURCE_DIR}/include
#   # ${EIGEN3_INCLUDE_DIR}
#   thirdparty/basalt-headers/include
# )


# add_executable(basalt_calibrate src/calibrate.cpp src/calibration/cam_calib.cpp)
# target_link_libraries(basalt_calibrate basalt pangolin basalt::cli11)

# add_executable(basalt_calibrate_imu src/calibrate_imu.cpp src/calibration/cam_imu_calib.cpp)
# target_link_libraries(basalt_calibrate_imu basalt pangolin basalt::cli11)


# add_executable(basalt_vio_sim src/vio_sim.cpp)
# target_link_libraries(basalt_vio_sim basalt pangolin basalt::cli11)

add_executable(basalt_mapper_sim src/mapper_sim.cpp)
target_link_libraries(basalt_mapper_sim basalt ${Pangolin_LIBRARY})

# mapper sim native doesn't use template free interface
if(BASALT_INSTANTIATIONS_DOUBLE)
  add_executable(basalt_mapper_sim_naive src/mapper_sim_naive.cpp)
  target_link_libraries(basalt_mapper_sim_naive basalt ${Pangolin_LIBRARY})
endif()

add_executable(basalt_mapper src/mapper.cpp)
target_link_libraries(basalt_mapper basalt ${Pangolin_LIBRARY})


add_executable(basalt_opt_flow src/opt_flow.cpp)
target_link_libraries(basalt_opt_flow basalt ${Pangolin_LIBRARY} fmt)

add_executable(basalt_vio src/vio.cpp)
target_link_libraries(basalt_vio basalt ${Pangolin_LIBRARY} fmt)

add_executable(basalt_time_alignment src/time_alignment.cpp)
target_link_libraries(basalt_time_alignment basalt ${Pangolin_LIBRARY} fmt)

# add_executable(basalt_kitti_eval src/kitti_eval.cpp)
# target_link_libraries(basalt_kitti_eval fmt)
