# MissionPlanner 高级功能

> 预计阅读：25 分钟 | 前置知识：MAVLink 协议、ArduPilot 基础

---

## 1. MissionPlanner 概述

### 1.1 功能特性

| 功能 | 说明 |
|------|------|
| 任务规划 | 航点编辑、区域扫描、测绘任务 |
| 遥测监控 | 实时数据、地图显示、仪表盘 |
| 参数配置 | 完整参数树、搜索、批量修改 |
| 日志分析 | 飞行日志回放、图表分析 |
| 固件更新 | ArduPilot 固件烧录 |
| 调参 | 自动调参、手动调参 |

### 1.2 安装

```
下载地址: https://ardupilot.org/planner2/

Windows: 安装包自动安装
Linux: mono MissionPlanner.exe
```

---

## 2. 自动调参（AutoTune）

### 2.1 AutoTune 原理

AutoTune 自动调整 PID 参数，优化飞行性能。

```
AutoTune 流程:

1. 切换到 AutoTune 模式
2. 飞控自动执行振荡测试
3. 测量系统响应
4. 计算最优 PID 参数
5. 应用新参数
6. 测试飞行
```

### 2.2 AutoTune 参数

| 参数 | 说明 | 默认值 |
|------|------|--------|
| AUTOTUNE_AGGR | 激进程度 | 0.1 |
| AUTOTUNE_MIN_D | 最小 D 增益 | 0.001 |
| AUTOTUNE_AXES | 调参轴 | Roll, Pitch, Yaw |

### 2.3 AutoTune 流程

```python
# AutoTune MAVLink 命令
# 1. 设置飞行模式为 AutoTune
conn.mav.command_long_send(
    conn.target_system,
    conn.target_component,
    176,  # MAV_CMD_DO_SET_MODE
    0,
    1,    # 自定义模式
    17,   # AutoTune 模式 ID (ArduCopter)
    0, 0, 0, 0, 0
)

# 2. 等待 AutoTune 完成
# AutoTune 会自动执行振荡测试
# 完成后会自动回到 AutoTune 模式

# 3. 保存参数
conn.mav.command_long_send(
    conn.target_system,
    conn.target_component,
    176,
    0,
    1,
    17,  # 保持 AutoTune
    0, 0, 0, 0, 0
)
```

---

## 3. 地理围栏（GeoFence）

### 3.1 围栏类型

| 类型 | 说明 | 配置 |
|------|------|------|
| 多边形 | 自定义区域 | 地图绘制 |
| 圆形 | 以起飞点为中心 | 半径设置 |
| 高度 | 最大/最小高度 | 高度值 |
| 圆柱 | 圆形 + 高度 | 半径 + 高度 |

### 3.2 围栏配置

```python
# 围栏参数
fence_params = {
    'FENCE_ENABLE': 1,        # 启用围栏
    'FENCE_TYPE': 6,          # 类型: 多边形 + 高度
    'FENCE_ACTION': 1,        # 动作: RTL
    'FENCE_ALT_MAX': 120,     # 最大高度 (m)
    'FENCE_ALT_MIN': 10,      # 最小高度 (m)
    'FENCE_RADIUS': 500,      # 圆形半径 (m)
    'FENCE_MARGIN': 10,       # 边距 (m)
}

# 上传多边形围栏顶点
fence_points = [
    (30.123, 120.456),
    (30.124, 120.456),
    (30.124, 120.457),
    (30.123, 120.457),
]

for i, (lat, lon) in enumerate(fence_points):
    conn.mav.fence_point_send(
        conn.target_system,
        conn.target_component,
        i,
        len(fence_points),
        lat,
        lon
    )
```

### 3.3 围栏动作

| 动作 | ID | 说明 |
|------|----|------|
| 报告 | 0 | 仅发送警告 |
| RTL | 1 | 返航 |
| Land | 2 | 原地降落 |
| Loiter | 3 | 盘旋等待 |

---

## 4. Rally Points

### 4.1 Rally Points 配置

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

# Rally Points 参数
rally_params = {
    'RALLY_LIMIT_KM': 5.0,    # 最大距离 (km)
    'RALLY_INCL_HOME': 1,      # 包含起飞点
}
```

---

## 5. 日志分析

### 5.1 飞行日志格式

ArduPilot 支持多种日志格式：

| 格式 | 说明 | 工具 |
|------|------|------|
| BIN | 二进制格式 | MissionPlanner |
| LOG | 文本格式 | MAVExplorer |
| TLOG | MAVLink 通信日志 | MissionPlanner |

### 5.2 日志分析工具

```python
# 使用 pymavlink 分析日志
from pymavlink import mavutil

# 打开日志文件
log = mavutil.mavlink_connection('flight.bin', dialect='ardupilotmega')

