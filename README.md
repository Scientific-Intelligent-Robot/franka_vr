

# Franka VR Teleoperation System

VR teleoperation system for Franka FR3 robot based on ROS 2, MoveIt Servo, and Oculus Reader

![Franka VR Demo](https://github.com/user-attachments/assets/ff2d912c-bed8-414c-b882-5f3fe7406fdc)

## Project Overview

This project implements a complete VR teleoperation system that allows users to control a Franka FR3 robotic arm in real-time using Meta Quest VR headset and controllers. The system integrates ROS 2, MoveIt Servo motion planning framework, and Oculus Reader VR data acquisition module to provide an intuitive and smooth robot teleoperation experience.

### Key Features

1. **Real-time VR Control**: 6-DOF end-effector position and orientation control via Meta Quest controllers
2. **Low-Latency Communication**: Real-time communication architecture based on ROS 2 ensures low-latency command transmission
3. **Intelligent Motion Planning**: Integrated MoveIt Servo for online trajectory interpolation and singularity handling
4. **Gripper Control**: Support for controlling Franka gripper opening/closing via VR controller trigger buttons
5. **Collision Detection**: Real-time collision detection and avoidance functionality ensures operation safety
6. **Configurable Parameters**: Support for runtime adjustment of control parameters and coordinate transformations

### System Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Meta Quest    │    │   ROS 2 Nodes   │    │   Franka FR3    │
│  VR Controller  │◄──►│                 │◄──►│  Robotic Arm    │
│                 │    │  ┌─────────────┐│    │                 │
└─────────────────┘    │  │Oculus Reader ││    └─────────────────┘
                       │  │    Node      ││
                       │  └─────────────┘│
                       │  ┌─────────────┐│
                       │  │MoveIt Servo ││
                       │  │    Node     ││
                       │  └─────────────┘│
                       │  ┌─────────────┐│
                       │  │Franka VR    ││
                       │  │    Node     ││
                       │  └─────────────┘│
                       └─────────────────┘
```

### Main Components

- **Oculus Reader**: Responsible for reading controller position, orientation, and button states from Meta Quest device
- **Franka VR Node**: Core control node that processes VR data and converts it to robot control commands
- **MoveIt Servo**: Provides real-time motion planning and singularity handling
- **Franka Controller**: Low-level joint controller that executes specific motion commands


## Installation Guide

### System Requirements

- **Operating System**: Ubuntu 20.04/22.04/24.04 LTS
- **Real-time Kernel**: Recommended for better control performance
- **Docker**: Used to run Franka ROS 2 environment
- **Meta Quest**: Quest 2/3/Pro VR headset and controllers

### Prerequisites

1. **Real-time Kernel Installation**:
   - If your system doesn't have a real-time kernel installed, please follow the official Franka tutorial for installation and testing
   - Reference: https://github.com/frankarobotics/franka_ros2?tab=readme-ov-file#docker-container-installation

2. **Docker Environment**:
   - Install the official franka_ros2 Docker environment
   - Reference: https://github.com/frankarobotics/franka_ros2
   - It's recommended to switch git commit before 'Build the workspace' to match the VR code
```
git fetch --all
git checkout 1530569
```

3. **ADB Tools**:
   ```bash
   # For communication with Meta Quest device
   sudo apt install android-tools-adb
   ```

### Workspace Setup

1. **Clone MoveIt2 Tutorials**:
   ```bash
   mkdir
   cd /ws_moveit/src
   git clone -b main https://github.com/moveit/moveit2_tutorials
   vcs import --recursive < moveit2_tutorials/moveit2_tutorials.repos
   ```
   
   > **⚠️ Important Note**: MoveIt 2 is continuously being updated. Ensure the MoveIt 2 packages pulled via `moveit2_tutorials.repos` are version `branch main` (tag `2.13.0`) to match the correct `moveit_servo` version. To avoid conflicts, remove any existing MoveIt installations:
   ```bash
   sudo apt remove ros-$ROS_DISTRO-moveit*
   ```

2. **Install Dependencies**:
   ```bash
   sudo apt update && rosdep install -r --from-paths . --ignore-src --rosdistro $ROS_DISTRO -y
   ```

3. **Build Workspace**:
   ```bash
   cd ..
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

4. **Add franka_vr Package**:
   ```bash
   cd src
   git clone https://github.com/ZorAttC/franka_vr.git
   cd ..
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

### Meta Quest Device Configuration
   
You can refer to this blog post for guidance: https://www.reddit.com/r/OculusQuest/comments/1bkixqx/adb_not_finding_quest_3/

1. **Enable Developer Mode**:
   - Open Settings in the Oculus mobile app
   - Select Device → More Settings → Developer Mode
   - Toggle Developer Mode on

2. **USB Debugging Setup**:
   - Connect Quest to computer using USB-C cable
   - Put on the headset and accept the prompts for "Allow USB debugging" and "Always allow from this computer"

3. **Verify Connection**:
   ```bash
   adb devices
   # Should display output similar to:
   # List of devices attached
   # ce0551e7                device
   ```

## Usage Guide

### Starting the System

1. **Launch Franka Robotic Arm**:
   Open a terminal inside the Docker container:
   ```bash
   source /ros2_ws/install/setup.bash  # franka_ros2 environment
   source /ws_moveit2/install/setup.bash  # MoveIt 2 environment
   ros2 launch franka_vr franka_twist.launch.py   namespace:=franka   gripper_namespace:=franka   robot_ip:=xxx.xx.xxx   use_fake_hardware:=False   arm_id:=fr3
   ```
   After launching, RViz will display the current pose of the Franka robotic arm.

2. **Launch VR Teleoperation**:
   Open a second terminal:
   ```bash
   source /ros2_ws/install/setup.bash  # franka_ros2 environment
   source /ws_moveit2/install/setup.bash  # MoveIt 2 environment
   sh /ws_moveit2/src/oculus_reader/start_vr.sh
   ```
   This step will display the left and right hand controller coordinate frames in RViz.

### Control Instructions

#### VR Controller Operations
- **Right Trigger (rightTrig)**: Hold to enable robotic arm motion control
- **Right Grip (rightGrip)**: 
  - Press (>0.6) to close gripper
  - Release (<0.4) to open gripper

#### Coordinate Systems
- **World Frame**: Robot base coordinate system
- **Oculus Frame**: VR device reference coordinate system
- **Controller Frame**: Real-time position and orientation of left and right hand controllers

### Parameter Configuration

#### Configuration Files
Main configuration files are located in the `franka_vr/config/` directory:
- `fr3_real_config.yaml`: MoveIt Servo configuration parameters
- `joint_limits.yaml`: Joint limit configuration
- `kinematics.yaml`: Kinematic parameter configuration

## Troubleshooting

### Common Issues

1. **Missing `osqp` Library**:
   ```bash
   # Install osqp version <= 0.6.0
   pip install osqp==0.6.0
   ```

2. **RViz Won't Start**:
   ```bash
   # Fix Docker display issues
   xhost +local:docker
   ```

3. **Meta Quest Connection Issues**:
   ```bash
   # Check ADB connection
   adb devices
   
   # Restart ADB service
   adb kill-server
   adb start-server
   
   # Check Quest IP address (network mode)
   adb shell ip route
   ```

### Performance Optimization

1. **Reduce Latency**:
   - Use wired connection instead of WiFi
   - Ensure real-time kernel is properly configured
   - Adjust `publish_period` parameter (default 0.005 seconds)

2. **Improve Stability**:
   - Regularly calibrate VR controllers
   - Ensure adequate lighting environment
   - Avoid blocking Quest cameras

3. **Memory Optimization**:
   ```bash
   # Monitor system resources
   htop
   
   # Clean ROS 2 logs
   ros2 log clean
   ```

## System Limitations

### Known Issues

1. **Planning Scene Issue**: `planning_scene` cannot retrieve `fr3_finger_joint1` (doesn't affect gripper functionality, but gripper state won't update in RViz)

2. **Singularity Behavior**: MoveIt Servo slows down when approaching singularities but doesn't avoid them (advanced solutions like DLS control can address this)

3. **Oculus Data Stability**: Oculus Reader data occasionally jumps; filtering has been applied but it's still unstable

4. **Joint Limits**: Joint limits are not enforced for the 7-DOF arm; implementation might reduce singularity occurrences

### Improvement Suggestions

1. **Singularity Avoidance**: Implement DLS (Damped Least Squares) method to compute Jacobian pseudo-inverse
2. **Joint Limits**: Add joint limit checking to reduce singularity occurrences
3. **Data Filtering**: Improve filtering algorithms for Oculus data
4. **Workspace Initialization**: Develop fast workspace initialization algorithm

## Configuration

### Custom Coordinate Transformations
Modify transformation parameters in `start_franka_vr.py`:
```python
# Adjust scaling factor
self.scale = 2.0  # default 1.5

# Adjust rotation angle
y_angle = np.deg2rad(45)  # default 60 degrees
```

### Gripper Control Parameters
Adjust gripper control thresholds:
```python
# Modify in start_franka_vr.py
if grip_value > 0.7:  # default 0.6
    # Close gripper
elif grip_value < 0.3:  # default 0.4
    # Open gripper
```

## Technical References

### Related Documentation
- [MoveIt Servo Tutorial](https://moveit.picknik.ai/main/doc/examples/realtime_servo/realtime_servo_tutorial.html)
- [MoveIt Tutorials](https://github.com/moveit/moveit_tutorials)
- [Oculus Reader](https://github.com/rail-berkeley/oculus_reader)
- [Franka ROS 2](https://github.com/frankaemika/franka_ros2)
- [Franka ROS 2 Documentation](https://frankaemika.github.io/docs/franka_ros2.html)
- [Franka Installation Guide](https://frankaemika.github.io/docs/installation_linux.html)

### Academic Citation
If you use this project in your research, please consider citing:
```bibtex
@misc{franka_vr_2024,
  title={Franka VR: VR-based Teleoperation System for Franka FR3 Robot},
  author={Your Name},
  year={2024},
  url={https://github.com/ZorAttC/franka_vr}
}
```

## Technical Deep Dive

### Singularity Handling

Singularities occur when the robotic arm reaches a pose where the end-effector loses one degree of freedom in Cartesian space, causing the Jacobian matrix to become singular. This produces infinite joint velocities when mapping from Cartesian space to joint space. In teleoperation, this manifests as:

1. **Sudden Jittering**: High-speed sudden jittering at specific positions
2. **Motion Stopping**: Arm stops at singular positions due to velocity limits and no longer follows commands
3. **Joint Flipping**: Shoulder singularities lead to multiple solutions, potentially causing sudden 180-degree rotations of shoulder or wrist joints

In teleoperation, the goal is to prevent the arm from entering singularities or approaching them at minimal speed. Near singularities, small end-effector movements can cause large joint changes, leading to poor motion quality or loss of control.

#### Singularity Avoidance Strategies

1. **Hardware Design**: Minimize singularities in the arm's workspace through careful kinematic design
2. **DLS Method**: Use Damped Least Squares (DLS) method to compute Jacobian pseudo-inverse. Switch to DLS control when Jacobian determinant is small, reducing joint velocities near singularities at the cost of end-effector accuracy
3. **Planning Avoidance**: Avoid regions prone to singularities during trajectory planning
4. **Predictive Deceleration**: Use Jacobian norm or determinant as criteria. Predict arm direction using recent end-effector position increments, decelerate if criteria worsen, preventing entry into singular regions (this sacrifices tracking accuracy)

For teleoperation, the most elegant solution is option 2 (DLS), balancing responsiveness, safety (avoiding excessive joint velocities/accelerations), and slight accuracy tradeoffs. The current MoveIt Servo implementation is similar to option 4, decelerating near singularities to maintain control.

### Workspace Initialization

The arm's workspace often doesn't match the operator's workspace, especially for non-humanoid designs. Direct Euclidean mapping limits flexibility as humans cannot reach many positions the arm's end-effector can. Proper alignment and initialization between the arm's workspace and the operator's comfort range is crucial. This involves:

- **Setting up the VR device's base coordinate system**
- **Adjusting pose scaling and offsets**, typically fine-tuned through experimentation

For non-laboratory commercial scenarios (e.g., data collection centers), fast workspace initialization algorithms are needed. A preliminary idea is to provide online parameter adjustment controllers (scaling and offset). While slightly risky and suitable for experienced operators, this allows users to adjust parameters during specific tasks and save them for future use.

### System Performance Analysis

#### Latency Analysis
- **VR Data Acquisition**: ~14ms (70Hz)
- **ROS 2 Communication**: ~1-2ms
- **MoveIt Servo Processing**: ~5ms
- **Franka Controller**: ~1ms
- **Total Latency**: ~20-25ms

#### Bandwidth Requirements
- **VR Data Transfer**: ~1MB/s
- **ROS 2 Topics**: ~100KB/s
- **Total Bandwidth**: ~1.1MB/s

## Contributing

### Development Environment Setup
1. Fork this project
2. Create feature branch: `git checkout -b feature/new-feature`
3. Commit changes: `git commit -am 'Add new feature'`
4. Push branch: `git push origin feature/new-feature`
5. Create Pull Request

### Coding Standards
- Follow ROS 2 C++ and Python coding standards
- Add appropriate comments and documentation
- Ensure code passes all tests

### Reporting Issues
Use GitHub Issues to report bugs or request features. Please include:
- Operating system and ROS 2 version
- Detailed error messages
- Steps to reproduce
- Relevant log files

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](LICENSE) file for details.

## Acknowledgments

- Franka Emika GmbH - For providing Franka robotic arm and ROS 2 support
- MoveIt Team - For providing MoveIt Servo framework
- RAIL Berkeley - For providing Oculus Reader tool
- ROS 2 Community - For providing robust Robot Operating System framework

