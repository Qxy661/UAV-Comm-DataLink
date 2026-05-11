# MAVSDK 开发

> 预计阅读：30 分钟 | 前置知识：MAVLink 协议、Python/C++ 编程

---

## 1. MAVSDK 概述

### 1.1 什么是 MAVSDK

MAVSDK 是 MAVLink 的高级 SDK，提供简洁的 API 用于无人机应用开发。

| 特性 | 说明 |
|------|------|
| 多语言 | C++, Python, Java, Swift |
| 异步 | 基于异步编程模型 |
| 插件化 | 功能模块化（Action, Telemetry, Mission 等） |
| 跨平台 | Linux, macOS, Windows, Android, iOS |

### 1.2 安装

```bash
# Python 安装
pip install mavsdk

# C++ 安装 (Ubuntu)
sudo apt-get install libmavsdk-dev

# 从源码编译
git clone https://github.com/mavlink/MAVSDK.git
cd MAVSDK
git submodule update --init --recursive
cmake -Bbuild -DCMAKE_BUILD_TYPE=Release
cmake --build build -j$(nproc)
sudo cmake --build build --target install
```

---

## 2. Python MAVSDK 入门

### 2.1 基本连接

```python
#!/usr/bin/env python3
"""MAVSDK Python 基本示例"""

import asyncio
from mavsdk import System

async def main():
    # 创建无人机对象
    drone = System()
    
    # 连接到飞控
    await drone.connect(system_address="udp://:14540")
    
    # 等待连接
    print("等待连接...")
    async for state in drone.core.connection_state():
        if state.is_connected:
            print("已连接!")
            break
    
    # 获取飞控版本信息
    info = await drone.info.get_version()
    print(f"飞控版本: {info.flight_sw_major}.{info.flight_sw_minor}.{info.flight_sw_patch}")

if __name__ == '__main__':
    asyncio.run(main())
```

### 2.2 异步编程模型

```python
# MAVSDK 使用 Python asyncio
# 所有 API 调用都是异步的

async def example():
    drone = System()
    await drone.connect("udp://:14540")
    
    # 并行执行多个任务
    await asyncio.gather(
        get_telemetry(drone),
        monitor_health(drone),
        print_position(drone)
    )

async def get_telemetry(drone):
    """获取遥测数据"""
    async for position in drone.telemetry.position():
        print(f"位置: {position.latitude_deg}, {position.longitude_deg}")

async def monitor_health(drone):
    """监控健康状态"""
    async for health in drone.telemetry.health():
        print(f"GPS: {health.is_global_position_ok}, "
              f"电池: {health.is_battery_ok}")

async def print_position(drone):
    """打印位置"""
    async for attitude in drone.telemetry.attitude_euler():
        print(f"姿态: roll={attitude.roll_deg:.1f}, "
              f"pitch={attitude.pitch_deg:.1f}, "
              f"yaw={attitude.yaw_deg:.1f}")
```

---

## 3. 遥测数据流

### 3.1 位置信息

```python
async def get_position(drone):
    """获取位置信息"""
    # GPS 位置
    async for position in drone.telemetry.position():
        print(f"纬度: {position.latitude_deg}")
        print(f"经度: {position.longitude_deg}")
        print(f"海拔: {position.absolute_altitude_m}")
        print(f"相对高度: {position.relative_altitude_m}")

    # 融合位置
    async for home in drone.telemetry.home():
        print(f"起飞点: {home.latitude_deg}, {home.longitude_deg}")
```

### 3.2 姿态信息

```python
async def get_attitude(drone):
    """获取姿态信息"""
    # 欧拉角
    async for attitude in drone.telemetry.attitude_euler():
        print(f"Roll: {attitude.roll_deg:.1f}°")
        print(f"Pitch: {attitude.pitch_deg:.1f}°")
        print(f"Yaw: {attitude.yaw_deg:.1f}°")
    
    # 四元数
    async for attitude in drone.telemetry.attitude_quaternion():
        print(f"W: {attitude.w:.3f}")
        print(f"X: {attitude.x:.3f}")
        print(f"Y: {attitude.y:.3f}")
        print(f"Z: {attitude.z:.3f}")
```

