# MAVLink 通信编程

> 预计阅读：35 分钟 | 前置知识：Python/C 编程基础、MAVLink 协议架构

> **相关文档：** [核心消息类型](./02-核心消息类型.md) | [MAVSDK 开发](../04-遥测与可视化/02-MAVSDK开发.md)

---

## 1. pymavlink 库入门

### 1.1 安装与配置

```bash
# 安装 pymavlink
pip install pymavlink

# 安装可选依赖
pip install pyserial    # 串口通信
pip install numpy       # 数据处理
```

### 1.2 建立连接

pymavlink 支持多种连接方式：

```python
from pymavlink import mavutil

# 方式1: UDP 连接（最常用）
conn = mavutil.mavlink_connection('udp:127.0.0.1:14550')

# 方式2: UDP 监听（等待飞控连接）
conn = mavutil.mavlink_connection('udp:0.0.0.0:14550')

# 方式3: TCP 连接
conn = mavutil.mavlink_connection('tcp:192.168.1.100:5760')

# 方式4: 串口连接
conn = mavutil.mavlink_connection('/dev/ttyUSB0', baud=57600)

# 方式5: Windows 串口
conn = mavutil.mavlink_connection('COM3', baud=57600)

# 方式6: UDP 输出（发送到指定地址）
conn = mavutil.mavlink_connection('udpout:192.168.1.200:14550')
```

### 1.3 等待心跳

```python
# 等待第一个心跳消息
print("等待心跳...")
conn.wait_heartbeat()
print(f"收到心跳! SYS={conn.target_system}, COMP={conn.target_component}")

# 带超时的心跳等待
import time
start = time.time()
while time.time() - start < 10:  # 10秒超时
    msg = conn.recv_match(type='HEARTBEAT', blocking=True, timeout=1)
    if msg:
        print(f"收到心跳: SYS={conn.target_system}")
        break
else:
    print("超时未收到心跳!")
```

---

## 2. 消息接收与解析

### 2.1 基本消息接收

```python
# 方式1: 接收任意消息
msg = conn.recv_match(blocking=True)
if msg:
    print(f"消息类型: {msg.get_type()}")
    print(f"消息内容: {msg}")

# 方式2: 接收特定类型消息
msg = conn.recv_match(type='ATTITUDE', blocking=True, timeout=3)
if msg:
    print(f"Roll: {msg.roll:.3f} rad")

# 方式3: 接收多个类型中的一个
msg = conn.recv_match(type=['ATTITUDE', 'GPS_RAW_INT'], blocking=True)

# 方式4: 非阻塞接收
msg = conn.recv_match(blocking=False)
if msg:
    print(f"收到: {msg.get_type()}")
else:
    print("没有待处理消息")
```

### 2.2 完整遥测接收程序

