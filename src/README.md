# ROS 2 Jazzy Workspace: Fundamentals, myCobot Arm & Yahboom ROSMASTER

A hands-on ROS 2 workspace built while learning ROS 2 Jazzy. It starts with minimal publisher/subscriber nodes and builds up to simulating a robot arm (myCobot) and a mecanum-wheeled mobile robot (Yahboom ROSMASTER X3) in Gazebo.

**Environment:** Ubuntu 24.04 · ROS 2 Jazzy · Gazebo (`gz sim`) · C++17 and Python 3

---

## Table of Contents

1. [What's inside](#whats-inside)
2. [Prerequisites](#prerequisites)
3. [Setup and build](#setup-and-build)
4. [Tutorial 1: Publisher and subscriber basics](#tutorial-1-publisher-and-subscriber-basics)
5. [Tutorial 2: Simulate the myCobot arm](#tutorial-2-simulate-the-mycobot-arm)
6. [Tutorial 3: Simulate the Yahboom ROSMASTER X3](#tutorial-3-simulate-the-yahboom-rosmaster-x3)
7. [Workspace layout](#workspace-layout)
8. [Troubleshooting](#troubleshooting)
9. [Credits and license](#credits-and-license)

---

## What's inside

| Package group | What it teaches |
|---|---|
| `ros2_fundamentals_examples` | Nodes, topics, publishers and subscribers in **both C++ and Python** |
| `mycobot_ros2` | URDF description, Gazebo world, MoveIt 2 config, and an arm/gripper test node for the myCobot 280 arm |
| `yahboom_rosmaster` | Mecanum drive controller, Gazebo simulation, custom messages, and system tests (square path, timed rotation action) for the ROSMASTER X3 |

---

## Prerequisites

- Ubuntu 24.04 with [ROS 2 Jazzy](https://docs.ros.org/en/jazzy/Installation.html) installed
- `colcon` and `rosdep`:
  ```bash
  sudo apt update
  sudo apt install -y python3-colcon-common-extensions python3-rosdep git
  sudo rosdep init   # skip if already initialised
  rosdep update
  ```
- Git, with a GitHub account (see [GitHub's guide](https://docs.github.com/en/get-started/getting-started-with-git/set-up-git))

---

## Setup and build

```bash
# 1. Clone into your home folder
cd ~
git clone https://github.com/arjun4036/<your-repo-name>.git ros2_arjun
cd ~/ros2_arjun

# 2. Install dependencies declared in each package.xml
source /opt/ros/jazzy/setup.bash
rosdep install --from-paths src --ignore-src -r -y

# 3. Build
colcon build

# 4. Source the workspace (do this in every new terminal)
source ~/ros2_arjun/install/setup.bash
```

> **Important:** a few launch files read config files from `~/ros2_arjun/...`. Clone into a folder named `ros2_arjun` in your home directory (as above), or update those paths.

Tip: add the source lines to your `~/.bashrc` so every terminal is ready automatically:

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
echo "source ~/ros2_arjun/install/setup.bash" >> ~/.bashrc
```

---

## Tutorial 1: Publisher and subscriber basics

Package: `ros2_fundamentals_examples`

Open two terminals (workspace sourced in both).

**C++**
```bash
# Terminal 1
ros2 run ros2_fundamentals_examples minimal_cpp_publisher
# Terminal 2
ros2 run ros2_fundamentals_examples minimal_cpp_subscriber
```

**Python**
```bash
# Terminal 1
ros2 run ros2_fundamentals_examples py_minimal_publisher.py
# Terminal 2
ros2 run ros2_fundamentals_examples py_minimal_subscriber.py
```

Inspect what is happening:
```bash
ros2 node list
ros2 topic list
ros2 topic echo /<topic_name>
rqt_graph
```

**What you learn:** how a node is created, how to publish on a timer, how to subscribe with a callback, and how the same pattern looks in C++ and Python.

Run the unit tests:
```bash
colcon test --packages-select ros2_fundamentals_examples
colcon test-result --verbose
```

---

## Tutorial 2: Simulate the myCobot arm

Packages: `mycobot_description`, `mycobot_gazebo`, `mycobot_moveit_config`, `mycobot_system_tests`

Launch the arm in Gazebo with RViz:
```bash
ros2 launch mycobot_gazebo mycobot.gazebo.launch.py
```

Useful launch arguments (all optional):

| Argument | Default | Meaning |
|---|---|---|
| `use_rviz` | `true` | Open RViz |
| `use_gazebo` | `true` | Start the Gazebo simulation |
| `load_controllers` | `true` | Load ros2_control controllers |
| `use_camera` | `false` | Add the camera |
| `jsp_gui` | `false` | Joint State Publisher GUI |
| `world_file` | `pick_and_place_demo.world` | Choose from `empty.world`, `house.world`, `pick_and_place_demo.world` |
| `x`, `y`, `z`, `roll`, `pitch`, `yaw` | `0` | Spawn pose of the robot |

Example: use the house world without RViz:
```bash
ros2 launch mycobot_gazebo mycobot.gazebo.launch.py use_rviz:=false world_file:=house.world
```

Run the arm and gripper loop test while the simulation is running:
```bash
ros2 run mycobot_system_tests arm_gripper_loop_controller
```

---

## Tutorial 3: Simulate the Yahboom ROSMASTER X3

Packages: `yahboom_rosmaster_description`, `yahboom_rosmaster_gazebo`, `yahboom_rosmaster_bringup`, `mecanum_drive_controller`, `yahboom_rosmaster_msgs`, `yahboom_rosmaster_system_tests`

Launch the robot in Gazebo:
```bash
ros2 launch yahboom_rosmaster_gazebo yahboom_rosmaster.gazebo.launch.py
```

The default world is `empty.world`. Pick another with `world_file`:
```bash
ros2 launch yahboom_rosmaster_gazebo yahboom_rosmaster.gazebo.launch.py world_file:=cafe.world
```
Available worlds: `empty.world`, `house.world`, `cafe.world`, `pick_and_place_demo.world`.

Drive it manually from the keyboard:
```bash
sudo apt install -y ros-jazzy-teleop-twist-keyboard
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Run the system tests (with the simulation running):
```bash
ros2 run yahboom_rosmaster_system_tests square_mecanum_controller       # drive in a square
ros2 run yahboom_rosmaster_system_tests timed_rotation_action_server    # action server
ros2 run yahboom_rosmaster_system_tests timed_rotation_action_client    # action client
ros2 run yahboom_rosmaster_system_tests mecanum_parameters_node         # parameter demo
```

**What you learn:** how mecanum wheel kinematics produce sideways motion, how `ros2_control` loads a custom controller, and how ROS 2 actions work.

---

## Workspace layout

```
ros2_arjun/
├── .gitignore
└── src/
    ├── ros2_fundamentals_examples/   # pub/sub in C++ and Python
    ├── mycobot_ros2/
    │   ├── mycobot_description/      # URDF, meshes, RViz config
    │   ├── mycobot_gazebo/           # worlds, models, launch
    │   ├── mycobot_moveit_config/    # MoveIt 2 configuration
    │   ├── mycobot_bringup/
    │   └── mycobot_system_tests/     # arm/gripper test node
    └── yahboom_rosmaster/
        ├── yahboom_rosmaster_description/
        ├── yahboom_rosmaster_gazebo/
        ├── yahboom_rosmaster_bringup/
        ├── yahboom_rosmaster_msgs/
        ├── yahboom_rosmaster_system_tests/
        └── mecanum_drive_controller/
```

`build/`, `install/` and `log/` are generated by `colcon build` and are intentionally not tracked by git.

---

## Troubleshooting

| Problem | Fix |
|---|---|
| `Package '...' not found` | Run `source ~/ros2_arjun/install/setup.bash` in that terminal |
| Launch file cannot find `config/...yaml` | Make sure the workspace is at `~/ros2_arjun` and has been built |
| Missing dependency during build | Re-run `rosdep install --from-paths src --ignore-src -r -y` |
| Strange build errors after pulling changes | `rm -rf build install log && colcon build` |
| Gazebo opens but the robot does not move | Check controllers: `ros2 control list_controllers` |

---

## Credits and license

This project was built by following the ROS 2 tutorials from **Automatic Addison** (Addison Sears-Collins) on YouTube and his website, [automaticaddison.com](https://automaticaddison.com). Thanks to him for the excellent material. The robot descriptions and parts of the code build on open-source work by Elephant Robotics (myCobot), Yahboom, the ros2_controllers project, and AWS RoboMaker (Gazebo models).

Each package keeps its original license (Apache-2.0 or BSD-3-Clause); see the `LICENSE` file inside each package.

Maintained by **Arjun** ([@arjun4036](https://github.com/arjun4036)).
