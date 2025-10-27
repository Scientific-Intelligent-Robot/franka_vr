

# Franka VR 遥操作系统

基于ROS 2、MoveIt Servo和Oculus Reader的Franka FR3机器人VR遥操作系统

![Franka VR Demo](https://github.com/user-attachments/assets/ff2d912c-bed8-414c-b882-5f3fe7406fdc)

## 项目概述

本项目实现了一个完整的VR遥操作系统，允许用户通过Meta Quest VR头显和控制器来实时控制Franka FR3机械臂。系统集成了ROS 2、MoveIt Servo运动规划框架和Oculus Reader VR数据读取模块，提供了直观、流畅的机器人遥操作体验。

### 核心特性

1. **实时VR控制**: 通过Meta Quest控制器实现6自由度末端执行器位置和姿态控制
2. **低延迟通信**: 基于ROS 2的实时通信架构，确保控制指令的低延迟传输
3. **智能运动规划**: 集成MoveIt Servo进行在线轨迹插值和奇异性处理
4. **夹爪控制**: 支持通过VR控制器触发按钮控制Franka夹爪的开合
5. **碰撞检测**: 实时碰撞检测和避障功能，确保操作安全
6. **可配置参数**: 支持运行时调整控制参数和坐标变换

### 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Meta Quest    │    │   ROS 2 节点    │    │   Franka FR3    │
│   VR 控制器     │◄──►│                 │◄──►│   机械臂        │
│                 │    │  ┌─────────────┐│    │                 │
└─────────────────┘    │  │Oculus Reader ││    └─────────────────┘
                       │  │   节点       ││
                       │  └─────────────┘│
                       │  ┌─────────────┐│
                       │  │MoveIt Servo ││
                       │  │   节点      ││
                       │  └─────────────┘│
                       │  ┌─────────────┐│
                       │  │Franka VR    ││
                       │  │   节点      ││
                       │  └─────────────┘│
                       └─────────────────┘
```

### 主要组件

- **Oculus Reader**: 负责从Meta Quest设备读取控制器位置、姿态和按钮状态
- **Franka VR Node**: 核心控制节点，处理VR数据并转换为机器人控制指令
- **MoveIt Servo**: 提供实时运动规划和奇异性处理
- **Franka Controller**: 底层关节控制器，执行具体的运动指令


## 安装指南

### 系统要求

- **操作系统**: Ubuntu 20.04/22.04/24.04 LTS
- **实时内核**: 推荐安装实时内核以获得更好的控制性能
- **Docker**: 用于运行Franka ROS 2环境
- **Meta Quest**: Quest 2/3/Pro VR头显及控制器

### 前置条件

1. **实时内核安装**:
   - 如果系统尚未安装实时内核，请按照Franka官方教程安装和测试
   - 参考: https://github.com/frankarobotics/franka_ros2?tab=readme-ov-file#docker-container-installation

2. **Docker环境**:
   - 安装官方franka_ros2 Docker环境
   - 参考: https://github.com/frankarobotics/franka_ros2
   - 建议在 'Build the workspace' 前切换 git 提交，以适配后面vr代码
```
git fetch --all
git checkout 1530569
```

3. **ADB工具**:
   ```bash
   # 用于与Meta Quest设备通信
   sudo apt install android-tools-adb
   ```

### 工作空间设置

1. **克隆MoveIt2教程**:
   ```bash
   mkdir
   cd /ws_moveit/src
   git clone -b main https://github.com/moveit/moveit2_tutorials
   vcs import --recursive < moveit2_tutorials/moveit2_tutorials.repos
   ```
   
   > **⚠️ 重要提示**: MoveIt 2持续更新中。确保通过`moveit2_tutorials.repos`拉取的MoveIt 2包版本为`branch main` (tag `2.13.0`)，以匹配正确的`moveit_servo`版本。为避免冲突，请移除现有的MoveIt安装：
   ```bash
   sudo apt remove ros-$ROS_DISTRO-moveit*
   ```

2. **安装依赖**:
   ```bash
   sudo apt update && rosdep install -r --from-paths . --ignore-src --rosdistro $ROS_DISTRO -y
   ```

3. **编译工作空间**:
   ```bash
   cd ..
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

4. **添加franka_vr包**:
   ```bash
   cd src
   git clone https://github.com/ZorAttC/franka_vr.git
   cd ..
   colcon build --cmake-args -DCMAKE_BUILD_TYPE=Release
   ```

### Meta Quest设备配置
   
可以参考这个博客中的回答：https://www.reddit.com/r/OculusQuest/comments/1bkixqx/adb_not_finding_quest_3/

1. **启用开发者模式**:
   - 在Oculus手机应用中进入设置
   - 选择设备 → 更多设置 → 开发者模式
   - 开启开发者模式开关

2. **USB调试设置**:
   - 使用USB-C线连接Quest到电脑
   - 戴上头显并接受"允许USB调试"和"始终允许来自此计算机"的提示

3. **验证连接**:
   ```bash
   adb devices
   # 应该显示类似输出:
   # List of devices attached
   # ce0551e7                device
   ```

## 运行指南

### 启动系统

1. **启动Franka机械臂**:
   在Docker容器内打开终端：
   ```bash
   source /ros2_ws/install/setup.bash  # franka_ros2环境
   source /ws_moveit2/install/setup.bash  # MoveIt 2环境
   ros2 launch franka_vr franka_twist.launch.py   namespace:=franka   gripper_namespace:=franka   robot_ip:=xxx.xx.xxx   use_fake_hardware:=False   arm_id:=fr3
   ```
   启动后，RViz将显示Franka机械臂的当前姿态。

2. **启动VR遥操作**:
   打开第二个终端：
   ```bash
   source /ros2_ws/install/setup.bash  # franka_ros2环境
   source /ws_moveit2/install/setup.bash  # MoveIt 2环境
   sh /ws_moveit2/src/oculus_reader/start_vr.sh
   ```
   此步骤将在RViz中显示左右手控制器的坐标框架。

### 控制说明

#### VR控制器操作
- **右扳机 (rightTrig)**: 按住启用机械臂运动控制
- **右握把 (rightGrip)**: 
  - 按下 (>0.6) 关闭夹爪
  - 松开 (<0.4) 打开夹爪

#### 坐标系统
- **世界坐标系**: 机械臂基座坐标系
- **Oculus坐标系**: VR设备的基准坐标系
- **控制器坐标系**: 左右手控制器的实时位置和姿态

### 参数配置

#### 配置文件
主要配置文件位于`franka_vr/config/`目录：
- `fr3_real_config.yaml`: MoveIt Servo配置参数
- `joint_limits.yaml`: 关节限制配置
- `kinematics.yaml`: 运动学参数配置

## 故障排除

### 常见问题

1. **缺少`osqp`库**:
   ```bash
   # 安装osqp版本 <= 0.6.0
   pip install osqp==0.6.0
   ```

2. **RViz无法启动**:
   ```bash
   # 解决Docker显示问题
   xhost +local:docker
   ```

3. **Meta Quest连接问题**:
   ```bash
   # 检查ADB连接
   adb devices
   
   # 重启ADB服务
   adb kill-server
   adb start-server
   
   # 检查Quest IP地址（网络模式）
   adb shell ip route
   ```

### 性能优化

1. **降低延迟**:
   - 使用有线连接而非WiFi
   - 确保实时内核正确配置
   - 调整`publish_period`参数（默认0.005秒）

2. **提高稳定性**:
   - 定期校准VR控制器
   - 确保充足的光照环境
   - 避免遮挡Quest摄像头

3. **内存优化**:
   ```bash
   # 监控系统资源
   htop
   
   # 清理ROS 2日志
   ros2 log clean
   ```

## 系统限制

### 已知问题

1. **规划场景问题**: `planning_scene`无法检索`fr3_finger_joint1`（不影响夹爪功能，但RViz中的夹爪状态不会更新）

2. **奇异性行为**: MoveIt Servo在接近奇异性时会减速但不会避开它们（高级解决方案如DLS控制可以解决此问题）

3. **Oculus数据稳定性**: Oculus Reader的数据偶尔会跳跃；已应用过滤但仍不稳定

4. **关节限制**: 7自由度机械臂没有强制执行关节限制，如果实现可能会减少奇异性发生

### 改进建议

1. **奇异性避免**: 实现DLS（阻尼最小二乘）方法计算雅可比矩阵的伪逆
2. **关节限制**: 添加关节限制检查以减少奇异性发生
3. **数据过滤**: 改进Oculus数据的滤波算法
4. **工作空间初始化**: 开发快速工作空间初始化算法

## 配置

### 自定义坐标变换
修改`start_franka_vr.py`中的变换参数：
```python
# 调整缩放因子
self.scale = 2.0  # 默认1.5

# 调整旋转角度
y_angle = np.deg2rad(45)  # 默认60度
```

### 夹爪控制参数
调整夹爪控制阈值：
```python
# 在start_franka_vr.py中修改
if grip_value > 0.7:  # 默认0.6
    # 关闭夹爪
elif grip_value < 0.3:  # 默认0.4
    # 打开夹爪
```

## 技术参考

### 相关文档
- [MoveIt Servo教程](https://moveit.picknik.ai/main/doc/examples/realtime_servo/realtime_servo_tutorial.html)
- [MoveIt教程](https://github.com/moveit/moveit_tutorials)
- [Oculus Reader](https://github.com/rail-berkeley/oculus_reader)
- [Franka ROS 2](https://github.com/frankaemika/franka_ros2)
- [Franka ROS 2文档](https://frankaemika.github.io/docs/franka_ros2.html)
- [Franka安装指南](https://frankaemika.github.io/docs/installation_linux.html)

### 学术引用
如果您在研究中使用了本项目，请考虑引用：
```bibtex
@misc{franka_vr_2024,
  title={Franka VR: VR-based Teleoperation System for Franka FR3 Robot},
  author={Your Name},
  year={2024},
  url={https://github.com/ZorAttC/franka_vr}
}
```

## 技术深入

### 奇异性处理

奇异性发生在机械臂达到末端执行器在笛卡尔空间中失去一个自由度的姿态时，导致雅可比矩阵变为奇异。这会在从笛卡尔空间映射到关节空间时产生无限大的关节速度。在遥操作中，这表现为：

1. **突然抖动**: 在特定位置出现高速突然抖动
2. **运动停止**: 机械臂在奇异位置停止，由于速度限制不再跟随命令
3. **关节翻转**: 肩关节奇异性导致多解，可能引起肩部或腕部关节的突然180度旋转

在遥操作中，目标是防止机械臂进入奇异性或以最小速度接近它们。接近奇异性时，小的末端执行器运动可能导致大的关节变化，导致运动质量差或失去控制。

#### 奇异性避免策略

1. **硬件设计**: 通过仔细的运动学设计最小化机械臂工作空间中的奇异性
2. **DLS方法**: 使用阻尼最小二乘(DLS)方法计算雅可比的伪逆。当雅可比行列式较小时切换到DLS控制，以末端执行器精度为代价减少接近奇异性时的关节速度
3. **规划避让**: 在轨迹规划期间避开容易产生奇异性的区域
4. **预测减速**: 使用雅可比的范数或行列式作为准则。使用最近的末端执行器位置增量预测机械臂方向，如果准则恶化则减速，防止进入奇异区域（这会牺牲跟踪精度）

对于遥操作，最优雅的解决方案是选项2（DLS），平衡响应性、安全性（避免过度的关节速度/加速度）和轻微的精度权衡。MoveIt Servo的当前实现类似于选项4，在接近奇异性时减速以保持控制。

### 工作空间初始化

机械臂的工作空间经常与操作员的工作空间不匹配，特别是对于非人形设计。直接的欧几里得映射限制了灵活性，因为人类无法到达机械臂末端执行器的许多位置。机械臂工作空间与操作员舒适范围之间的适当对齐和初始化至关重要。这涉及：

- **设置VR设备的基坐标系**
- **调整姿态缩放和偏移**，通常通过实验进行微调

对于非实验室商业场景（例如，数据收集中心），需要快速的工作空间初始化算法。初步想法是提供在线参数调整控制器（缩放和偏移）。虽然略有风险且适合有经验的操作员，但这允许用户在特定任务期间调整参数并保存以供将来使用。

### 系统性能分析

#### 延迟分析
- **VR数据采集**: ~14ms (70Hz)
- **ROS 2通信**: ~1-2ms
- **MoveIt Servo处理**: ~5ms
- **Franka控制器**: ~1ms
- **总延迟**: ~20-25ms

#### 带宽需求
- **VR数据传输**: ~1MB/s
- **ROS 2话题**: ~100KB/s
- **总带宽**: ~1.1MB/s

## 贡献指南

### 开发环境设置
1. Fork本项目
2. 创建功能分支: `git checkout -b feature/new-feature`
3. 提交更改: `git commit -am 'Add new feature'`
4. 推送分支: `git push origin feature/new-feature`
5. 创建Pull Request

### 代码规范
- 遵循ROS 2 C++和Python编码标准
- 添加适当的注释和文档
- 确保代码通过所有测试

### 报告问题
使用GitHub Issues报告bug或提出功能请求，请包含：
- 操作系统和ROS 2版本
- 详细的错误信息
- 复现步骤
- 相关日志文件

## 许可证

本项目采用Apache License 2.0许可证。详情请参阅[LICENSE](LICENSE)文件。

## 致谢

- Franka Emika GmbH - 提供Franka机械臂和ROS 2支持
- MoveIt团队 - 提供MoveIt Servo框架
- RAIL Berkeley - 提供Oculus Reader工具
- ROS 2社区 - 提供强大的机器人操作系统框架
