# Apex_WujiDex_Deb

ROS 2 Humble Debian packages for WujiHand (`ros-humble-wujihand`).

## Packages

| File | Arch |
|------|------|
| `ros-humble-wujihand_1.0.1-1_amd64.deb` | amd64 |
| `ros-humble-wujihand_1.0.1-1_arm64.deb` | arm64 (Jetson / aarch64) |

## Install

```bash
# dependency
sudo apt install wujihandcpp

# pick the matching arch
sudo dpkg -i ros-humble-wujihand_1.0.1-1_$(dpkg --print-architecture).deb
sudo apt -f install   # if needed

source /opt/ros/humble/setup.bash
ros2 launch wujihand_bringup wujihand.launch.py hand_name:=hand_right serial_number:=YOUR_SN
```

## Depends

- `ros-humble-ros-base`
- `ros-humble-sensor-msgs`
- `ros-humble-std-msgs`
- `ros-humble-robot-state-publisher`
- `wujihandcpp (>= 1.4.0)`