```python
#!/usr/bin/env python3
"""MAVLink 遥测接收示例"""

import math
import time
from pymavlink import mavutil

def connect(connection_string):
    """建立 MAVLink 连接"""
    print(f"连接到: {connection_string}")
    conn = mavutil.mavlink_connection(connection_string)
    conn.wait_heartbeat()
    print(f"已连接: SYS={conn.target_system}")
    return conn

def request_message_interval(conn, msg_id, interval_us):
    """请求消息以指定间隔发送"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_SET_MESSAGE_INTERVAL,
        0,
        msg_id,
        interval_us,
        0, 0, 0, 0, 0
    )

def handle_heartbeat(msg):
    """处理心跳消息"""
    mode_map = {
        0: 'STABILIZE', 2: 'ALT_HOLD', 3: 'AUTO',
        4: 'GUIDED', 5: 'LOITER', 6: 'RTL', 9: 'LAND'
    }
    mode = mode_map.get(msg.custom_mode, f"MODE_{msg.custom_mode}")
    print(f"模式: {mode} | 状态: {msg.system_status}")

def handle_attitude(msg):
    """处理姿态消息"""
    roll = math.degrees(msg.roll)
    pitch = math.degrees(msg.pitch)
    yaw = math.degrees(msg.yaw)
    print(f"姿态: R={roll:.1f}° P={pitch:.1f}° Y={yaw:.1f}°")

def handle_gps(msg):
    """处理 GPS 消息"""
    lat = msg.lat / 1e7
    lon = msg.lon / 1e7
    alt = msg.alt / 1000.0
    print(f"GPS: {lat:.7f}, {lon:.7f}, {alt:.1f}m, "
          f"Sats={msg.satellites_visible}")

def handle_battery(msg):
    """处理电池消息"""
    voltage = msg.voltages[0] / 1000.0
    print(f"电池: {voltage:.1f}V, {msg.battery_remaining}%")

def main():
    # 连接
    conn = connect('udp:127.0.0.1:14550')

    # 请求消息流
    request_message_interval(conn, 30, 100000)   # ATTITUDE 10Hz
    request_message_interval(conn, 24, 200000)   # GPS 5Hz
    request_message_interval(conn, 147, 1000000) # BATTERY 1Hz

    # 主循环
    handlers = {
        'HEARTBEAT': handle_heartbeat,
        'ATTITUDE': handle_attitude,
        'GPS_RAW_INT': handle_gps,
        'BATTERY_STATUS': handle_battery,
    }

    print("开始接收遥测数据...")
    try:
        while True:
            msg = conn.recv_match(blocking=True, timeout=1)
            if msg:
                handler = handlers.get(msg.get_type())
                if handler:
                    handler(msg)
            else:
                # 检查连接状态
                if time.time() - conn.last_message_time() > 5:
                    print("警告: 5秒未收到消息!")
    except KeyboardInterrupt:
        print("\n程序结束")
    finally:
        conn.close()

if __name__ == '__main__':
    main()
```

---

## 3. 消息发送

### 3.1 发送命令

```python
# 解锁飞控
def arm(conn):
    """解锁飞控"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_COMPONENT_ARM_DISARM,
        0,      # 确认
        1,      # param1: 1=解锁, 0=锁定
        0, 0, 0, 0, 0, 0
    )
    # 等待 ACK
    ack = conn.recv_match(type='COMMAND_ACK', blocking=True, timeout=3)
    if ack and ack.result == 0:
        print("解锁成功!")
        return True
    else:
        print(f"解锁失败: {ack.result if ack else '超时'}")
        return False

# 锁定飞控
def disarm(conn):
    """锁定飞控"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_COMPONENT_ARM_DISARM,
        0,
        0,  # param1: 0=锁定
        0, 0, 0, 0, 0, 0
    )

# 设置飞行模式
def set_mode(conn, mode_id):
    """设置飞行模式"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_DO_SET_MODE,
        0,
        1,          # param1: 1=自定义模式
        mode_id,    # param2: 模式 ID
        0, 0, 0, 0, 0
    )
```

### 3.2 发送航点命令

```python
def goto_location(conn, lat, lon, alt):
    """飞往指定位置"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_NAV_WAYPOINT,
        0,
        0,          # param1: 停留时间
        0,          # param2: 接受半径
        0,          # param3: 绕行半径
        0,          # param4: 绕行方向
        lat,        # param5: 纬度
        lon,        # param6: 经度
        alt         # param7: 高度
    )

def takeoff(conn, alt):
    """起飞到指定高度"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_NAV_TAKEOFF,
        0,
        0, 0, 0, 0,
        0, 0,
        alt
    )

def return_to_launch(conn):
    """返航"""
    conn.mav.command_long_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_CMD_NAV_RETURN_TO_LAUNCH,
        0,
        0, 0, 0, 0, 0, 0, 0
    )
```

### 3.3 发送 MAV_CMD_INT

