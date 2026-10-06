# UR12e ROS 2 Manual

This repository contains instructions and scripts for controlling the **Universal Robots UR12e** using **ROS 2 Jazzy**.

## 1. Prerequisites

Make sure the following are available before starting:

- Ubuntu 24.04
- ROS 2 Jazzy
- A UR12e robotic arm
- Ethernet connection between the UR12e control box and the desktop
- Universal Robots ROS 2 driver

### Install ROS 2 Jazzy

If ROS 2 Jazzy is not already installed, install it following the official ROS 2 Jazzy installation instructions.

After installation, source ROS 2:

```bash
source /opt/ros/jazzy/setup.bash
```

You can verify that ROS 2 is available with:

```bash
ros2 --version
```

---

## 2. Install the Universal Robots ROS 2 Driver

Install the Universal Robots ROS 2 driver:

```bash
sudo apt update
sudo apt install ros-jazzy-ur
```

After installation, source the ROS 2 environment:

```bash
source /opt/ros/jazzy/setup.bash
```

Verify that the UR packages are installed:

```bash
ros2 pkg list | grep ur
```

You should see packages including:

```text
ur_robot_driver
ur_description
ur_bringup
ur_calibration
...
```

You can also check the controller-related packages:

```bash
ros2 pkg list | grep controller
```

You should see packages such as:

```text
controller_manager
controller_interface
joint_state_broadcaster
...
```

---

## 3. Network Connection

Connect the **UR12e control box directly to the desktop using an Ethernet cable**.

The robot used by this repository has the following IP address:

```text
192.168.50.10
```

Before starting ROS, verify that the desktop can communicate with the robot:

```bash
ping 192.168.50.10
```

You should receive replies similar to:

```text
64 bytes from 192.168.50.10: icmp_seq=1 ttl=64 time=...
64 bytes from 192.168.50.10: icmp_seq=2 ttl=64 time=...
```

If the ping fails, **do not start the ROS driver yet**. Check the Ethernet connection and the network configuration first.

---

# 4. Prepare the UR12e

Before starting the ROS driver:

1. Turn on the UR12e control box.
2. Turn on the teach pendant.
3. Wait for the robot to finish booting.
4. Make sure the robot is not in an emergency-stop state.
5. Release the emergency stop if necessary.
6. Make sure the robot is ready to receive external control commands.

The UR ROS 2 driver uses the **External Control** interface on the UR teach pendant.

---

# 5. Start External Control on the UR12e

On the teach pendant, open the robot program containing the **External Control** program node.

The exact menu layout may vary depending on the PolyScope version and installed URCap.

Start the External Control program and make sure the robot is ready for external control.

> **Important:** Do not attempt to move the robot until the ROS driver reports that the connection has been successfully established.

---

# 6. Start the ROS 2 UR Driver

Open a terminal and source ROS 2 Jazzy:

```bash
source /opt/ros/jazzy/setup.bash
```

Start the UR driver:

```bash
ros2 launch ur_robot_driver ur_control.launch.py \
    ur_type:=ur12e \
    robot_ip:=192.168.50.10 \
    launch_rviz:=true
```

### Headless mode

If RViz is not required, use:

```bash
ros2 launch ur_robot_driver ur_control.launch.py \
    ur_type:=ur12e \
    robot_ip:=192.168.50.10 \
    launch_rviz:=false
```

---

# 7. Verify the Connection

If the driver starts successfully, you should see ROS nodes and controller-related messages in the terminal.

When RViz is enabled, a GUI should appear showing the UR12e model.

The robot visualization should update according to the actual robot joint positions.

You can also check the ROS nodes:

```bash
ros2 node list
```

You should see UR-related nodes, including the driver/controller infrastructure.

Check the available topics:

```bash
ros2 topic list
```

In particular, you should see:

```text
/joint_states
```

You can inspect the current joint positions with:

```bash
ros2 topic echo /joint_states
```

The six UR12e joints should be listed:

```text
shoulder_pan_joint
shoulder_lift_joint
elbow_joint
wrist_1_joint
wrist_2_joint
wrist_3_joint
```

---

# 8. Check the Controllers

If the ROS 2 control CLI is available, run:

```bash
ros2 control list_controllers
```

The exact controller names may depend on the driver configuration, but you should see the joint-state broadcaster and a trajectory controller.

A typical configuration includes:

```text
joint_state_broadcaster
scaled_joint_trajectory_controller
```

The trajectory controller should be in the:

```text
active
```

state before attempting to send a trajectory.

---

# 9. Move the UR12e

**Be extremely careful when commanding the real robot.**

Before sending a motion command:

- Make sure the workspace is clear.
- Keep people away from the robot.
- Start with very small movements.
- Keep the robot within a safe configuration.
- Be ready to stop the robot using the teach pendant/emergency stop if necessary.