### 3.3 系统状态

```python
async def get_system_status(drone):
    """获取系统状态"""
    # 电池状态
    async for battery in drone.telemetry.battery():
        print(f"电量: {battery.remaining_percent:.1%}")
        print(f"电压: {battery.voltage_v:.1f}V")
    
    # GPS 信息
    async for gps_info in drone.telemetry.gps_info():
        print(f"卫星数: {gps_info.num_satellites}")
        print(f"定位类型: {gps_info.fix_type}")
    
    # 飞行模式
    async for flight_mode in drone.telemetry.flight_mode():
        print(f"飞行模式: {flight_mode}")
    
    # 在空中状态
    async for in_air in drone.telemetry.in_air():
        print(f"在空中: {in_air}")
```

---

## 4. 飞行动作

### 4.1 解锁与起飞

```python
async def arm_and_takeoff(drone):
    """解锁并起飞"""
    print("解锁...")
    await drone.action.arm()
    
    print("起飞到 10m...")
    await drone.action.set_takeoff_altitude(10.0)
    await drone.action.takeoff()
    
    # 等待达到高度
    async for position in drone.telemetry.position():
        if position.relative_altitude_m >= 9.5:
            print(f"已达到高度: {position.relative_altitude_m:.1f}m")
            break
    
    print("起飞完成!")
```

### 4.2 飞往位置

```python
async def goto_location(drone, lat, lon, alt):
    """飞往指定位置"""
    print(f"飞往: {lat}, {lon}, {alt}m")
    await drone.action.goto_location(lat, lon, alt, 0)
    
    # 等待到达
    async for position in drone.telemetry.position():
        distance = calculate_distance(
            position.latitude_deg, position.longitude_deg,
            lat, lon
        )
        if distance < 2.0:  # 2m 内视为到达
            print("已到达目标位置!")
            break
```

### 4.3 降落与锁定

```python
async def land_and_disarm(drone):
    """降落并锁定"""
    print("降落...")
    await drone.action.land()
    
    # 等待降落完成
    async for in_air in drone.telemetry.in_air():
        if not in_air:
            print("已降落!")
            break
    
    # 等待自动锁定
    print("等待锁定...")
    await asyncio.sleep(3)
    
    async for armed in drone.telemetry.armed():
        if not armed:
            print("已锁定!")
            break
```

---

## 5. 任务规划

### 5.1 创建任务

```python
from mavsdk.mission import MissionItem, MissionPlan

async def create_mission(drone):
    """创建飞行任务"""
    mission_items = []
    
    # 起飞点
    mission_items.append(MissionItem(
        latitude_deg=30.1234567,
        longitude_deg=120.1234567,
        relative_altitude_m=10.0,
        speed_m_s=5.0,
        is_fly_through=True,
        gimbal_pitch_deg=float('nan'),
        gimbal_yaw_deg=float('nan'),
        camera_action=MissionItem.CameraAction.NONE,
        loiter_time_s=float('nan'),
        camera_photo_interval_s=float('nan'),
        acceptance_radius_m=2.0,
        yaw_deg=0.0,
        camera_photo_distance_m=float('nan'),
    ))
    
    # 航点 1
    mission_items.append(MissionItem(
        latitude_deg=30.1240000,
        longitude_deg=120.1240000,
        relative_altitude_m=20.0,
        speed_m_s=5.0,
        is_fly_through=True,
        gimbal_pitch_deg=float('nan'),
        gimbal_yaw_deg=float('nan'),
        camera_action=MissionItem.CameraAction.NONE,
        loiter_time_s=float('nan'),
        camera_photo_interval_s=float('nan'),
        acceptance_radius_m=2.0,
        yaw_deg=0.0,
        camera_photo_distance_m=float('nan'),
    ))
    
    # 航点 2
    mission_items.append(MissionItem(
        latitude_deg=30.1250000,
        longitude_deg=120.1250000,
        relative_altitude_m=20.0,
        speed_m_s=5.0,
        is_fly_through=True,
        gimbal_pitch_deg=float('nan'),
        gimbal_yaw_deg=float('nan'),
        camera_action=MissionItem.CameraAction.NONE,
        loiter_time_s=float('nan'),
        camera_photo_interval_s=float('nan'),
        acceptance_radius_m=2.0,
        yaw_deg=0.0,
        camera_photo_distance_m=float('nan'),
    ))
    
    # 降落点
    mission_items.append(MissionItem(
        latitude_deg=30.1234567,
        longitude_deg=120.1234567,
        relative_altitude_m=0.0,
        speed_m_s=5.0,
        is_fly_through=False,
        gimbal_pitch_deg=float('nan'),
        gimbal_yaw_deg=float('nan'),
        camera_action=MissionItem.CameraAction.NONE,
        loiter_time_s=float('nan'),
        camera_photo_interval_s=float('nan'),
        acceptance_radius_m=2.0,
        yaw_deg=0.0,
        camera_photo_distance_m=float('nan'),
    ))
    
    return MissionPlan(mission_items)
```

