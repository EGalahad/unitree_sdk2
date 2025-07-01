# G1 Interface Python Binding

这是一个基于 pybind11 的 Unitree G1 机器人 Python 接口，提供了低级别的机器人控制功能。

## 功能特性

- **实时控制**: 支持 500Hz 的实时控制循环
- **数据读取**: 读取机器人状态（IMU、电机状态等）和无线控制器输入
- **命令发送**: 发送电机控制命令到机器人
- **双控制模式**: 支持 PR (Pitch/Roll) 和 AB (A/B) 控制模式
- **类型安全**: 提供完整的 Python 类型提示 (.pyi 文件)
- **线程安全**: 使用缓冲区机制确保线程安全的数据交换

## 系统要求

- Ubuntu 18.04/20.04/22.04 或兼容系统
- Python 3.6+
- CMake 3.12+
- GCC 7+ 或 Clang 6+
- Unitree SDK2 (需要安装到系统中)

## 依赖库

```bash
# 安装基本依赖
sudo apt-get update
sudo apt-get install build-essential cmake python3-dev python3-pip

# 安装 Python 依赖
pip3 install pybind11 pybind11-stubgen numpy
```

## 安装 Unitree SDK2

确保 Unitree SDK2 已经正确安装在系统中，默认路径为 `/opt/unitree/sdk2`。如果安装在其他位置，可以在编译时指定路径。

## 编译

### 自动编译

使用提供的构建脚本（推荐）：

```bash
# 基本编译
./build.sh

# 编译时安装 Python 依赖
./build.sh --install-deps

# 指定 Unitree SDK 路径
./build.sh --sdk-path /path/to/unitree/sdk2

# Debug 模式编译
./build.sh --debug

# 清理后重新编译
./build.sh --clean

# 查看所有选项
./build.sh --help
```

### 手动编译

```bash
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=Release -DUNITREE_SDK_PATH=/opt/unitree/sdk2 ..
make -j$(nproc)
```

## 编译指令详解

### CMake 选项

- `CMAKE_BUILD_TYPE`: 构建类型 (Release/Debug)
- `UNITREE_SDK_PATH`: Unitree SDK2 安装路径
- `GENERATE_STUBS`: 是否生成 Python 类型提示文件 (默认: ON)

### 构建输出

编译成功后会生成以下文件：

- `build/g1_interface*.so`: 编译后的 Python 模块
- `g1_interface_generated.pyi`: 自动生成的类型提示文件（如果启用）

## 使用方法

### 基本使用

```python
import g1_interface

# 初始化机器人接口
robot = g1_interface.G1Interface("eth0")  # 替换为实际的网络接口名

# 读取机器人状态
state = robot.read_low_state()
print(f"IMU RPY: {state.imu.rpy}")
print(f"Joint positions: {state.motor.q}")

# 读取无线控制器状态
controller = robot.read_wireless_controller()
print(f"Left stick: {controller.left_stick}")

# 创建零位置命令
cmd = robot.create_zero_command()

# 设置目标位置（例如：左踝关节）
q_target = list(cmd.q_target)
q_target[g1_interface.LeftAnklePitch] = 0.1  # 0.1 弧度
q_target[g1_interface.LeftAnkleRoll] = 0.0
cmd.q_target = q_target

# 发送命令到机器人
robot.write_low_command(cmd)
```

### 踝关节摆动示例

运行提供的踝关节摆动示例：

```bash
python3 example_ankle_swing.py eth0
```

这个示例演示了：
1. 机器人移动到零位置 (3秒)
2. 使用 PR 模式进行踝关节摆动 (3秒)
3. 保持零位置

## API 参考

### 主要类

#### `G1Interface`

主要的机器人控制接口类。

```python
def __init__(self, network_interface: str) -> None
def read_low_state(self) -> LowState
def read_wireless_controller(self) -> WirelessController
def write_low_command(self, command: MotorCommand) -> None
def set_control_mode(self, mode: ControlMode) -> None
def get_control_mode(self) -> ControlMode
def create_zero_command(self) -> MotorCommand
def get_default_kp(self) -> List[float]
def get_default_kd(self) -> List[float]
```

