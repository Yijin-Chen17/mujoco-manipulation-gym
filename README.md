# mujoco-manipulation-gym (`mmgym`)

> 从零手搓的现代工业级机器人仿真与具身控制框架：**C++20 计算内核 + Pinocchio 动力学 + MuJoCo C API + nanobind 零拷贝绑定 + HAL 硬件抽象 + Gymnasium 具身任务**。

本项目兼具**学习实践**与**工程规范落地**双重目的：彻底摆脱纯 Python 科研原型的 GIL 锁、慢速优化和软硬件耦合限制，通过手写每一层底层核心代码，深入掌握微秒级运动学/动力学、1kHz 无锁实时控制以及 Sim2Real 架构设计。

---

## 核心技术栈

* **包管理与构建**：Pixi (`conda-forge`) + CMake (`>=3.22`) + Ninja + C++20
* **线性代数与刚体动力学**：Eigen + Pinocchio
* **物理仿真引擎**：MuJoCo C API (`mujoco/mujoco.h`)
* **语言绑定与互操作**：nanobind (零拷贝 `ndarray` / DLPack)
* **异步日志与单元测试**：spdlog + Catch2 v3
* **上层强化学习与具身生态**：Python 3.10 + NumPy + Gymnasium + HDF5/Zarr

---

## 工程目标与六阶段路线

完整的技术规格与验收标准见 [`ROBOTICS_FRAMEWORK_DEV_BLUEPRINT.md`](./ROBOTICS_FRAMEWORK_DEV_BLUEPRINT.md)：

1. **阶段 1**：工程骨架、Eigen 强类型数学底座、Pinocchio 模型封装与微秒级 DLS-IK 求解器（进行中）
2. **阶段 2**：MuJoCo C API RAII 安全封装、离屏渲染管线与 nanobind 零拷贝 Python 绑定
3. **阶段 3**：`IHardwareInterface` 硬件抽象层 (HAL)、SPSC 无锁双缓冲与 1kHz 关节/笛卡尔操作空间阻抗控制 (OSC)
4. **阶段 4**：模块化机器人资产（Panda / UR5e + 夹爪）与 Pinocchio/MuJoCo 双模型对齐校验
5. **阶段 5**：标准 Gymnasium 具身任务集（Pick & Place、开抽屉、双臂交接）与 Reward 状态机
6. **阶段 6**：双线程遥操作（键盘/手柄）、HDF5/Zarr 示教数据录制回放与 Sim2Real 真机接口闭环

---

## 当前目录结构

```text
mujoco-manipulation-gym/
├── pixi.toml                           # Pixi 跨平台依赖声明
├── pixi.lock                           # 精确版本锁文件
├── CMakeLists.txt                      # 顶层 CMake (C++20, Release 优化, 依赖查找)
├── ROBOTICS_FRAMEWORK_DEV_BLUEPRINT.md # 六阶段完整技术规格与验收蓝图
├── AI_TUTOR_GUIDE.md                   # AI 导师教学规范与新电脑迁移指南
├── core/
│   ├── CMakeLists.txt
│   ├── include/mmgym/
│   │   ├── common/math_types.hpp       # 基础向量/矩阵别名、7-DoF 栈特化、SE(3) Pose
│   │   ├── kinematics/                 # (待手搓) Pinocchio 模型与 DLS-IK
│   │   ├── simulation/                 # (待手搓) MuJoCo C API RAII 封装
│   │   ├── control/                    # (待手搓) HAL 与 1kHz 阻抗控制器
│   │   └── realtime/                   # (待手搓) 无锁环形缓冲区
│   └── src/
├── bindings/                           # (待手搓) nanobind C++/Python 导出层
├── python/mmgym/                       # (待手搓) Gymnasium 环境、遥操作与数据集录制
├── assets/                             # 机器人 URDF/MJCF 模型与场景资产
└── tests/
    ├── cpp/
    │   ├── CMakeLists.txt
    │   └── test_math_types.cpp         # Pose 与基础数学单元测试
    └── python/
```

---

## 迁移到新电脑后的快速启动

### 1. 安装 Pixi
* **macOS / Linux**：
  ```bash
  curl -fsSL https://pixi.sh/install.sh | bash
  ```
* 如果新电脑是 **Linux x86_64**，请先在 `pixi.toml` 的 `platforms` 中加上 `"linux-64"`（例如 `platforms = ["osx-arm64", "linux-64"]`），再执行下一步。

### 2. 还原全部 C++ / Python 依赖环境
```bash
git clone <你的仓库地址>
cd mujoco-manipulation-gym
pixi install
```

### 3. 编译与运行 C++ 单元测试
```bash
# 生成 Ninja 构建文件
pixi run cmake -B build -G Ninja

# 编译
pixi run cmake --build build

# 运行现有测试可执行文件
./build/tests/cpp/test_math_types
```
*(提示：若想直接使用 `pixi run ctest --test-dir build --output-on-failure`，请确认根目录 `CMakeLists.txt` 中已开启 `enable_testing()`。)*

---

## 协作方式

本项目采用 **“开发者亲手编码 + AI 导师引导与 Code Review”** 模式推进。如果你在新电脑上使用 AI 编程助手继续开发，请先让它阅读 [`AI_TUTOR_GUIDE.md`](./AI_TUTOR_GUIDE.md)。
