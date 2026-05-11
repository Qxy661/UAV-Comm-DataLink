# 开源仓库导读

---

## 1. MAVLink 生态仓库

### 1.1 MAVLink 官方仓库

**仓库地址:** https://github.com/mavlink/mavlink

```
mavlink/
├── message_definitions/
│   └── v1.0/
│       ├── common.xml          # 通用消息定义
│       ├── ardupilotmega.xml   # ArduPilot 扩展
│       ├── standard.xml        # 标准扩展
│       ├── development.xml     # 开发中消息
│       └── ...
├── pymavlink/                  # Python MAVLink 库
├── C/                          # C 语言库
└── tools/                      # 代码生成工具
```

**关键文件:**
- `common.xml`: 所有通用消息定义，必读
- `ardupilotmega.xml`: ArduPilot 特有消息

**学习建议:**
- 先读 `common.xml` 理解消息格式
- 使用 `mavgenerate` 工具生成库代码
- 参考 `pymavlink` 学习使用方法

---

### 1.2 pymavlink

**仓库地址:** https://github.com/ArduPilot/pymavlink

```
pymavlink/
├── mavutil.py          # 连接管理工具
├── mavparse.py         # 消息解析
├── mavgen.py           # 代码生成
├── mavexpression.py    # 表达式求值
└── tools/
    ├── mavlogdump.py   # 日志分析
    ├── MAVExplorer.py  # 日志浏览器
    └── mavgraph.py     # 数据绘图
```

**关键模块:**
- `mavutil.py`: 连接管理、消息收发
- `mavparse.py`: MAVLink 消息解析
- `mavgen.py`: 从 XML 生成代码

**学习建议:**
- 阅读 `mavutil.py` 理解连接管理
- 使用 `mavlogdump.py` 分析飞行日志
- 参考示例代码学习编程

---

### 1.3 MAVSDK

**仓库地址:** https://github.com/mavlink/MAVSDK

```
MAVSDK/
├── src/
│   ├── mavsdk/              # 核心库
│   │   ├── system.h         # 系统抽象
│   │   ├── action.h         # 飞行动作
│   │   ├── telemetry.h      # 遥测数据
│   │   ├── mission.h        # 任务管理
│   │   └── ...
│   └── plugins/             # 插件模块
├── examples/                # 示例代码
└── docs/                    # 文档
```

**关键模块:**
- `system.h`: 无人机系统抽象
- `action.h`: 飞行动作（起飞、降落等）
- `telemetry.h`: 遥测数据流
- `mission.h`: 任务管理

**学习建议:**
- 从 `examples/` 开始学习
- 阅读 `telemetry.h` 理解数据流
- 参考 `mission.h` 学习任务管理

---

## 2. 飞控固件仓库

### 2.1 ArduPilot

**仓库地址:** https://github.com/ArduPilot/ardupilot

```
ardupilot/
├── ArduCopter/         # 多旋翼飞控
├── ArduPlane/          # 固定翼飞控
├── ArduRover/          # 地面车辆
├── libraries/          # 共享库
│   ├── AP_HAL/         # 硬件抽象层
│   ├── AP_GPS/         # GPS 驱动
│   ├── AP_Compass/     # 磁力计
│   └── ...
└── Tools/              # 工具脚本
```

**关键目录:**
- `ArduCopter/`: 多旋翼控制代码
- `libraries/`: 共享库，通用功能
- `Tools/`: 构建和测试工具

**学习建议:**
- 从 `ArduCopter/GCS_Mavlink.cpp` 学习 MAVLink 处理
- 阅读 `libraries/AP_GPS/` 学习 GPS 驱动
- 参考 `Tools/autotest/` 学习自动化测试

---

### 2.2 PX4

**仓库地址:** https://github.com/PX4/PX4-Autopilot

```
PX4-Autopilot/
├── src/
│   ├── modules/
│   │   ├── mavlink/          # MAVLink 模块
│   │   ├── navigator/        # 导航模块
│   │   ├── commander/        # 指挥模块
│   │   └── ...
│   ├── lib/                  # 共享库
│   └── drivers/              # 驱动程序
├── msg/                      # uORB 消息定义
└── boards/                   # 板级支持
```

**关键目录:**
- `src/modules/mavlink/`: MAVLink 通信模块
- `src/modules/navigator/`: 导航和任务管理
- `msg/`: uORB 消息定义

**学习建议:**
- 从 `src/modules/mavlink/` 学习 MAVLink 实现
- 阅读 `msg/` 理解消息系统
- 参考 `src/modules/navigator/` 学习导航

---

## 3. 地面站仓库

### 3.1 QGroundControl

**仓库地址:** https://github.com/mavlink/qgroundcontrol

```
qgroundcontrol/
├── src/
│   ├── QmlControls/          # QML 控件
│   ├── FlightDisplay/        # 飞行显示
│   ├── MissionEditor/        # 任务编辑
│   ├── Vehicle/              # 无人机管理
│   └── ...
├── qml/                      # QML 界面文件
└── resources/                # 资源文件
```

**关键目录:**
- `src/FlightDisplay/`: 飞行仪表盘
- `src/MissionEditor/`: 任务规划界面
- `qml/`: QML 界面定义

