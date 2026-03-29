# 🤖 mujoco_ros2_control + MoveIt 2

Simulate and motion-plan for ROS 2 robots using **MuJoCo** as the physics backend, **mujoco_ros2_control** as the hardware interface, and **MoveIt 2** for motion planning.

---

## 📋 Overview

This repository provides a complete integration stack that lets you:

- Run ROS 2 `mujoco_ros2_control` controllers inside a MuJoCo physics simulation.
- Use MoveIt 2 for collision-aware motion planning and execution.
- Swap in real hardware with zero changes to your planning stack (same URDF / `mujoco_ros2_control` tags).

**Tested on:** Ubuntu 24.04 · ROS 2 Jazzy · MuJoCo 3.x

---

## 🗂️ Repository Structure

```
.
├── config/
│   ├── controllers.yaml          # ros2_control controller configuration
│   ├── moveit/                   # MoveIt SRDF, kinematics, planning pipeline configs
│   └── mujoco/                   # MuJoCo scene XML files
├── description/
│   ├── urdf/                     # Robot URDF / xacro files
│   └── meshes/                   # Visual and collision meshes
├── launch/
│   ├── sim.launch.py             # Start MuJoCo simulation + controllers
│   └── moveit.launch.py          # Start MoveIt 2 move_group node
├── src/                          # Optional custom nodes / plugins
└── README.md
```

---

## 🔧 Prerequisites

| Dependency | Version | Install |
|---|---|---|
| ROS 2 | Jazzy (or later) | [docs.ros.org](https://docs.ros.org) |
| MuJoCo | 3.x | [mujoco.org](https://mujoco.org) |
| mujoco_ros2_control | latest | see below |
| MoveIt 2 | Jazzy | see below |

### System dependencies

```bash
sudo apt update && sudo apt install -y \
  python3-colcon-common-extensions \
  python3-rosdep \
  ros-jazzy-ros2-control \
  ros-jazzy-ros2-controllers \
  ros-jazzy-moveit \
  ros-jazzy-moveit-ros-planning-interface
```

---

## 🚀 Installation

### 1 — Create a workspace and clone

```bash
mkdir -p ~/ros2_ws/src && cd ~/ros2_ws/src

# Clone this repository
git clone https://github.com/nithishreddy1101/mujoco_ros2_control_ur5.git

# Clone mujoco_ros2_control (if not already a vendored dependency)
https://github.com/ros-controls/mujoco_ros2_control
```

### 2 — Install ROS dependencies

```bash
cd ~/ros2_ws
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```

### 3 — Build

```bash
cd ~/ros2_ws
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=RelWithDebInfo
```

### 4 — Source the workspace

```bash
source ~/ros2_ws/install/setup.bash

# Add to ~/.bashrc to avoid repeating this step
echo "source ~/ros2_ws/install/setup.bash" >> ~/.bashrc
```

---

## ▶️ Usage

### Launch the MuJoCo simulation + controllers

```bash
ros2 launch ur5_moveit_config demo.launch.py
```

This starts:
- The MuJoCo physics engine with your robot scene
- `robot_state_publisher`
- `controller_manager` with `mujoco_ros2_control` as the hardware interface
- Any controllers defined in `config/controllers.yaml` (e.g. `joint_trajectory_controller`)

### Launch MoveIt 2 (in a second terminal)

```bash
source ~/ros2_ws/install/setup.bash
ros2 launch ur5_moveit_config  moveit.launch.py(no need for now)
```

This starts:
- `move_group` with OMPL / other configured planners
- RViz with the MoveIt Motion Planning plugin

### Send a planning goal from the command line

```bash
ros2 run moveit2_tutorials motion_planning_python_api
```

Or use the RViz **Motion Planning** panel to drag the interactive marker to a goal and click **Plan & Execute**.

---

## ⚙️ Configuration

### Controllers (`config/controllers.yaml`)

```yaml
# This config file is used by ros2_control
controller_manager:
  ros__parameters:
    update_rate: 100  # Hz

    ur5_arm_controller:
      type: joint_trajectory_controller/JointTrajectoryController


    hand_controller:
      type: position_controllers/GripperActionController


    joint_state_broadcaster:
      type: joint_state_broadcaster/JointStateBroadcaster

ur5_arm_controller:
  ros__parameters:
    joints:
      - shoulder_pan_joint
      - shoulder_lift_joint
      - elbow_joint
      - wrist_1_joint
      - wrist_2_joint
      - wrist_3_joint
    command_interfaces:
      - position
    state_interfaces:
      - position
      - velocity
    allow_nonzero_velocity_at_trajectory_end: true
hand_controller:
  ros__parameters:
    joint: finger_joint
```

### MuJoCo scene (`config/mujoco/scene.xml`)

The scene XML references your robot MJCF model and any additional objects (ground plane, obstacles, cameras). Point the launch file to a different scene file to switch environments without recompiling.

### MoveIt SRDF (`config/moveit/ur5_robot.srdf`)

Define planning groups, end-effectors, and virtual joints here. Regenerate with the MoveIt Setup Assistant if you change the kinematic chain.

---


---

## 🐛 Troubleshooting

**MuJoCo viewer does not open**
Make sure `DISPLAY` is set (or use `MUJOCO_GL=osmesa` for headless rendering):
```bash
export MUJOCO_GL=osmesa
```

**`controller_manager` reports hardware interface not found**
Verify your URDF `<ros2_control>` block references `mujoco_ros2_control/MujocoSystem` as the plugin and that the joint names match the MuJoCo model exactly.

**MoveIt cannot find IK solution**
Check `config/moveit/kinematics.yaml` — ensure the solver (e.g. `KDLKinematicsPlugin`) and the planning group tip link are correct.

---

## 🤝 Contributing

Pull requests are welcome! Please open an issue first to discuss major changes.

1. Fork the repository
2. Create a feature branch (`git checkout -b feat/my-feature`)
3. Commit your changes (`git commit -m 'feat: add my feature'`)
4. Push to the branch (`git push origin feat/my-feature`)
5. Open a Pull Request

---

## 🙏 Acknowledgements

- [mujoco_ros2_control](https://github.com/ros-controls/mujoco_ros2_control) — MuJoCo hardware interface for ros2_control
- [MoveIt 2](https://moveit.ros.org/) — Motion planning framework
- [MuJoCo](https://mujoco.org/) — Physics simulation engine by DeepMind