```python
def goto_position_int(conn, lat, lon, alt):
    """使用 COMMAND_INT 发送位置命令"""
    conn.mav.command_int_send(
        conn.target_system,
        conn.target_component,
        mavutil.mavlink.MAV_FRAME_GLOBAL_RELATIVE_ALT_INT,
        mavutil.mavlink.MAV_CMD_NAV_WAYPOINT,
        0,          # current
        0,          # autocontinue
        0,          # param1
        0,          # param2
        0,          # param3
        0,          # param4
        int(lat * 1e7),  # x: 纬度 (degE7)
        int(lon * 1e7),  # y: 经度 (degE7)
        alt              # z: 高度 (m)
    )
```

---

## 4. 串口通信编程

### 4.1 基本串口连接

```python
import serial
from pymavlink import mavutil

# 连接到数传电台
conn = mavutil.mavlink_connection(
    '/dev/ttyUSB0',  # 串口设备
    baud=57600,       # 波特率
    source_system=255,  # 本机系统 ID（255=地面站）
    source_component=190  # 本机组件 ID（190=GCS）
)

# 等待飞控连接
conn.wait_heartbeat()
print(f"已连接到飞控 {conn.target_system}")
```

### 4.2 串口调试工具

```python
#!/usr/bin/env python3
"""MAVLink 串口调试工具"""

import sys
import time
from pymavlink import mavutil

def dump_messages(port, baud, duration=10):
    """打印所有接收到的 MAVLink 消息"""
    conn = mavutil.mavlink_connection(port, baud=baud)
    
    print(f"监听 {port} ({baud} baud) {duration}秒...")
    start = time.time()
    
    while time.time() - start < duration:
        msg = conn.recv_match(blocking=True, timeout=0.1)
        if msg:
            elapsed = time.time() - start
            print(f"[{elapsed:8.3f}] {msg.get_type():20s} "
                  f"SYS={msg.get_srcSystem()} COMP={msg.get_srcComponent()} "
                  f"MSG={msg.get_msgId()}")

if __name__ == '__main__':
    port = sys.argv[1] if len(sys.argv) > 1 else '/dev/ttyUSB0'
    baud = int(sys.argv[2]) if len(sys.argv) > 2 else 57600
    dump_messages(port, baud)
```

---

## 5. UDP/TCP 网络通信

### 5.1 UDP 通信

```python
import socket
from pymavlink import mavutil

# 创建 UDP 监听
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.bind(('0.0.0.0', 14550))
sock.settimeout(1.0)

print("监听 UDP 14550...")

# MAVLink 解析器
parser = mavutil.mavlink_connection('udp:0.0.0.0:14550')

while True:
    try:
        data, addr = sock.recvfrom(1024)
        # feed data to parser
        for byte in data:
            msg = parser.mav.parse_char(chr(byte))
            if msg:
                print(f"收到来自 {addr}: {msg.get_type()}")
    except socket.timeout:
        continue
    except KeyboardInterrupt:
        break

sock.close()
```

### 5.2 UDP 转发器

```python
#!/usr/bin/env python3
"""MAVLink UDP 转发器"""

import socket
import threading
from pymavlink import mavutil

class UDPForwarder:
    def __init__(self, listen_port, forward_host, forward_port):
        self.listen_port = listen_port
        self.forward_host = forward_host
        self.forward_port = forward_port
        self.running = False
        
    def start(self):
        self.running = True
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        self.sock.bind(('0.0.0.0', self.listen_port))
        self.sock.settimeout(1.0)
        
        self.forward_sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        
        print(f"转发器启动: 监听 {self.listen_port} → "
              f"{self.forward_host}:{self.forward_port}")
        
        self.thread = threading.Thread(target=self._run)
        self.thread.start()
        
    def _run(self):
        while self.running:
            try:
                data, addr = self.sock.recvfrom(1024)
                self.forward_sock.sendto(
                    data, (self.forward_host, self.forward_port)
                )
            except socket.timeout:
                continue
                
    def stop(self):
        self.running = False
        self.thread.join()
        self.sock.close()
        self.forward_sock.close()

# 使用示例
forwarder = UDPForwarder(14550, '192.168.1.100', 14551)
forwarder.start()

try:
    while True:
        pass
except KeyboardInterrupt:
    forwarder.stop()
```