**学习建议:**
- 从 `qml/` 学习界面设计
- 阅读 `src/Vehicle/` 理解无人机管理
- 参考 `src/MissionEditor/` 学习任务规划

---

### 3.2 MissionPlanner

**仓库地址:** https://github.com/ArduPilot/MissionPlanner

```
MissionPlanner/
├── ExtLibs/                # 外部库
├── GCSViews/               # 界面视图
│   ├── FlightData.cs       # 飞行数据
│   ├── FlightPlanner.cs    # 任务规划
│   └── ConfigurationView.cs # 配置
├── MainV2.cs               # 主窗口
└── Utilities/              # 工具类
```

**关键文件:**
- `GCSViews/FlightData.cs`: 飞行数据显示
- `GCSViews/FlightPlanner.cs`: 任务规划
- `MainV2.cs`: 主窗口入口

**学习建议:**
- 阅读 `GCSViews/` 理解界面设计
- 参考 `Utilities/` 学习工具开发
- 使用源码调试学习内部机制

---

## 4. 通信相关仓库

### 4.1 SiK Radio 固件

**仓库地址:** https://github.com/ArduPilot/SiK

```
SiK/
├── Firmware/
│   ├── radio/            # 主固件
│   ├── bootloader/       # 引导程序
│   └── library/          # 共享库
├── Tools/                # 工具
└── Hardware/             # 硬件设计
```

**关键文件:**
- `Firmware/radio/`: 主要固件代码
- `Firmware/library/`: RF 驱动库

**学习建议:**
- 阅读 `Firmware/radio/` 理解跳频机制
- 参考 `Firmware/library/` 学习 RF 编程

---

### 4.2 DroneKit

**仓库地址:** https://github.com/dronekit/dronekit-python

```
dronekit-python/
├── dronekit/
│   ├── __init__.py       # 主模块
│   ├── mavlink.py        # MAVLink 封装
│   └── util.py           # 工具函数
├── examples/             # 示例代码
└── docs/                 # 文档
```

**关键文件:**
- `dronekit/__init__.py`: 核心 API
- `examples/`: 丰富的示例代码

**学习建议:**
- 从 `examples/` 开始学习
- 阅读 `dronekit/__init__.py` 理解 API 设计

---

## 5. 5G/物联网相关仓库

### 5.1 OpenAirInterface (OAI)

**仓库地址:** https://github.com/openairinterface/openair-cn

**说明:** 开源 5G 核心网和接入网实现

**学习建议:**
- 适合深入学习 5G 协议栈
- 需要较强的通信背景知识

---

### 5.2 Eclipse Paho MQTT

**仓库地址:** https://github.com/eclipse/paho.mqtt.python

```
paho.mqtt.python/
├── paho/
│   └── mqtt/
│       ├── client.py     # MQTT 客户端
│       └── ...
└── examples/             # 示例代码
```

**学习建议:**
- 阅读 `paho/mqtt/client.py` 学习 MQTT 编程
- 参考示例代码实现无人机 MQTT 通信

---

### 5.3 CycloneDDS

**仓库地址:** https://github.com/eclipse-cyclonedds/cyclonedds

**说明:** 开源 DDS 实现，适合实时通信

**学习建议:**
- 适合学习 DDS 协议
- 可用于无人机集群实时通信

---

## 6. 学习路线建议

### 入门阶段（2-4 周）

```
1. pymavlink
   - 学习 MAVLink 协议基础
   - 掌握 Python 编程接口
   - 完成基本通信实验

2. MAVSDK
   - 学习高级 API
   - 掌握任务管理
   - 完成自动化飞行
```

### 进阶阶段（4-8 周）

```
3. ArduPilot
   - 理解飞控架构
   - 学习 MAVLink 处理
   - 参与社区贡献

4. QGroundControl
   - 学习地面站开发
   - 理解界面设计
   - 自定义功能开发
```

### 高级阶段（8+ 周）

```
5. MAVLink 官方仓库
   - 理解协议设计
   - 参与标准制定
   - 扩展自定义消息

6. 5G/物联网仓库
   - 学习 5G 协议栈
   - 掌握 MQTT/DDS
   - 开发网联无人机
```

---

## 7. 贡献指南

### 7.1 参与开源社区

```
贡献方式:
├── 代码贡献
│   ├── Bug 修复
│   ├── 新功能开发
│   └── 性能优化
│
├── 文档贡献
│   ├── 文档翻译
│   ├── 使用教程
│   └── API 文档
│
├── 测试贡献
│   ├── Bug 报告
│   ├── 测试用例
│   └── 兼容性测试
│
└── 社区贡献
    ├── 回答问题
    ├── 参与讨论
    └── 组织活动
```

### 7.2 学习资源

| 仓库 | 文档 | 社区 | 论坛 |
|------|------|------|------|
| ArduPilot | ardupilot.org | GitHub | discuss.ardupilot.org |
| PX4 | dev.px4.io | GitHub | discuss.px4.io |
| MAVLink | mavlink.io | GitHub | groups.google.com |
| QGC | qgroundcontrol.com | GitHub | discuss.ardupilot.org |
