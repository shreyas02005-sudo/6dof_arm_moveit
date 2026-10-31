# 6-DOF Robotic Arm with Gripper (ROS 2 Jazzy, MoveIt 2)

Simulated 6-axis arm with a gripper, built with ROS 2 Jazzy and MoveIt 2 on Ubuntu 24.04.

## Packages
- `my_robot_description` - URDF/xacro model of the arm and gripper
- `my_robot_moveit_config` - MoveIt 2 configuration (planning groups, kinematics, controllers)
- `my_robot_bringup` - launch files that start the full application
- `my_robot_interfaces` - custom messages
- `my_robot_commander_cpp` - C++ node that controls the arm and gripper with the MoveIt API

## Build
    cd ~/ros2_ws
    colcon build --symlink-install
    source install/setup.bash

## Run
    ros2 launch my_robot_moveit_config demo.launch.py
    ros2 run my_robot_commander_cpp commander
