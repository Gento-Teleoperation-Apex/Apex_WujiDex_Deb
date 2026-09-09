# Apex_WujiDex_Deb

Debian packages of the **WujiHand ROS 2 Humble** driver stack for Apex / teleoperation deployments.

This repository distributes pre-built `.deb` installers so you can install the driver without building from source.

| Item | Value |
|------|--------|
| Package name | `ros-humble-wujihand` |
| Version | `1.0.1-1` |
| ROS distro | Humble (Ubuntu 22.04) |
| Install prefix | `/opt/ros/humble` |
| Upstream stack | `wujihand_msgs`, `wujihand_driver`, `wujihand_bringup`, `wuji_hand_description` |

---

## Packages

| File | Architecture | Target platforms |
|------|--------------|------------------|
| [`ros-humble-wujihand_1.0.1-1_amd64.deb`](./ros-humble-wujihand_1.0.1-1_amd64.deb) | `amd64` | x86_64 PCs / workstations |
| [`ros-humble-wujihand_1.0.1-1_arm64.deb`](./ros-humble-wujihand_1.0.1-1_arm64.deb) | `arm64` | NVIDIA Jetson / aarch64 |

Pick the file that matches `dpkg --print-architecture` on your machine.

---

## Prerequisites

1. **Ubuntu 22.04** with **ROS 2 Humble** installed.
2. **WujiHand C++ SDK** (`wujihandcpp` ≥ 1.4.0).
3. WujiHand connected over **USB** (device appears as `WUJIHAND` in `lsusb`).

```bash
# Check ROS
source /opt/ros/humble/setup.bash
ros2 --version

# Check architecture
dpkg --print-architecture

# Check hand USB presence
lsusb | grep -i wuji
```

---

## Installation

```bash
# 1) Install the C++ SDK dependency (from your Wuji / Apex package source)
sudo apt update
sudo apt install wujihandcpp

# 2) Download the matching .deb from this repository, then install
cd /path/to/Apex_WujiDex_Deb
sudo dpkg -i ros-humble-wujihand_1.0.1-1_$(dpkg --print-architecture).deb

# 3) Fix missing dependencies if dpkg reports broken packages
sudo apt -f install
```

Verify:

```bash
dpkg -l | grep ros-humble-wujihand
ls /opt/ros/humble/share/wujihand_bringup/launch/wujihand.launch.py
```

---

## Quick Start

```bash
source /opt/ros/humble/setup.bash

# Read SN from:
#   ls /dev/serial/by-id/

# Launch one hand (right example)
ros2 launch wujihand_bringup wujihand.launch.py \
  hand_name:=hand_right \
  serial_number:=YOUR_SERIAL_NUMBER
```

Check joint state:

```bash
ros2 topic echo /hand_right/joint_states --once
```

### Dual-hand setup

Run **two launch processes** in separate terminals (each owns one USB device):

```bash
# Terminal A — left hand
source /opt/ros/humble/setup.bash
ros2 launch wujihand_bringup wujihand.launch.py \
  hand_name:=hand_left \
  serial_number:=LEFT_SERIAL_NUMBER

# Terminal B — right hand
source /opt/ros/humble/setup.bash
ros2 launch wujihand_bringup wujihand.launch.py \
  hand_name:=hand_right \
  serial_number:=RIGHT_SERIAL_NUMBER
```

---

## Launch Arguments

| Argument | Default | Description |
|----------|---------|-------------|
| `hand_name` | `hand_0` | ROS namespace / TF prefix (`hand_left`, `hand_right`, …) |
| `serial_number` | `""` | USB serial of the target WujiHand (required when multiple hands are connected) |
| `publish_rate` | `1000.0` | Joint-state publish rate (Hz) |
| `filter_cutoff_freq` | *(package default)* | Low-pass filter cutoff (Hz) |
| `diagnostics_rate` | *(package default)* | Diagnostics publish rate (Hz) |
| `rviz` | `false` | Launch RViz visualization |
| `foxglove` | `false` | Launch Foxglove Bridge |

Examples:

```bash
# With RViz
ros2 launch wujihand_bringup wujihand.launch.py \
  hand_name:=hand_right serial_number:=YOUR_SN rviz:=true

# Custom publish rate
ros2 launch wujihand_bringup wujihand.launch.py \
  hand_name:=hand_right serial_number:=YOUR_SN publish_rate:=500.0
```

---

## Main ROS Interfaces

For `hand_name:=hand_right` (replace with your namespace):

| Topic / interface | Type | Direction | Notes |
|-------------------|------|-----------|-------|
| `/hand_right/joint_states` | `sensor_msgs/JointState` | output | Measured joint positions |
| `/hand_right/joint_commands` | `sensor_msgs/JointState` | input | Desired joint positions (20 DoF) |
| `/hand_right/hand_diagnostics` | custom | output | Driver diagnostics |
| `/control/footkey` | `std_msgs/Bool` | input | When used with Apex teleop stacks: driver accepts commands only while `true` |

Find your serial number:

```bash
ls -l /dev/serial/by-id/
# Example:
# usb-WUJITECH_WUJIHAND_365939643134-if00 -> ../../ttyACM0
```

---

## Dependencies

Declared in the package control file:

- `ros-humble-ros-base`
- `ros-humble-sensor-msgs`
- `ros-humble-std-msgs`
- `ros-humble-robot-state-publisher`
- `wujihandcpp (>= 1.4.0)`

---

## Uninstall

```bash
sudo dpkg -r ros-humble-wujihand
# or purge config leftovers:
sudo dpkg -P ros-humble-wujihand
```

---

## Troubleshooting

| Symptom | What to check |
|---------|----------------|
| `dpkg: dependency problems` | Run `sudo apt -f install`; ensure `wujihandcpp` is installed |
| Hand not detected | Re-seat USB; confirm `lsusb` shows `WUJIHAND`; check cable supports data |
| Wrong hand moves | Pass the correct `serial_number`; do not omit SN when two hands are plugged in |
| Commands ignored | Some Apex setups require `/control/footkey` = `true` before accepting `joint_commands` |
| Launch hangs on Ctrl+C | Force-kill stuck launch: `pkill -9 -f 'wujihand.launch.py'` |

---

## Support

- Hardware / SDK docs: [Wuji Docs Center](https://docs.wuji.tech/)
- Contact: [support@wuji.tech](mailto:support@wuji.tech)