The recommended ROS interface for trajectory control is the controller's `FollowJointTrajectory` action.

First inspect the available actions:

```bash
ros2 action list
```

You should see an action similar to:

```text
/scaled_joint_trajectory_controller/follow_joint_trajectory
```

You can inspect it with:

```bash
ros2 action info \
    /scaled_joint_trajectory_controller/follow_joint_trajectory
```

---

## 10. Read the Current Joint Position

Before commanding a movement, read the current joint positions:

```bash
ros2 topic echo /joint_states
```

Record the current values of:

```text
shoulder_pan_joint
shoulder_lift_joint
elbow_joint
wrist_1_joint
wrist_2_joint
wrist_3_joint
```

Do **not** use arbitrary joint angles on the real robot.

A safe test should start from the robot's current configuration and make a very small change.

For example, changing one joint by a few degrees:

```text
5 degrees ≈ 0.0873 radians
```

---

# 11. ROS 2 Workspace

If this repository contains additional ROS 2 packages, create a workspace and build it:

```bash
cd ~/projects
mkdir -p ur_ws/src
cd ur_ws/src
```

Clone this repository into `src`, if necessary:

```bash
git clone <repository-url>
```

Then build:

```bash
cd ~/projects/ur_ws
source /opt/ros/jazzy/setup.bash
colcon build
```

After building:

```bash
source install/setup.bash
```

---

# 12. Recommended Terminal Setup

Every new terminal needs access to the ROS 2 environment.

For ROS 2 Jazzy:

```bash
source /opt/ros/jazzy/setup.bash
```

If you have a workspace:

```bash
source ~/projects/ur_ws/install/setup.bash
```

You can optionally add the ROS 2 setup command to your `~/.bashrc` so that it is automatically sourced whenever a new terminal is opened:

```bash
echo "source /opt/ros/jazzy/setup.bash" >> ~/.bashrc
```

Then reload:

```bash
source ~/.bashrc
```

After this, you should not need to manually source `/opt/ros/jazzy/setup.bash` every time you open a new terminal.

---

# 13. Basic Startup Procedure

For normal operation, the workflow is:

```text
1. Turn on UR12e
        ↓
2. Connect Ethernet
        ↓
3. Verify robot network connection
        ↓
4. Start External Control on teach pendant
        ↓
5. Source ROS 2 Jazzy
        ↓
6. Start ur_robot_driver
        ↓
7. Verify /joint_states
        ↓
8. Verify controllers
        ↓
9. Send trajectory / motion command
```

### Quick startup commands

```bash
source /opt/ros/jazzy/setup.bash

ping 192.168.50.10

ros2 launch ur_robot_driver ur_control.launch.py \
    ur_type:=ur12e \
    robot_ip:=192.168.50.10 \
    launch_rviz:=true
```

Then, in another terminal:

```bash
source /opt/ros/jazzy/setup.bash

ros2 topic echo /joint_states
```

---

# 14. Troubleshooting

### `Package 'ur_robot_driver' not found`

Make sure the UR driver is installed:

```bash
sudo apt update
sudo apt install ros-jazzy-ur
```

Then:

```bash
source /opt/ros/jazzy/setup.bash
```

Verify:

```bash
ros2 pkg list | grep ur_robot_driver
```

---

### `ros2 control` is not recognized

If you see:

```text
invalid choice: 'control'
```

the ROS 2 control CLI extension is not available in the current environment.

This does **not necessarily mean that ros2_control itself is missing**.

You can still inspect the controllers through the driver output and ROS topics/actions.

---

### Cannot connect to the robot

First check:

```bash
ping 192.168.50.10
```

If this fails:

1. Check the Ethernet cable.
2. Check the desktop Ethernet interface configuration.
3. Check the robot IP address.
4. Check that the control box is powered on.
5. Check that the robot is not disconnected from the network.

---

### RViz opens but the robot does not move

Check:

```bash
ros2 topic echo /joint_states
```

If joint states are updating, the ROS driver is communicating with the robot.

Then check whether the trajectory controller is available:

```bash
ros2 action list
```

Look for:

```text
/scaled_joint_trajectory_controller/follow_joint_trajectory
```

Also verify that the **External Control** program is running on the teach pendant.

---

# 15. Safety

This repository is intended for controlling a **physical UR12e robot**.

Always test with conservative trajectories first.

Before executing a trajectory:

- Verify the robot's current configuration.
- Verify the target joint positions.
- Verify the trajectory duration.
- Ensure the workspace is clear.
- Keep a hand near the emergency stop.
- Never execute an unverified trajectory on the real robot.

**Never assume that a trajectory that is safe in simulation is automatically safe on the physical robot.**