---

## 6. C 语言 MAVLink 编程

### 6.1 C 语言基础示例

```c
/* MAVLink C 语言通信示例 */

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

/* MAVLink 头文件 */
#include "mavlink/common/mavlink.h"

#define BUFFER_SIZE 2048

int main() {
    /* 创建 UDP socket */
    int sock = socket(AF_INET, SOCK_DGRAM, 0);
    if (sock < 0) {
        perror("socket");
        return 1;
    }
    
    /* 绑定地址 */
    struct sockaddr_in addr;
    memset(&addr, 0, sizeof(addr));
    addr.sin_family = AF_INET;
    addr.sin_port = htons(14550);
    addr.sin_addr.s_addr = INADDR_ANY;
    
    if (bind(sock, (struct sockaddr*)&addr, sizeof(addr)) < 0) {
        perror("bind");
        close(sock);
        return 1;
    }
    
    printf("监听 UDP 14550...\n");
    
    /* MAVLink 解析器 */
    mavlink_message_t msg;
    mavlink_status_t status;
    
    uint8_t buffer[BUFFER_SIZE];
    
    while (1) {
        /* 接收数据 */
        struct sockaddr_in sender;
        socklen_t sender_len = sizeof(sender);
        ssize_t len = recvfrom(sock, buffer, BUFFER_SIZE, 0,
                              (struct sockaddr*)&sender, &sender_len);
        
        if (len < 0) {
            perror("recvfrom");
            continue;
        }
        
        /* 解析 MAVLink 消息 */
        for (ssize_t i = 0; i < len; i++) {
            if (mavlink_parse_char(MAVLINK_COMM_0, buffer[i], &msg, &status)) {
                /* 处理消息 */
                switch (msg.msgid) {
                    case MAVLINK_MSG_ID_HEARTBEAT: {
                        mavlink_heartbeat_t hb;
                        mavlink_msg_heartbeat_decode(&msg, &hb);
                        printf("HEARTBEAT: type=%d, autopilot=%d\n",
                               hb.type, hb.autopilot);
                        break;
                    }
                    case MAVLINK_MSG_ID_ATTITUDE: {
                        mavlink_attitude_t att;
                        mavlink_msg_attitude_decode(&msg, &att);
                        printf("ATTITUDE: roll=%.1f, pitch=%.1f, yaw=%.1f\n",
                               att.roll * 180.0 / 3.14159,
                               att.pitch * 180.0 / 3.14159,
                               att.yaw * 180.0 / 3.14159);
                        break;
                    }
                    default:
                        printf("MSG ID: %d\n", msg.msgid);
                        break;
                }
            }
        }
    }
    
    close(sock);
    return 0;
}
```

### 6.2 编译 C 示例

```bash
# 假设 mavlink 头文件在 /usr/local/include/mavlink/
gcc -o mavlink_example mavlink_example.c -I/usr/local/include

# 或使用 CMake
# CMakeLists.txt
# find_package(MAVLink REQUIRED)
# target_link_libraries(myapp MAVLink)
```

### 6.3 嵌入式 MAVLink 解析（STM32/Pixhawk）

以下示例针对 STM32 HAL 环境，适用于 Pixhawk 等飞控硬件的裸机或 RTOS 部署。

