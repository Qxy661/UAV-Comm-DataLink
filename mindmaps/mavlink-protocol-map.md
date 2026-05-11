# MAVLink 协议知识图谱

---

## 1. MAVLink 协议层次

```
MAVLink 协议体系
│
├── 协议版本
│   ├── v1 (2009)
│   │   ├── 头部: 8 字节
│   │   ├── 最大消息 ID: 255
│   │   └── 无签名
│   │
│   └── v2 (2013)
│       ├── 头部: 10 字节
│       ├── 最大消息 ID: 16M
│       ├── 消息签名 (可选)
│       └── 消息扩展
│
├── 消息格式
│   ├── 起始标记 (STX)
│   ├── 载荷长度 (LEN)
│   ├── 序列号 (SEQ)
│   ├── 系统 ID (SYS)
│   ├── 组件 ID (COMP)
│   ├── 消息 ID (MSG)
│   ├── 载荷 (PAYLOAD)
│   ├── CRC
│   └── 签名 (可选)
│
├── 消息类型
│   ├── 系统消息 (0-99)
│   │   ├── HEARTBEAT (0)
│   │   ├── SYS_STATUS (1)
│   │   └── PROTOCOL_VERSION (300)
│   │
│   ├── 飞行数据 (100-299)
│   │   ├── ATTITUDE (30)
│   │   ├── GPS_RAW_INT (24)
│   │   ├── GLOBAL_POSITION_INT (33)
│   │   └── RC_CHANNELS (65)
│   │
│   ├── 命令消息 (300-499)
│   │   ├── COMMAND_LONG (76)
│   │   ├── COMMAND_INT (75)
│   │   └── COMMAND_ACK (77)
│   │
│   ├── 任务消息 (400-499)
│   │   ├── MISSION_ITEM (39)
│   │   ├── MISSION_REQUEST (40)
│   │   ├── MISSION_COUNT (44)
│   │   └── MISSION_ACK (47)
│   │
│   └── 参数消息 (500-599)
│       ├── PARAM_REQUEST_READ (20)
│       ├── PARAM_REQUEST_LIST (21)
│       ├── PARAM_VALUE (22)
│       └── PARAM_SET (23)
│
└── 协议扩展
    ├── 方言 (Dialects)
    │   ├── common.xml
    │   ├── ardupilotmega.xml
    │   ├── standard.xml
    │   └── development.xml
    │
    ├── 签名机制
    │   ├── HMAC-SHA256
    │   ├── 32 字节密钥
    │   └── 48 位时间戳
    │
    └── 自定义消息
        ├── ID 范围: 60000-69999
        └── 用户定义
```

---

## 2. 消息分类树

```
MAVLink 消息分类
│
├── 通用消息 (Common)
│   ├── 心跳与状态
│   │   ├── HEARTBEAT
│   │   ├── SYS_STATUS
│   │   └── SYSTEM_TIME
│   │
│   ├── 命令
│   │   ├── COMMAND_LONG
│   │   ├── COMMAND_INT
│   │   └── COMMAND_ACK
│   │
│   └── 任务
│       ├── MISSION_ITEM
│       ├── MISSION_REQUEST
│       └── MISSION_COUNT
│
├── 飞行数据
│   ├── 姿态
│   │   ├── ATTITUDE
│   │   ├── ATTITUDE_QUATERNION
│   │   └── ATTITUDE_TARGET
│   │
│   ├── 位置
│   │   ├── GPS_RAW_INT
│   │   ├── GLOBAL_POSITION_INT
│   │   └── LOCAL_POSITION_NED
│   │
│   └── 遥控
│       ├── RC_CHANNELS
│       ├── RC_CHANNELS_RAW
│       └── RC_CHANNELS_OVERRIDE
│
├── 系统信息
│   ├── 传感器
│   │   ├── RAW_IMU
│   │   ├── SCALED_IMU
│   │   └── SENSOR_OFFSETS
│   │
│   ├── 电源
│   │   ├── BATTERY_STATUS
│   │   └── POWER_STATUS
│   │
│   └── 通信
│       ├── RADIO_STATUS
│       └── MEMINFO
│
└── 扩展消息 (ArduPilot)
    ├── 飞控状态
    │   ├── AHRS
    │   ├── AHRS2
│   │   └── AHRS3
│   │
│   ├── 传感器
│   │   ├── EKF_STATUS_REPORT
│   │   └── VIBRATION
│   │
│   └── 调试
│       ├── DEBUG
│       └── DEBUG_VECT
```

---

## 3. 协议栈映射

