# QGroundControl 深度使用

> 预计阅读：25 分钟 | 前置知识：MAVLink 协议基础

---

## 1. QGroundControl 概述

### 1.1 什么是 QGroundControl

QGroundControl（QGC）是开源的无人机地面站软件，支持 PX4 和 ArduPilot 飞控。

| 特性 | 说明 |
|------|------|
| 开源 | GPL 许可证 |
| 跨平台 | Windows, macOS, Linux, Android, iOS |
| 支持飞控 | PX4, ArduPilot |
| 功能 | 任务规划、遥测监控、参数配置、日志分析 |
| 开发框架 | Qt/QML |

### 1.2 安装

```bash
# 下载地址
# https://qgroundcontrol.com/downloads/

# Linux 安装
chmod +x QGroundControl.AppImage
./QGroundControl.AppImage

# 源码编译
git clone https://github.com/mavlink/qgroundcontrol.git
cd qgroundcontrol
git submodule update --init --recursive
qmake qgroundcontrol.pro
make -j$(nproc)
```

---

## 2. 连接配置

### 2.1 连接方式

| 连接方式 | 配置 | 适用场景 |
|---------|------|---------|
| USB 串口 | 自动检测 | 调试、初始配置 |
| 数传电台 | UDP 14550 | 飞行中遥测 |
| 4G/LTE | TCP/UDP | 超视距飞行 |
| Wi-Fi | UDP/TCP | 近距测试 |

### 2.2 连接设置

```
QGroundControl 连接配置:

1. 打开 QGroundControl
2. 点击左上角图标 → 通信设置
3. 添加新连接:
   - 类型: 串口 / UDP / TCP
   - 端口: COM3 (Windows) / /dev/ttyUSB0 (Linux)
   - 波特率: 57600 / 115200
   - 目标: 自动检测

UDP 监听配置:
- 端口: 14550 (默认)
- 可添加多个 UDP 端口
- 支持多机连接
```

---

## 3. 任务规划

### 3.1 任务规划界面

```
QGroundControl 任务规划:

地图界面:
┌─────────────────────────────────────────┐
│  [起飞] [航点] [盘旋] [降落] [返航]     │
├─────────────────────────────────────────┤
│                                         │
│          地图区域                        │
│          (点击添加航点)                  │
│                                         │
│                                         │
├─────────────────────────────────────────┤
│ 航点列表:                               │
│  1. 起飞 - 高度 10m                     │
│  2. 航点 - 30.123, 120.456, 20m         │
│  3. 航点 - 30.124, 120.457, 20m         │
│  4. 降落 - 30.123, 120.456              │
└─────────────────────────────────────────┘
```

### 3.2 航点参数设置

| 参数 | 说明 | 默认值 |
|------|------|--------|
| 高度 | 航点高度 | 10m |
| 速度 | 飞行速度 | 5 m/s |
| 停留时间 | 到达后等待时间 | 0s |
| 接受半径 | 判定到达的距离 | 2m |
| 通过半径 | 不停留直接通过 | 0m |

### 3.3 高级任务功能

```python
# QGroundControl 支持的任务类型

# 1. 起飞
takeoff_item = {
    'type': 'takeoff',
    'altitude': 10,
    'heading': 0
}

# 2. 航点
waypoint_item = {
    'type': 'waypoint',
    'lat': 30.1234567,
    'lon': 120.1234567,
    'alt': 20,
    'speed': 5,
    'hold_time': 0
}

# 3. 盘旋
loiter_item = {
    'type': 'loiter',
    'lat': 30.1234567,
    'lon': 120.1234567,
    'alt': 20,
    'radius': 10,      # 盘旋半径
    'turns': 3,        # 盘旋圈数
    'clockwise': True  # 方向
}

# 4. 降落
land_item = {
    'type': 'land',
    'lat': 30.1234567,
    'lon': 120.1234567
}

# 5. 返航
rtl_item = {
    'type': 'rtl'
}
```