#### `LowState`

机器人低级状态信息。

```python
class LowState:
    imu: ImuState          # IMU 状态
    motor: MotorState      # 电机状态
    mode_machine: int      # 机器人模式
```

#### `MotorCommand`

电机控制命令。

```python
class MotorCommand:
    q_target: List[float]    # 目标关节位置 [rad] (29个)
    dq_target: List[float]   # 目标关节速度 [rad/s] (29个)
    kp: List[float]          # 位置增益 (29个)
    kd: List[float]          # 速度增益 (29个)
    tau_ff: List[float]      # 前馈力矩 [N*m] (29个)
```

### 关节索引

机器人有29个关节，可以使用预定义的常量来访问：

```python
# 腿部关节
g1_interface.LeftHipPitch      # 0
g1_interface.LeftHipRoll       # 1
g1_interface.LeftHipYaw        # 2
g1_interface.LeftKnee          # 3
g1_interface.LeftAnklePitch    # 4
g1_interface.LeftAnkleRoll     # 5
# ... 右腿关节 6-11

# 腰部关节 12-14
g1_interface.WaistYaw          # 12
g1_interface.WaistRoll         # 13
g1_interface.WaistPitch        # 14

# 手臂关节 15-28
g1_interface.LeftShoulderPitch # 15
# ... 其他手臂关节
```

### 控制模式

```python
g1_interface.ControlMode.PR    # Pitch/Roll 模式
g1_interface.ControlMode.AB    # A/B 模式
```

## 故障排除

### 编译错误

1. **找不到 Unitree SDK**:
   ```
   CMake Error: UNITREE_SDK not found
   ```
   解决：确保 SDK 正确安装，或使用 `--sdk-path` 指定路径

2. **找不到 pybind11**:
   ```
   CMake Error: pybind11 not found
   ```
   解决：`pip install pybind11`

3. **编译错误**:
   - 确保系统有足够的内存（建议4GB+）
   - 检查 GCC 版本是否支持 C++17

### 运行时错误

1. **模块导入失败**:
   ```python
   ImportError: No module named 'g1_interface'
   ```
   解决：确保编译后的 `.so` 文件在 Python 路径中

2. **网络连接失败**:
   ```
   Error: Failed to initialize DDS
   ```
   解决：检查网络接口名称是否正确，机器人是否连接

3. **权限错误**:
   解决：可能需要 sudo 权限或将用户添加到相应的组

## 性能优化

- 使用 Release 模式编译以获得最佳性能
- 控制循环频率建议不超过 500Hz
- 避免在控制循环中进行重的计算或 I/O 操作
- 使用实时内核以获得更好的时间确定性

## 安全注意事项

⚠️ **重要安全提示**:

- 在运行任何控制程序之前，确保机器人处于安全环境
- 始终准备紧急停止按钮
- 测试新控制算法时使用较小的运动幅度
- 监控关节温度和电压
- 使用无线控制器的 B 按钮作为紧急停止

## 示例程序

### 1. 基本状态读取

```python
import g1_interface
import time

robot = g1_interface.G1Interface("eth0")

for i in range(100):
    state = robot.read_low_state()
    print(f"IMU RPY: {state.imu.rpy}")
    time.sleep(0.01)
```

### 2. 简单关节控制

```python
import g1_interface
import math
import time

robot = g1_interface.G1Interface("eth0")
robot.set_control_mode(g1_interface.ControlMode.PR)

for i in range(1000):
    cmd = robot.create_zero_command()
    
    # 正弦波踝关节运动
    t = i * 0.002  # 2ms 时间步
    cmd.q_target[g1_interface.LeftAnklePitch] = 0.1 * math.sin(2 * math.pi * t)
    
    robot.write_low_command(cmd)
    time.sleep(0.002)
```

## 贡献

欢迎提交 issue 和 pull request 来改进这个项目。

## 许可证

请参考 Unitree SDK2 的许可证条款。

## 支持

如果遇到问题，请：
1. 检查上述故障排除部分
2. 查看 Unitree 官方文档
3. 提交 GitHub issue（如果适用）
