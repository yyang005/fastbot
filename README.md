# FastBot Software Installation Guide

This guide explains how to install and run the FastBot ROS 2 software on an existing FastBot robot.

## Prerequisites

The FastBot robot should already include:

* Raspberry Pi with Ubuntu 22.04
* ROS 2 Humble
* Arduino Nano motor controller
* LSLiDAR N10
* Raspberry Pi Camera Module 2

---

# 1. Connect to the Robot

SSH into the Raspberry Pi:

```bash
ssh <username>@<robot_ip_address>
```

Example:

```bash
ssh ubuntu@192.168.1.100
```

---

# 2. Create a ROS 2 Workspace

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
```

---

# 3. Download the FastBot Repository

```bash
git clone https://github.com/yyang005/fastbot.git
```

---

# 4. Install Dependencies

Update package information:

```bash
sudo apt update
```

Install packages required by FastBot:

```bash
sudo apt install \
    ros-humble-v4l2-camera \
    ros-humble-image-transport-plugins v4l-utils
```

---

# 5. Build the Workspace

```bash
cd ~/ros2_ws

source /opt/ros/humble/setup.bash

colcon build --symlink-install
```

After a successful build:

```bash
source install/setup.bash
```

---

# 6. Launch FastBot

## Launch Parameters

The FastBot bringup launch file accepts the following parameters:

| Parameter    | Default Value     | Description                                                                             |
| ------------ | ----------------- | --------------------------------------------------------------------------------------- |
| use_sim_time | false             | Use simulation clock instead of system clock. Set to `true` when running in simulation. |
| robot_name   | fastbot           | Robot namespace and frame prefix.                                                       |
| serial_port  | /dev/arduino_nano | Serial port connected to the Arduino motor controller.                                  |
| baud_rate    | 57600             | Baud rate used for communication with the motor controller.                             |
| loop_rate    | 30                | Motor controller update rate (Hz).                                                      |
| encoder_cpr  | 2550              | Encoder counts per wheel revolution.                                                    |

### Default Launch

```bash
ros2 launch fastbot_bringup bringup.launch.xml
```

### Example with Custom Parameters

```bash
ros2 launch fastbot_bringup bringup.launch.xml \
    robot_name:=fastbot01 \
    serial_port:=/dev/ttyUSB0 \
    baud_rate:=57600 \
    loop_rate:=30 \
    encoder_cpr:=2550
```

> **Important:** Make sure the parameter values match the hardware configuration of the robot being used. In particular, verify:
>
> * `serial_port` points to the correct Arduino device.
> * `baud_rate` matches the Arduino firmware settings.
> * `encoder_cpr` matches the wheel encoder installed on the robot.
> * `robot_name` is unique if multiple robots are operating on the same network.

This launch file starts:

* Serial motor driver
* LSLiDAR N10 driver
* Raspberry Pi camera driver

---

# 7. Verify the System

Check that the nodes are running:

```bash
ros2 node list
```

Expected nodes:

```text
/serial_motor
/lslidar_driver
/fastbot/v4l2_camera_node
```

Check available topics:

```bash
ros2 topic list
```

Important topics include:

```text
/scan
/image_raw
/cmd_vel
/odom
/tf
/tf_static
```

---

# 8. Test Robot Motion

Open a new terminal:

```bash
source ~/ros2_ws/install/setup.bash
```

Run keyboard teleoperation:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard
```

Use the keyboard commands displayed on screen to drive the robot.

---

# 9. Visualize Sensor Data

## LiDAR

On a computer connected to the same ROS network:

```bash
rviz2
```

Add a LaserScan display and select:

```text
/scan
```

## Camera

Run:

```bash
rqt_image_view
```

Select:

```text
/fastbot/image_raw
```