---

## 4. 遥测监控

### 4.1 仪表盘

```
QGroundControl 仪表盘:

┌─────────────────────────────────────────┐
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ 姿态仪  │  │ 航向    │  │ 高度    │ │
│  │         │  │   N     │  │  100m   │ │
│  │  ○────○ │  │   ↑     │  │  ▲      │ │
│  └─────────┘  └─────────┘  └─────────┘ │
│  ┌─────────┐  ┌─────────┐  ┌─────────┐ │
│  │ 速度    │  │ 电量    │  │ GPS     │ │
│  │ 5 m/s   │  │ 80%     │  │ 3D Fix  │ │
│  └─────────┘  └─────────┘  └─────────┘ │
└─────────────────────────────────────────┘
```

### 4.2 消息监控

```python
# QGroundControl 消息面板显示:
# - STATUSTEXT 消息
# - 警告和错误信息
# - 系统状态变化

# 常见消息:
# "Prearm: GPS OK" - GPS 预检通过
# "EKF2 IMU0 is using GPS" - EKF 使用 GPS
# "Battery 1 low" - 电池低电量警告
# "Geofence breach" - 地理围栏突破
```

---

## 5. 参数配置

### 5.1 参数浏览器

```
QGroundControl 参数配置:

参数分类:
├── ATC (姿态控制)
│   ├── ATC_ANG_P_RP - Roll/Pitch P 增益
│   ├── ATC_ANG_P_YAW - Yaw P 增益
│   └── ATC_RAT_RP_P - Roll/Pitch Rate P
├── GPS
│   ├── GPS_TYPE - GPS 类型
│   └── GPS_AUTO_CONFIG - 自动配置
├── BATT (电池)
│   ├── BATT_CAPACITY - 电池容量
│   ├── BATT_LOW_VOLT - 低电压阈值
│   └── BATT_CRT_VOLT - 临界电压
├── FENCE (围栏)
│   ├── FENCE_ENABLE - 围栏使能
│   ├── FENCE_ALT_MAX - 最大高度
│   └── FENCE_RADIUS - 围栏半径
└── RTL (返航)
    ├── RTL_ALT - 返航高度
    ├── RTL_SPEED - 返航速度
    └── RTL_ALT_FINAL - 最终高度
```

### 5.2 参数修改示例

```python
# 通过 MAVLink 修改 QGroundControl 参数

# 1. 读取参数
conn.mav.param_request_read_send(
    conn.target_system,
    conn.target_component,
    b'RTL_ALT',
    -1
)
msg = conn.recv_match(type='PARAM_VALUE', blocking=True)
print(f"RTL_ALT = {msg.param_value}")

# 2. 修改参数
conn.mav.param_set_send(
    conn.target_system,
    conn.target_component,
    b'RTL_ALT',
    50.0,  # 新值
    9      # MAV_PARAM_TYPE_REAL32
)
```

---

## 6. 地理围栏

### 6.1 围栏类型

| 类型 | 说明 | 用途 |
|------|------|------|
| 多边形围栏 | 自定义多边形区域 | 限制飞行区域 |
| 圆形围栏 | 以起飞点为圆心 | 简单限制 |
| 高度围栏 | 最大/最小高度 | 高度限制 |
| 圆柱围栏 | 圆形 + 高度 | 3D 限制 |

### 6.2 围栏配置

```python
# 地理围栏配置示例
fence_items = [
    # 多边形顶点
    {'lat': 30.123, 'lon': 120.456},
    {'lat': 30.124, 'lon': 120.456},
    {'lat': 30.124, 'lon': 120.457},
    {'lat': 30.123, 'lon': 120.457},
]

# 围栏参数
fence_params = {
    'FENCE_ENABLE': 1,
    'FENCE_TYPE': 6,        # 多边形 + 高度
    'FENCE_ALT_MAX': 120,   # 最大高度 (m)
    'FENCE_ALT_MIN': 10,    # 最小高度 (m)
    'FENCE_RADIUS': 500,    # 圆形围栏半径 (m)
    'FENCE_ACTION': 1,      # 动作: 0=报告, 1=RTL, 2=Land, 3=Loiter
}
```

