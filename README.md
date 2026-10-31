# 6-DOF Arm with Gripper (ROS 2 Jazzy + MoveIt 2)

I built this to learn the full ROS 2 manipulation stack end to end: modelling a robot, configuring MoveIt 2, running it through ros2_control, and commanding it from my own C++ node. The arm and gripper run in simulation on mock hardware, so no real robot is needed.

## Demo

| MoveIt in RViz (manual planning) | Commander node (`ros2 topic pub`) |
|---|---|
| ![MoveIt in RViz](docs/rviz-1.jpg) | ![Commander API](docs/rviz-2.jpg) |

Two short clips are in this [Google Drive folder](https://drive.google.com/drive/folders/1jh_nVYlnCgurntZmtTK8vnjJQu4upVwr?usp=sharing):
- Setting goal poses by hand in the RViz MotionPlanning panel and using Plan & Execute
- Driving the arm and gripper from the terminal through the commander node

## What's in it
- **Robot model:** a URDF/xacro description of a 6-joint arm with a prismatic parallel gripper. The right finger follows the left through a mimic joint.
- **Motion planning:** a MoveIt 2 config generated with the Setup Assistant, then edited by hand (joint limits, kinematics, controllers).
- **Control:** ros2_control with `mock_components/GenericSystem`, with one trajectory controller for the arm and one for the gripper.
- **Commander node (C++):** takes pose, joint and gripper commands over ROS topics and sends them to MoveIt. A pose command can either plan a normal move or a straight-line Cartesian path.

## Requirements
- Ubuntu 24.04 (I developed this on WSL2)
- ROS 2 Jazzy and MoveIt 2: `sudo apt install ros-jazzy-moveit`
- ros2_control 4.48.1 or newer. The version that came with my first install segfaulted in `JointTrajectoryController::on_init`, so run `sudo apt update && sudo apt upgrade` before building.

## Build
```bash
git clone https://github.com/shreyas02005-sudo/6dof_arm_moveit.git
cd 6dof_arm_moveit

# first time only: set up rosdep
sudo rosdep init
rosdep update

rosdep install --from-paths . --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```
Always run `colcon build` from the workspace root (the repo folder), not from `src/`.

## Run

### 1. Plan and execute by hand in RViz
```bash
ros2 launch my_robot_moveit_config demo.launch.py
```
In RViz, drag the goal pose in the MotionPlanning panel and click Plan & Execute.

### 2. Control it with the commander node
Use three terminals, and source the workspace in each.

Terminal 1 starts robot_state_publisher, ros2_control with mock hardware, the controllers, move_group and RViz:
```bash
ros2 launch my_robot_bringup my_robot.launch.xml
```

Terminal 2 starts the commander:
```bash
ros2 run my_robot_commander_cpp commander
```

Terminal 3 sends commands:
```bash
# Planned move to a pose
ros2 topic pub -1 /pose_command my_robot_interfaces/msg/PoseCommand "{x: 0.7, y: 0.0, z: 0.4, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: false}"

# Straight-line move down
ros2 topic pub -1 /pose_command my_robot_interfaces/msg/PoseCommand "{x: 0.7, y: 0.0, z: 0.2, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: true}"

# Close the gripper
ros2 topic pub --once /open_gripper example_interfaces/msg/Bool "{data: false}"

# Straight-line move sideways
ros2 topic pub -1 /pose_command my_robot_interfaces/msg/PoseCommand "{x: 0.7, y: 0.3, z: 0.2, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: true}"

# Open the gripper
ros2 topic pub --once /open_gripper example_interfaces/msg/Bool "{data: true}"
```

Only run one launch at a time (see the notes at the bottom).

## Commander topics
| Topic | Type | What it does |
|---|---|---|
| `/pose_command` | `my_robot_interfaces/msg/PoseCommand` | Moves the end effector to x, y, z, roll, pitch, yaw. With `cartesian_path: true` the tool travels in a straight line. With `false`, the planner picks any collision-free path. |
| `/joint_command` | `std_msgs/msg/Float64MultiArray` | Moves to six joint values |
| `/open_gripper` | `example_interfaces/msg/Bool` | `true` opens the gripper, `false` closes it |

## Robot details
- Joints: `joint1` to `joint6`, plus `gripper_left_finger_joint` (prismatic, 0 to 0.06 m) with `gripper_right_finger_joint` mimicking it
- Planning groups: `arm` and `gripper`
- Named states: `home`, `pose_1`, `pose_2`, `gripper_open`, `gripper_half_closed`, `gripper_closed`

## Packages
| Package | Contents |
|---|---|
| `my_robot_description` | URDF/xacro files, display launch, RViz config |
| `my_robot_moveit_config` | Setup Assistant output (SRDF, kinematics, joint limits, controllers) |
| `my_robot_bringup` | Launch file, controller config, RViz config |
| `my_robot_interfaces` | `PoseCommand.msg` |
| `my_robot_commander_cpp` | The C++ commander node |

## Problems I ran into
- **Planning failed without a clear error.** My joint limits had no acceleration values, and the time parameterization step needs them. Adding them fixed it.
- **The right gripper finger moved the wrong way.** The mimic joint had its lower and upper limits inverted for a multiplier of -1.0. Correcting the bounds fixed it.
- **The controller crashed on startup.** The ros2_control version from my first install segfaulted in `JointTrajectoryController::on_init`. Upgrading to 4.48.1 solved it.
- **Old builds kept causing odd errors.** Stale copies in `install/` were being picked up. Now I build only from the workspace root and delete `build/ install/ log/` when something looks wrong.
- **The arm flickered in RViz.** I had two launches running, so there were duplicate `robot_state_publisher` and `ros2_control_node` instances. One launch at a time fixes it.

## Limitations
- Simulation only. The mock hardware just echoes commands back, so there is no dynamics or contact.
- RViz can look choppy on WSL2 because I run it with software rendering (`LIBGL_ALWAYS_SOFTWARE=1`).
- After any URDF or YAML change, relaunch everything. The running nodes don't pick up changes.

## License
Apache-2.0