### 5.2 上传与执行任务

```python
async def upload_and_start_mission(drone, mission_plan):
    """上传并执行任务"""
    # 上传任务
    print("上传任务...")
    await drone.mission.upload_mission(mission_plan)
    print("任务上传完成!")
    
    # 开始任务
    print("开始任务...")
    await drone.mission.start_mission()
    
    # 监控任务进度
    async for mission_progress in drone.mission.mission_progress():
        print(f"任务进度: {mission_progress.current}/{mission_progress.total}")
        if mission_progress.current == mission_progress.total:
            print("任务完成!")
            break
```

### 5.3 任务控制

```python
async def mission_control(drone):
    """任务控制"""
    # 暂停任务
    await drone.mission.pause_mission()
    
    # 恢复任务
    await drone.mission.start_mission()
    
    # 取消任务
    await drone.mission.clear_mission()
    
    # 下载当前任务
    mission_plan = await drone.mission.download_mission()
    for i, item in enumerate(mission_plan.mission_items):
        print(f"航点 {i}: {item.latitude_deg}, {item.longitude_deg}")
```

---

## 6. 参数管理

### 6.1 参数读写

```python
async def parameter_management(drone):
    """参数管理"""
    # 获取所有参数
    params = await drone.param.get_all_params()
    
    # 获取整数参数
    param_int = await drone.param.get_param_int("SYSID_THISMAV")
    print(f"系统 ID: {param_int}")
    
    # 获取浮点参数
    param_float = await drone.param.get_param_float("RTL_ALT")
    print(f"返航高度: {param_float}")
    
    # 设置参数
    await drone.param.set_param_float("RTL_ALT", 50.0)
    print("返航高度已设置为 50m")
    
    await drone.param.set_param_int("SYSID_THISMAV", 1)
    print("系统 ID 已设置为 1")
```

---

## 7. 实际应用示例

### 7.1 自动巡检程序

