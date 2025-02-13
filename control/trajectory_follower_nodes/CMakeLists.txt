cmake_minimum_required(VERSION 3.14)
project(trajectory_follower_nodes)

find_package(autoware_cmake REQUIRED)
autoware_package()

set(CONTROLLER_NODE controller_node)
ament_auto_add_library(${CONTROLLER_NODE} SHARED
  include/trajectory_follower_nodes/controller_node.hpp
  src/controller_node.cpp
)

target_compile_options(${CONTROLLER_NODE} PRIVATE -Wno-error=old-style-cast)
rclcpp_components_register_node(${CONTROLLER_NODE}
  PLUGIN "autoware::motion::control::trajectory_follower_nodes::Controller"
  EXECUTABLE ${CONTROLLER_NODE}_exe
)

# simple trajectory follower
set(SIMPLE_TRAJECTORY_FOLLOWER_NODE simple_trajectory_follower)
ament_auto_add_library(${SIMPLE_TRAJECTORY_FOLLOWER_NODE} SHARED
  include/trajectory_follower_nodes/simple_trajectory_follower.hpp
  src/simple_trajectory_follower.cpp
)

# TODO(simple_trajectory_follower) : RCLCPP_ERROR_THROTTLE() has built-in old-style casts.
# TODO(simple_trajectory_follower) : Temporary workaround until this is fixed.
target_compile_options(${SIMPLE_TRAJECTORY_FOLLOWER_NODE} PRIVATE -Wno-error=old-style-cast)
rclcpp_components_register_node(${SIMPLE_TRAJECTORY_FOLLOWER_NODE}
  PLUGIN "simple_trajectory_follower::SimpleTrajectoryFollower"
  EXECUTABLE ${SIMPLE_TRAJECTORY_FOLLOWER_NODE}_exe
)

if(BUILD_TESTING)
  set(TRAJECTORY_FOLLOWER_NODES_TEST test_trajectory_follower_nodes)
  ament_add_ros_isolated_gtest(${TRAJECTORY_FOLLOWER_NODES_TEST}
    test/trajectory_follower_test_utils.hpp
    test/test_controller_node.cpp
  )
  ament_target_dependencies(${TRAJECTORY_FOLLOWER_NODES_TEST} fake_test_node)
  target_link_libraries(
    ${TRAJECTORY_FOLLOWER_NODES_TEST} ${CONTROLLER_NODE})

  find_package(autoware_testing REQUIRED)
  add_smoke_test(${PROJECT_NAME} ${CONTROLLER_NODE}_exe
    PARAM_FILENAMES "lateral_controller_defaults.param.yaml longitudinal_controller_defaults.param.yaml test_vehicle_info.param.yaml"
  )
endif()

ament_auto_package(
  INSTALL_TO_SHARE
  param
  launch
)