```c
// Embedded MAVLink parsing on STM32/Pixhawk
// Using MAVLink C header library (auto-generated from mavlink repo)

#include "mavlink/common/mavlink.h"

// UART receive callback - parse incoming MAVLink bytes
volatile mavlink_message_t msg;
volatile mavlink_status_t status;

void USART2_IRQHandler(void) {
    if (USART2->SR & USART_SR_RXNE) {
        uint8_t byte = USART2->DR;
        if (mavlink_parse_char(MAVLINK_COMM_0, byte, &msg, &status)) {
            // Message complete - dispatch by ID
            switch (msg.msgid) {
                case MAVLINK_MSG_ID_HEARTBEAT:
                    handle_heartbeat(&msg);
                    break;
                case MAVLINK_MSG_ID_COMMAND_LONG:
                    handle_command(&msg);
                    break;
                case MAVLINK_MSG_ID_SET_POSITION_TARGET_LOCAL_NED:
                    handle_position_target(&msg);
                    break;
            }
        }
    }
}

// Send attitude telemetry
void send_attitude(float roll, float pitch, float yaw,
                   float roll_rate, float pitch_rate, float yaw_rate) {
    mavlink_msg_attitude_pack(
        1,    // system ID
        1,    // component ID
        &msg,
        HAL_GetTick(),  // timestamp ms
        roll, pitch, yaw,
        roll_rate, pitch_rate, yaw_rate
    );
    uint8_t buf[MAVLINK_MAX_PACKET_LEN];
    uint16_t len = mavlink_msg_to_send_buffer(buf, &msg);
    HAL_UART_Transmit(&huart2, buf, len, 100);
}

// Heartbeat at 1Hz (call from timer interrupt)
void send_heartbeat(void) {
    mavlink_msg_heartbeat_pack(
        1, 1, &msg,
        MAV_TYPE_QUADROTOR,
        MAV_AUTOPILOT_PX4,
        MAV_MODE_FLAG_CUSTOM_MODE_ENABLED,
        custom_mode,    // e.g., PX4 offboard mode
        MAV_STATE_ACTIVE
    );
    uint8_t buf[MAVLINK_MAX_PACKET_LEN];
    uint16_t len = mavlink_msg_to_send_buffer(buf, &msg);
    HAL_UART_Transmit(&huart2, buf, len, 100);
}
```

> **安全提示：** 在生产环境中部署时，应启用 MAVLink 消息签名（Message Signing）以防止中间人攻击和命令注入。详见 [MAVLink 安全与扩展](./05-MAVLink安全与扩展.md)。

---

## 7. 常见问题与调试

### 7.1 连接问题排查

```python
# 连接调试检查清单
def debug_connection(port, baud):
    """调试连接问题"""
    print(f"=== 连接调试 ===")
    print(f"端口: {port}")
    print(f"波特率: {baud}")
    
    # 1. 检查串口是否存在
    import serial.tools.list_ports
    ports = [p.device for p in serial.tools.list_ports.comports()]
    print(f"可用串口: {ports}")
    
    if port not in ports:
        print(f"错误: {port} 不存在!")
        return
    
    # 2. 尝试打开串口
    try:
        ser = serial.Serial(port, baud, timeout=1)
        print(f"串口打开成功")
        ser.close()
    except Exception as e:
        print(f"串口打开失败: {e}")
        return
    
    # 3. 尝试 MAVLink 连接
    try:
        conn = mavutil.mavlink_connection(port, baud=baud)
        print("等待心跳...")
        conn.wait_heartbeat(timeout=10)
        print(f"连接成功! SYS={conn.target_system}")
    except Exception as e:
        print(f"MAVLink 连接失败: {e}")
```

### 7.2 消息丢失排查

```python
def monitor_packet_loss(conn, duration=60):
    """监控丢包率"""
    seq_last = None
    lost = 0
    received = 0
    
    start = time.time()
    while time.time() - start < duration:
        msg = conn.recv_match(blocking=True, timeout=1)
        if msg:
            received += 1
            seq = msg.get_seq()
            
            if seq_last is not None:
                expected = (seq_last + 1) % 256
                if seq != expected:
                    gap = (seq - expected) % 256
                    lost += gap
                    print(f"丢包: expected={expected}, got={seq}, gap={gap}")
            
            seq_last = seq
    
    total = received + lost
    loss_rate = lost / total * 100 if total > 0 else 0
    print(f"\n统计: received={received}, lost={lost}, "
          f"loss_rate={loss_rate:.2f}%")
```