# 读取所有消息
while True:
    msg = log.recv_match(blocking=False)
    if msg is None:
        break
    
    msg_type = msg.get_type()
    
    if msg_type == 'ATT':
        # 姿态数据
        print(f"Roll: {msg.Roll}, Pitch: {msg.Pitch}, Yaw: {msg.Yaw}")
    
    elif msg_type == 'GPS':
        # GPS 数据
        print(f"Lat: {msg.Lat}, Lon: {msg.Lng}, Alt: {msg.Alt}")
    
    elif msg_type == 'BAT':
        # 电池数据
        print(f"Volt: {msg.Volt}, Curr: {msg.Curr}")
```

### 5.3 日志关键数据

| 数据 | 消息类型 | 说明 |
|------|---------|------|
| 姿态 | ATT | Roll, Pitch, Yaw |
| GPS | GPS | 位置、速度 |
| 电池 | BAT | 电压、电流、电量 |
| 模式 | MODE | 飞行模式变化 |
| 事件 | EV | 解锁、起飞等事件 |
| 电机 | MOT | 电机输出 |
| PID | PID | PID 控制数据 |

---

## 6. 数据分析示例

### 6.1 飞行轨迹分析

```python
import matplotlib.pyplot as plt
import pandas as pd

def analyze_flight_path(log_file):
    """分析飞行轨迹"""
    log = mavutil.mavlink_connection(log_file)
    
    positions = []
    while True:
        msg = log.recv_match(type='GPS', blocking=False)
        if msg is None:
            break
        positions.append({
            'lat': msg.Lat,
            'lon': msg.Lng,
            'alt': msg.Alt,
            'time': msg.TimeUS
        })
    
    df = pd.DataFrame(positions)
    
    # 绘制轨迹
    fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
    
    # 2D 轨迹
    ax1.plot(df['lon'], df['lat'])
    ax1.set_xlabel('经度')
    ax1.set_ylabel('纬度')
    ax1.set_title('飞行轨迹')
    
    # 高度变化
    ax2.plot(df['time'] - df['time'][0], df['alt'])
    ax2.set_xlabel('时间 (μs)')
    ax2.set_ylabel('高度 (m)')
    ax2.set_title('高度变化')
    
    plt.tight_layout()
    plt.show()
```

### 6.2 电池分析

```python
def analyze_battery(log_file):
    """分析电池数据"""
    log = mavutil.mavlink_connection(log_file)
    
    battery_data = []
    while True:
        msg = log.recv_match(type='BAT', blocking=False)
        if msg is None:
            break
        battery_data.append({
            'voltage': msg.Volt,
            'current': msg.Curr,
            'consumed': msg.CurrTot,
            'time': msg.TimeUS
        })
    
    df = pd.DataFrame(battery_data)
    
    # 绘制电池数据
    fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 8))
    
    # 电压
    ax1.plot(df['time'] - df['time'][0], df['voltage'])
    ax1.set_ylabel('电压 (V)')
    ax1.set_title('电池电压')
    
    # 电流
    ax2.plot(df['time'] - df['time'][0], df['current'])
    ax2.set_xlabel('时间 (μs)')
    ax2.set_ylabel('电流 (A)')
    ax2.set_title('电池电流')
    
    plt.tight_layout()
    plt.show()
```

---

## 思考题

1. **AutoTune 的工作原理是什么？为什么不能手动调整 PID 参数？**

2. **地理围栏有哪些类型？各适用于什么场景？**

3. **如何使用 MissionPlanner 分析飞行日志中的异常？**

4. **Rally Points 和返航点有什么区别？在什么情况下使用 Rally Points？**

5. **如何通过日志分析判断飞控的健康状态？**

<details>
<summary>参考答案</summary>

**1. AutoTune 原理：**

- 执行振荡测试，测量系统响应
- 分析频率和阻尼特性
- 计算最优 PID 参数
- 比手动调整更精确、更安全

**2. 围栏类型及场景：**

- 多边形：限制飞行区域（如禁飞区）
- 圆形：简单限制（以起飞点为中心）
- 高度：防止飞太高或太低
- 圆柱：3D 限制

**3. 日志异常分析：**

- 查看模式变化（MODE 消息）
- 检查 GPS 状态（GPS 消息）
- 分析电池数据（BAT 消息）
- 查看事件日志（EV 消息）
- 绘制姿态曲线检查振荡

**4. Rally Points 区别：**

- 返航点：固定为起飞点
- Rally Points：多个备降点
- 场景：起飞点不适合降落时

**5. 健康状态判断：**

- GPS：卫星数、定位类型
- 电池：电压、电流、温度
- 姿态：稳定性、振荡
- 电机：输出平衡

</details>