```
MAVLink 在通信协议栈中的位置
│
├── 应用层
│   ├── MAVLink 协议
│   │   ├── 消息序列化
│   │   ├── CRC 校验
│   │   └── 签名验证
│   │
│   └── 应用逻辑
│       ├── 飞行控制
│       ├── 任务管理
│       └── 参数配置
│
├── 传输层
│   ├── UDP (实时数据)
│   │   ├── 低延迟
│   │   └── 允许丢包
│   │
│   ├── TCP (可靠传输)
│   │   ├── 任务上传
│   │   └── 参数读写
│   │
│   └── Serial (串口)
│       ├── 数传电台
│       └── USB 连接
│
├── 网络层
│   ├── IP (4G/5G)
│   │   ├── IPv4/IPv6
│   │   └── 路由
│   │
│   └── 点对点 (数传)
│       ├── 无网络层
│       └── 直接传输
│
├── 链路层
│   ├── HDLC (数传)
│   ├── 802.11 (Wi-Fi)
│   └── LTE MAC (4G)
│
└── 物理层
    ├── GFSK (数传)
    ├── OFDM (Wi-Fi/LTE)
    └── 扩频 (LoRa)
```

---

## 4. 编程接口

```
MAVLink 编程接口
│
├── Python (pymavlink)
│   ├── 连接管理
│   │   ├── mavutil.mavlink_connection()
│   │   ├── wait_heartbeat()
│   │   └── close()
│   │
│   ├── 消息接收
│   │   ├── recv_match()
│   │   ├── recv_msg()
│   │   └── messages.get()
│   │
│   ├── 消息发送
│   │   ├── mav.heartbeat_send()
│   │   ├── mav.command_long_send()
│   │   └── mav.mission_item_send()
│   │
│   └── 工具函数
│       ├── mavlink_crc()
│       └── calculate_crc_extra()
│
├── C/C++ (MAVLink C Library)
│   ├── 消息解析
│   │   ├── mavlink_parse_char()
│   │   ├── mavlink_msg_heartbeat_decode()
│   │   └── mavlink_msg_attitude_decode()
│   │
│   └── 消息打包
│       ├── mavlink_msg_heartbeat_pack()
│       ├── mavlink_msg_command_long_pack()
│       └── mavlink_msg_mission_item_pack()
│
├── MAVSDK
│   ├── Action
│   │   ├── arm()
│   │   ├── takeoff()
│   │   ├── land()
│   │   └── return_to_launch()
│   │
│   ├── Telemetry
│   │   ├── position()
│   │   ├── attitude()
│   │   ├── battery()
│   │   └── flight_mode()
│   │
│   └── Mission
│       ├── upload_mission()
│       ├── start_mission()
│       └── pause_mission()
│
└── MAVProxy
    ├── 模块系统
    │   ├── map
    │   ├── graph
│   │   └── console
│   │
│   └── 命令行
│       ├── arm throttle
│       ├── takeoff 10
│       └── mode rtl
```

---

## 5. 应用场景

```
MAVLink 应用场景
│
├── 地面站
│   ├── QGroundControl
│   │   ├── 任务规划
│   │   ├── 遥测监控
│   │   └── 参数配置
│   │
│   ├── MissionPlanner
│   │   ├── 高级任务
│   │   ├── 日志分析
│   │   └── 调参工具
│   │
│   └── 自定义 GCS
│       ├── Qt/QML
│       ├── Web GCS
│       └── 移动端
│
├── 自动化
│   ├── 任务自动化
│   │   ├── 自动巡检
│   │   ├── 航线飞行
│   │   └── 自动返航
│   │
│   ├── 数据采集
│   │   ├── 遥测记录
│   │   ├── 传感器数据
│   │   └── 飞行日志
│   │
│   └── 监控告警
│       ├── 电池告警
│       ├── GPS 丢失
│       └── 通信中断
│
├── 多机协同
│   ├── 集群管理
│   │   ├── 任务分配
│   │   ├── 状态监控
│   │   └── 故障处理
│   │
│   ├── 编队飞行
│   │   ├── 编队控制
│   │   ├── 位置同步
│   │   └── 避碰协调
│   │
│   └── 通信中继
│       ├── 多跳通信
│       ├── 路由转发
│       └── 网络扩展
│
└── 研发调试
    ├── 协议调试
    │   ├── 消息抓包
    │   ├── 协议分析
    │   └── 错误诊断
    │
    ├── 性能测试
    │   ├── 延迟测试
    │   ├── 吞吐量测试
    │   └── 可靠性测试
    │
    └── 固件开发
        ├── 新消息定义
        ├── 方言扩展
        └── 自定义功能
```