### 7.3 常见错误码

| 错误 | 原因 | 解决方案 |
|------|------|---------|
| 无心跳 | 连接错误/飞控未运行 | 检查连接、确认飞控固件 |
| 消息乱码 | 波特率不匹配 | 检查波特率配置 |
| CRC 错误 | 数据损坏/版本不匹配 | 检查线缆、更新固件 |
| 超时 | 信号弱/干扰 | 检查天线、减少干扰 |
| 权限拒绝 | 串口权限不足 | sudo 或添加用户到 dialout 组 |

---

## 思考题

1. **pymavlink 的 `recv_match` 函数的 `blocking` 和 `timeout` 参数有什么作用？在什么场景下使用非阻塞模式？**

2. **为什么 MAVLink 消息需要序列号（SEQ）？如何利用序列号检测丢包？**

3. **编写一个 Python 程序，实现以下功能：连接飞控、解锁、起飞到 10m、等待 5 秒、降落、锁定。**

4. **在 C 语言中，`mavlink_parse_char` 函数是如何工作的？为什么需要逐字节解析？**

5. **比较串口连接和 UDP 连接在 MAVLink 通信中的优缺点。**

<details>
<summary>参考答案</summary>

**1. blocking 和 timeout 参数：**

- `blocking=True`：阻塞等待直到收到消息
- `blocking=False`：立即返回，没有消息则返回 None
- `timeout`：阻塞等待的最长时间（秒）

非阻塞模式适用于：实时控制系统（不能阻塞主循环）、多路复用（同时监听多个连接）、GUI 应用（不能阻塞 UI 线程）。

**2. 序列号的作用：**

- 检测丢包：序列号连续递增（0-255 循环），如果收到的序列号不连续，说明中间有丢包
- 检测乱序：如果收到的序列号比上一个还小，说明消息乱序
- 重传请求：知道哪些消息丢失后，可以请求重传

**3. 解锁-起飞-降落程序：**

```python
from pymavlink import mavutil
import time

conn = mavutil.mavlink_connection('udp:127.0.0.1:14550')
conn.wait_heartbeat()

# 解锁
conn.mav.command_long_send(conn.target_system, conn.target_component,
    400, 0, 1, 0, 0, 0, 0, 0, 0)
ack = conn.recv_match(type='COMMAND_ACK', blocking=True, timeout=3)

# 起飞到 10m
conn.mav.command_long_send(conn.target_system, conn.target_component,
    22, 0, 0, 0, 0, 0, 0, 0, 10)

# 等待 5 秒
time.sleep(5)

# 降落
conn.mav.command_long_send(conn.target_system, conn.target_component,
    21, 0, 0, 0, 0, 0, 0, 0, 0)

# 锁定
conn.mav.command_long_send(conn.target_system, conn.target_component,
    400, 0, 0, 0, 0, 0, 0, 0, 0)
```

**4. mavlink_parse_char 工作原理：**

MAVLink 是流式协议，数据通过串口逐字节到达。`mavlink_parse_char` 维护一个状态机，每收到一个字节就更新状态：
- 等待起始字节（0xFE 或 0xFD）
- 接收头部字段
- 接收载荷
- 接收并校验 CRC
- 完成一个完整消息

逐字节解析的原因：串口数据是流式的，一个消息可能被分成多个读取操作，也可能多个消息在一次读取中。

**5. 串口 vs UDP 对比：**

| 特性 | 串口 | UDP |
|------|------|-----|
| 连接方式 | 点对点 | 网络 |
| 延迟 | 低（<1ms） | 中（1-10ms） |
| 距离 | 短（数传电台覆盖） | 任意（网络覆盖） |
| 可靠性 | 高（无丢包） | 中（可能丢包） |
| 多客户端 | 不支持 | 支持 |
| 配置 | 简单 | 需要网络配置 |

</details>