```python
#!/usr/bin/env python3
"""自动巡检程序"""

import asyncio
from mavsdk import System
from mavsdk.mission import MissionItem, MissionPlan

class InspectionDrone:
    def __init__(self):
        self.drone = System()
    
    async def connect(self, address="udp://:14540"):
        await self.drone.connect(system_address=address)
        async for state in self.drone.core.connection_state():
            if state.is_connected:
                print("已连接!")
                return
    
    async def preflight_check(self):
        """飞行前检查"""
        async for health in self.drone.telemetry.health():
            checks = {
                'GPS': health.is_global_position_ok,
                '电池': health.is_battery_ok,
                '遥控': health.is_local_position_ok,
            }
            
            all_ok = all(checks.values())
            for name, status in checks.items():
                print(f"  {name}: {'通过' if status else '失败'}")
            
            if all_ok:
                print("飞行前检查通过!")
                return True
            else:
                print("飞行前检查失败!")
                return False
    
    async def fly_inspection_route(self, waypoints):
        """执行巡检任务"""
        # 创建任务
        mission_items = []
        for wp in waypoints:
            mission_items.append(MissionItem(
                latitude_deg=wp['lat'],
                longitude_deg=wp['lon'],
                relative_altitude_m=wp['alt'],
                speed_m_s=wp.get('speed', 5.0),
                is_fly_through=True,
                gimbal_pitch_deg=float('nan'),
                gimbal_yaw_deg=float('nan'),
                camera_action=MissionItem.CameraAction.NONE,
                loiter_time_s=wp.get('loiter', float('nan')),
                camera_photo_interval_s=float('nan'),
                acceptance_radius_m=2.0,
                yaw_deg=0.0,
                camera_photo_distance_m=float('nan'),
            ))
        
        mission_plan = MissionPlan(mission_items)
        
        # 上传并执行
        await self.drone.mission.upload_mission(mission_plan)
        await self.drone.action.arm()
        await self.drone.mission.start_mission()
        
        # 监控进度
        async for progress in self.drone.mission.mission_progress():
            print(f"进度: {progress.current}/{progress.total}")
            if progress.current == progress.total:
                print("巡检完成!")
                break
    
    async def return_and_land(self):
        """返航降落"""
        await self.drone.action.return_to_launch()
        
        async for in_air in self.drone.telemetry.in_air():
            if not in_air:
                print("已降落!")
                break

async def main():
    # 巡检航点
    waypoints = [
        {'lat': 30.123, 'lon': 120.456, 'alt': 20, 'loiter': 5},
        {'lat': 30.124, 'lon': 120.457, 'alt': 25, 'loiter': 5},
        {'lat': 30.125, 'lon': 120.458, 'alt': 20, 'loiter': 5},
        {'lat': 30.126, 'lon': 120.459, 'alt': 15, 'loiter': 5},
    ]
    
    drone = InspectionDrone()
    await drone.connect()
    
    if await drone.preflight_check():
        await drone.fly_inspection_route(waypoints)
        await drone.return_and_land()

if __name__ == '__main__':
    asyncio.run(main())
```

---

## 思考题

1. **MAVSDK 和 pymavlink 有什么区别？各适用于什么场景？**

2. **MAVSDK 的异步编程模型有什么优势？为什么选择 asyncio？**

3. **如何使用 MAVSDK 实现一个自动起飞、巡航、降落的完整流程？**

4. **MAVSDK 的 MissionItem 有哪些参数？如何设置相机动作？**

5. **如何处理 MAVSDK 连接断开和重连？**

<details>
<summary>参考答案</summary>

**1. MAVSDK vs pymavlink：**

MAVSDK：
- 高级 API，易用
- 异步编程
- 多语言支持
- 适合应用开发

pymavlink：
- 低级 API，灵活
- 直接操作 MAVLink 消息
- 适合协议开发和调试

**2. 异步编程优势：**

- 并发执行：同时处理遥测和发送命令
- 非阻塞：不会阻塞主循环
- 响应性：UI 保持响应
- 效率：减少线程开销

**3. 完整流程：**

```python
async def auto_flight(drone):
    await drone.connect()
    await drone.action.arm()
    await drone.action.takeoff()
    await asyncio.sleep(10)
    await drone.action.goto_location(lat, lon, alt, 0)
    await asyncio.sleep(30)
    await drone.action.land()
```

**4. MissionItem 参数：**

- 位置：latitude_deg, longitude_deg, relative_altitude_m
- 速度：speed_m_s
- 动作：camera_action, gimbal_pitch_deg
- 容差：acceptance_radius_m
- 盘旋：loiter_time_s

**5. 断线重连：**

```python
async def connect_with_retry(drone, address, max_retries=10):
    for i in range(max_retries):
        try:
            await drone.connect(address)
            async for state in drone.core.connection_state():
                if state.is_connected:
                    return True
        except Exception as e:
            print(f"连接失败: {e}")
            await asyncio.sleep(1)
    return False
```

</details>