---

## 7. Rally Points

### 7.1 Rally Points 概念

Rally Points 是备降点，当触发返航时可以选择最近的 Rally Point 而非起飞点。

```
Rally Points 示意图:

        Rally Point 2
            ★
           ╱
          ╱
起飞点 ★─────── 无人机当前位置
          ╲       ●
           ╲
            ★
        Rally Point 1

返航时选择最近的 Rally Point
```

### 7.2 Rally Points 配置

```python
# Rally Points 上传
rally_points = [
    {'lat': 30.125, 'lon': 120.458, 'alt': 30},
    {'lat': 30.122, 'lon': 120.455, 'alt': 20},
]

for i, rp in enumerate(rally_points):
    conn.mav.rally_point_send(
        conn.target_system,
        conn.target_component,
        i,
        len(rally_points),
        int(rp['lat'] * 1e7),
        int(rp['lon'] * 1e7),
        int(rp['alt'] * 100),
        0, 0, 0
    )
```

---

## 8. 自定义工具栏

### 8.1 QML 自定义

```qml
// 自定义工具栏按钮
import QGroundControl.Controls 1.0

QGCButton {
    text: "自定义动作"
    onClicked: {
        // 发送自定义 MAVLink 命令
        QGroundControl.sendCommand(
            1,  // target system
            1,  // target component
            511, // MAV_CMD_DO_SET_SERVO
            0,   // confirmation
            8,   // param1: servo channel
            1500 // param2: PWM value
        )
    }
}
```

### 8.2 自定义仪表盘

```qml
// 自定义仪表盘显示
import QtQuick 2.0
import QGroundControl.Controls 1.0

Item {
    width: 200
    height: 100
    
    Rectangle {
        anchors.fill: parent
        color: "black"
        opacity: 0.8
        
        Column {
            anchors.centerIn: parent
            
            Text {
                text: "自定义数据"
                color: "white"
                font.pixelSize: 16
            }
            
            Text {
                text: "值: " + QGroundControl.mavlink.getParam("CUSTOM_VALUE")
                color: "green"
                font.pixelSize: 24
            }
        }
    }
}
```

---

## 思考题

1. **QGroundControl 支持哪些连接方式？各适用于什么场景？**

2. **如何使用 QGroundControl 规划一个包含 5 个航点的矩形巡航任务？**

3. **地理围栏的 ACTION 参数有哪些选项？各有什么效果？**

4. **Rally Points 和普通返航点有什么区别？在什么场景下使用 Rally Points？**

5. **如何自定义 QGroundControl 的工具栏和仪表盘？**

<details>
<summary>参考答案</summary>

**1. 连接方式：**

- USB 串口：调试和初始配置
- 数传电台：飞行中遥测
- 4G/LTE：超视距飞行
- Wi-Fi：近距测试

**2. 矩形巡航任务规划：**

1. 打开任务规划界面
2. 点击地图添加 4 个航点形成矩形
3. 设置每个航点高度和速度
4. 添加起飞和降落点
5. 保存并上传任务

**3. 围栏 ACTION 选项：**

- 0: 仅报告
- 1: RTL（返航）
- 2: Land（原地降落）
- 3: Loiter（盘旋等待）
- 4: Land（降落并锁定）

**4. Rally Points 区别：**

- 返航点：固定为起飞点
- Rally Points：多个备降点，选择最近的
- 场景：起飞点不适合降落时（如移动平台、障碍物）

**5. 自定义方法：**

- 工具栏：修改 QML 文件
- 仪表盘：创建自定义 QML 组件
- 参考 QGC 源码中的示例

</details>
