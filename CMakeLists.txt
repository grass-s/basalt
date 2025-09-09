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
# include_directories(
#   ${EIGEN3_INCLUDE_DIR}
# )

find_package(Pangolin REQUIRED)


find_package(TBB REQUIRED)
message(STATUS "Found TBB ${TBB_VERSION_MAJOR}.${TBB_VERSION_MINOR} (interface version ${TBB_INTERFACE_VERSION}) headers in: ${TBB_INCLUDE_DIRS}")
if (TBB_INTERFACE_VERSION LESS 11004)
  # enable global_control header for earlier TBB versions (Ubuntu 16.04, 18.04)
  add_definitions(-DTBB_PREVIEW_GLOBAL_CONTROL)
endif()

# NOTE: not specifying version, since 2, 3 or 4 is fine
find_package(OpenCV REQUIRED COMPONENTS core imgproc calib3d highgui)
message(STATUS "Found OpenCV ${OpenCV_VERSION} headers in: ${OpenCV_INCLUDE_DIRS}")
message(STATUS "Found OpenCV_LIBS: ${OpenCV_LIBS}")

# Add our own custom scoped opencv target since none is provided by OpenCV itself
add_library(basalt::opencv INTERFACE IMPORTED)
set_property(TARGET basalt::opencv PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${OpenCV_INCLUDE_DIRS})
set_property(TARGET basalt::opencv PROPERTY INTERFACE_LINK_LIBRARIES ${OpenCV_LIBS})

find_package(fmt REQUIRED)
message(STATUS "Found {fmt} ${fmt_VERSION} in: ${fmt_DIR}")

add_subdirectory(thirdparty)

# custom scoped cli11 target
add_library(basalt::cli11 INTERFACE IMPORTED)
set_property(TARGET basalt::cli11 PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/CLI11/include)

# custom scoped magic_enum target
add_library(basalt::magic_enum INTERFACE IMPORTED)
set_property(TARGET basalt::magic_enum PROPERTY INTERFACE_INCLUDE_DIRECTORIES ${CMAKE_CURRENT_SOURCE_DIR}/thirdparty/magic_enum/include)

# main library
add_library(basalt SHARED)

# List all header and source files with target_sources.
# This ensures, e.g., that QtCreator >= 6.0 properly recognize these headers to belong to the project.
# To support cmake < 3.13, use absolute paths (see: https://crascit.com/2016/01/31/enhanced-source-file-handling-with-target_sources/)
target_sources(basalt
  PUBLIC
    # ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/aprilgrid.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/calibration_helper.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/cam_calib.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/cam_imu_calib.h
    # ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/calibration/vignette.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/device/rs_t265.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/hash_bow/hash_bow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_euroc.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_kitti.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_rosbag.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/dataset_io_uzh.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/io/marg_data_io.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/block_diagonal.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/imu_block.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/landmark_block.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/landmark_block_abs_dynamic.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_abs_qr.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_abs_sc.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_base.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/linearization/linearization_rel_sc.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/frame_to_frame_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/multiscale_frame_to_frame_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patch.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patch_optical_flow.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optical_flow/patterns.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/accumulator.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/poses_linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/poses_optimize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/spline_linearize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/optimization/spline_optimize.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/ba_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/cast_utils.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/common_types.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/filesystem.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/format.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/imu_types.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/keypoints.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/nfr.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/sim_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/system_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/test_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/time_utils.hpp
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/tracks.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/union_find.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/vio_config.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/utils/vis_utils.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/landmark_database.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/marg_helper.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/nfr_mapper.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sc_ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_ba_base.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_keypoint_vio.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/sqrt_keypoint_vo.h
    ${CMAKE_CURRENT_SOURCE_DIR}/include/basalt/vi_estimator/vio_estimator.h
  PRIVATE
    # ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/aprilgrid.cpp
    # ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/calibraiton_helper.cpp
    # ${CMAKE_CURRENT_SOURCE_DIR}/src/calibration/vignette.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/io/dataset_io.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/io/marg_data_io.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/landmark_block.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_abs_qr.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_abs_sc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/linearization/linearization_rel_sc.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/optical_flow/optical_flow.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/keypoints.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/system_utils.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/time_utils.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/utils/vio_config.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/landmark_database.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/marg_helper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/nfr_mapper.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sc_ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_ba_base.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_keypoint_vio.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/sqrt_keypoint_vo.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/src/vi_estimator/vio_estimator.cpp
)

target_link_libraries(basalt
  PUBLIC ${STD_CXX_FS} basalt::opencv basalt::basalt-headers TBB::tbb
  PRIVATE basalt::magic_enum rosbag opengv nlohmann::json fmt)
