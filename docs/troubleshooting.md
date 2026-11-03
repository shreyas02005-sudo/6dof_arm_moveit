# Troubleshooting

## The arm flickers in RViz

**Cause:** more than one `robot_state_publisher` or `ros2_control_node` is running, usually because two launch files were started together or an old one is still alive.

**Fix:** stop everything, check for leftovers, then start a single launch.

```bash
ros2 node list | sort | uniq -d    # lists node names that appear more than once
```

If that prints anything, close the terminals that are still running a launch file and try again.

## RViz is slow or choppy on WSL2

**Cause:** RViz is using software rendering (`LIBGL_ALWAYS_SOFTWARE=1`), which is slower than GPU rendering.

**What to do:** nothing is broken. Close other heavy windows, and avoid dragging large interactive markers while a plan is executing.

## `JointTrajectoryController` crashes

**Cause:** older ros2_control releases crash in this controller. This project needs **4.48.1 or newer**.

**Fix:**

```bash
apt policy ros-jazzy-ros2-control
sudo apt update && sudo apt upgrade
```

## Changes to the URDF or a YAML file have no effect

**Cause:** these files are read when a launch starts.

**Fix:** stop the launch and start it again. If you changed C++ code or added files, rebuild:

```bash
colcon build --symlink-install
source install/setup.bash
```

## `Package not found` or `The message type is invalid`

**Cause:** the workspace isn't sourced in this terminal.

**Fix:**

```bash
source /opt/ros/jazzy/setup.bash
source install/setup.bash
```

Every new terminal needs both lines.

## The commander does nothing when I publish a command

Check these in order:

1. The bringup launch is running (`ros2 launch my_robot_bringup my_robot.launch.xml`).
2. The commander is running: `ros2 node list` should show it.
3. The topic exists: `ros2 topic list | grep command`.
4. The target pose is reachable. An unreachable pose makes planning fail, and the failure is printed in the commander's terminal.

## Starting from a clean build

If something is badly out of sync, remove the build folders and rebuild:

```bash
rm -rf build install log
colcon build --symlink-install
source install/setup.bash
```
