cmake_minimum_required(VERSION 3.8)
project(mycobot_system_tests)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

# Find dependencies
find_package(ament_cmake REQUIRED)
find_package(controller_manager REQUIRED)
find_package(gripper_controllers REQUIRED)
find_package(rclcpp REQUIRED)
find_package(ros2_control REQUIRED)
find_package(ros2_controllers REQUIRED)
find_package(trajectory_msgs REQUIRED)
find_package(xacro REQUIRED)

# Create C++ executable
add_executable(arm_gripper_loop_controller
  src/arm_gripper_loop_controller.cpp
)

# Link ROS 2 dependencies
ament_target_dependencies(
  arm_gripper_loop_controller
  rclcpp
  controller_manager
  gripper_controllers
  ros2_control
  ros2_controllers
  trajectory_msgs
)

# Install executable
install(
  TARGETS
    arm_gripper_loop_controller
  DESTINATION lib/${PROJECT_NAME}
)

# Install source/scripts if needed
install(
  DIRECTORY
    scripts
  DESTINATION share/${PROJECT_NAME}
)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)

  set(ament_cmake_copyright_FOUND TRUE)
  set(ament_cmake_cpplint_FOUND TRUE)

  ament_lint_auto_find_test_dependencies()
endif()

ament_package()