target_include_directories(basalt PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/include)
target_compile_definitions(basalt PUBLIC ${BASALT_COMPILE_DEFINITIONS})
#target_compile_definitions(basalt PUBLIC BASALT_DISABLE_ASSERTS)


# add_executable(basalt_calibrate src/calibrate.cpp src/calibration/cam_calib.cpp)
# target_link_libraries(basalt_calibrate basalt pangolin basalt::cli11)

# add_executable(basalt_calibrate_imu src/calibrate_imu.cpp src/calibration/cam_imu_calib.cpp)
# target_link_libraries(basalt_calibrate_imu basalt pangolin basalt::cli11)


# add_executable(basalt_vio_sim src/vio_sim.cpp)
# target_link_libraries(basalt_vio_sim basalt pangolin basalt::cli11)

add_executable(basalt_mapper_sim src/mapper_sim.cpp)
target_link_libraries(basalt_mapper_sim basalt ${Pangolin_LIBRARY} basalt::cli11)

# mapper sim native doesn't use template free interface
if(BASALT_INSTANTIATIONS_DOUBLE)
  add_executable(basalt_mapper_sim_naive src/mapper_sim_naive.cpp)
  target_link_libraries(basalt_mapper_sim_naive basalt ${Pangolin_LIBRARY} basalt::cli11)
endif()

add_executable(basalt_mapper src/mapper.cpp)
target_link_libraries(basalt_mapper basalt ${Pangolin_LIBRARY} basalt::cli11)


add_executable(basalt_opt_flow src/opt_flow.cpp)
target_link_libraries(basalt_opt_flow basalt ${Pangolin_LIBRARY} basalt::cli11 fmt)

add_executable(basalt_vio src/vio.cpp)
target_link_libraries(basalt_vio basalt ${Pangolin_LIBRARY} basalt::cli11 fmt)

add_executable(basalt_time_alignment src/time_alignment.cpp)
target_link_libraries(basalt_time_alignment basalt ${Pangolin_LIBRARY} basalt::cli11 fmt)

add_executable(basalt_kitti_eval src/kitti_eval.cpp)
target_link_libraries(basalt_kitti_eval basalt::basalt-headers basalt::cli11 fmt)

find_package(realsense2 QUIET)
if(realsense2_FOUND)
  add_executable(basalt_rs_t265_record src/rs_t265_record.cpp src/device/rs_t265.cpp)
  target_link_libraries(basalt_rs_t265_record basalt realsense2::realsense2 ${Pangolin_LIBRARY} basalt::cli11)

  add_executable(basalt_rs_t265_vio src/rs_t265_vio.cpp src/device/rs_t265.cpp)
  target_link_libraries(basalt_rs_t265_vio basalt realsense2::realsense2 ${Pangolin_LIBRARY} basalt::cli11)
endif()



# install(TARGETS basalt_vio_sim basalt_mapper_sim basalt_mapper basalt_opt_flow basalt_vio basalt_kitti_eval basalt_time_alignment basalt
#   EXPORT BasaltTargets
#   RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
#   LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
#   ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)

# if(BASALT_INSTANTIATIONS_DOUBLE)
#   install(TARGETS basalt_mapper_sim_naive
#     EXPORT BasaltTargets
#     RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
#     LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
#     ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
# endif()

# if(realsense2_FOUND)
#     install(TARGETS basalt_rs_t265_record basalt_rs_t265_vio
#       EXPORT BasaltTargets
#       RUNTIME DESTINATION ${CMAKE_INSTALL_PREFIX}/bin
#       LIBRARY DESTINATION ${CMAKE_INSTALL_PREFIX}/lib
#       ARCHIVE DESTINATION ${CMAKE_INSTALL_PREFIX}/lib)
# endif()

# file(GLOB SCRIPTS_TO_INSTALL "${CMAKE_CURRENT_SOURCE_DIR}/scripts/basalt_*.py")
# install(PROGRAMS ${SCRIPTS_TO_INSTALL} DESTINATION ${CMAKE_INSTALL_PREFIX}/bin)

# file(GLOB CONFIG_FILES "${CMAKE_CURRENT_SOURCE_DIR}/data/*.json")
# install(FILES ${CONFIG_FILES}
#   DESTINATION ${CMAKE_INSTALL_PREFIX}/etc/basalt)


# # Replace install() to do-nothing macro.
# macro(install)
# endmacro()
# # Include subproject (or any other CMake code) with "disabled" install().
# enable_testing()
# add_subdirectory(thirdparty/basalt-headers/test)
# add_subdirectory(test)
# # Restore original install() behavior.
# macro(install)
#   _install(${ARGN})
# endmacro()
