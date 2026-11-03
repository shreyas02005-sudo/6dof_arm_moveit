# Commander API

The commander node (`my_robot_commander_cpp`) listens on three topics. Start it with the bringup launch running:

```bash
ros2 run my_robot_commander_cpp commander
```

## Topics

| Topic | Type | Use |
|-------|------|-----|
| `/pose_command` | `my_robot_interfaces/msg/PoseCommand` | Move the end effector to a pose |
| `/joint_command` | `std_msgs/msg/Float64MultiArray` | Move the arm to six joint values |
| `/open_gripper` | `example_interfaces/msg/Bool` | `true` opens the gripper, `false` closes it |

## `PoseCommand` message

| Field | Meaning |
|-------|---------|
| `x`, `y`, `z` | Target position of the end effector |
| `roll`, `pitch`, `yaw` | Target orientation in radians |
| `cartesian_path` | `false`: free planned move. `true`: straight-line move |

To see the exact field types:

```bash
ros2 interface show my_robot_interfaces/msg/PoseCommand
```

### Planned move vs straight-line move

- **`cartesian_path: false`** lets MoveIt choose any collision-free path to the target. Use it for large moves, such as going to a start position.
- **`cartesian_path: true`** keeps the end effector on a straight line. Use it for short, precise moves, such as descending onto an object or sliding sideways.

## Examples

### Pose commands

```bash
# Planned move
ros2 topic pub -1 /pose_command my_robot_interfaces/msg/PoseCommand "{x: 0.7, y: 0.0, z: 0.4, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: false}"

# Straight-line move down
ros2 topic pub -1 /pose_command my_robot_interfaces/msg/PoseCommand "{x: 0.7, y: 0.0, z: 0.2, roll: 3.14, pitch: 0.0, yaw: 0.0, cartesian_path: true}"
```

### Joint command

```bash
ros2 topic pub -1 /joint_command std_msgs/msg/Float64MultiArray "{data: [0.0, 0.0, 0.0, 0.0, 0.0, 0.0]}"
```

Use values that suit your arm's joint limits. The six numbers are in the order `joint1` to `joint6`.

### Gripper

```bash
ros2 topic pub --once /open_gripper example_interfaces/msg/Bool "{data: false}"   # close
ros2 topic pub --once /open_gripper example_interfaces/msg/Bool "{data: true}"    # open
```

## A full sequence

| Step | Command | Effect |
|------|---------|--------|
| 1 | `/pose_command` to (0.7, 0.0, 0.4), `cartesian_path: false` | Approach above the target |
| 2 | `/pose_command` to (0.7, 0.0, 0.2), `cartesian_path: true` | Descend in a straight line |
| 3 | `/open_gripper` with `false` | Close the gripper |
| 4 | `/pose_command` to (0.7, 0.3, 0.2), `cartesian_path: true` | Move sideways in a straight line |
| 5 | `/open_gripper` with `true` | Open the gripper |

Wait for each motion to finish before sending the next command.
