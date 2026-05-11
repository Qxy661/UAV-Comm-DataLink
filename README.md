# 无人机通信与数据链 — 从MAVLink协议到5G网联无人机

> 面向无人机飞控自动化学生的系统性通信知识体系

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Language](https://img.shields.io/badge/语言-中文-blue.svg)]()
[![MAVLink](https://img.shields.io/badge/MAVLink-v2.0-green.svg)]()

---

## 项目简介

本项目是一套完整的无人机（UAV）通信与数据链技术教学文档，专为无人机飞控自动化方向的学生和工程师设计。文档从无线通信基础理论出发，系统讲解 MAVLink 协议、数据链设计、遥测可视化、4G/5G 网联无人机等核心技术，并附带论文导读和前沿技术追踪。

**核心理念：** 理论与实践并重，每一章都包含可运行的代码示例和动手实验。

---

## 目标读者

| 读者类型 | 适合内容 | 预计学习周期 |
|---------|---------|------------|
| 无人机飞控方向本科生 | 第0-3章全部，第4-5章选读 | 8-10 周 |
| 通信工程方向研究生 | 第0-2章快速过，第3-6章重点 | 6-8 周 |
| 无人机开发工程师 | 第1-3章参考，第4-5章实践 | 按需查阅 |
| 竞赛/项目团队 | 全部文档，重点第2-4章 | 4-6 周（集中学习） |

---

## 前置知识

```
必备：
├── C/C++ 或 Python 编程基础
├── 基本电路与信号处理概念
└── 无人机飞行控制基本原理（PID、姿态控制）

推荐：
├── Linux 命令行操作
├── 嵌入式开发经验（STM32/ESP32）
└── 基本的线性代数与概率论
```

---

## 学习路线图

| 阶段 | 主题 | 核心内容 | 动手项目 | 预计时间 |
|------|------|---------|---------|---------|
| **阶段 0** | [导读与学习路线](docs/00-导读与学习路线.md) | 知识地图、学习方法、环境搭建 | 搭建开发环境 | 1 天 |
| **阶段 1** | [通信基础理论](docs/01-通信基础理论/) | 无线传播、调制编码、天线射频、协议栈、频谱管理 | 链路预算计算工具 | 2 周 |
| **阶段 2** | [MAVLink 协议](docs/02-MAVLink协议/) | 协议架构、消息类型、编程实现、任务/参数协议、安全扩展 | Python MAVLink 通信程序 | 2 周 |
| **阶段 3** | [数据链设计](docs/03-数据链设计/) | 数据链架构、数传电台、图传系统、天线优化、多机通信 | 数传电台配置与测试 | 2 周 |
| **阶段 4** | [遥测与可视化](docs/04-遥测与可视化/) | QGroundControl、MAVSDK、MissionPlanner、自定义 GCS、数据分析 | 自定义地面站开发 | 2 周 |
| **阶段 5** | [4G/5G 网联无人机](docs/05-4G-5G与网联无人机/) | 蜂窝网络、LTE 链路、5G URLLC、边缘计算、物联网集群 | LTE 遥测链路搭建 | 2 周 |
| **阶段 6** | [论文导读与前沿](docs/06-论文导读与前沿/) | 通信论文、网联无人机论文、多机通信论文 | 论文复现实验 | 持续 |

---

## 关键技术速览

| 技术领域 | 核心协议/标准 | 典型硬件 | 应用场景 |
|---------|-------------|---------|---------|
| 飞控通信 | MAVLink v2 | Pixhawk、CubeOrange | 飞控-地面站通信 |
| 数传电台 | SiK/MAVLink Radio | RFD900x、SiK Telemetry | 近距遥测（<40km） |
| 图传系统 | DJI O3/HDZero | DJI Vista、HDZero VTX | 实时视频回传 |
| 4G 遥测 | LTE Cat-1/Cat-M | SIM7600、EC20 | 超视距遥测 |
| 5G 网联 | 5G NR URLLC | 5G CPE/模组 | 低延迟控制、边缘计算 |
| 多机通信 | DDS/MQTT/mesh | ESP-NOW、nRF24L01+ | 无人机集群编队 |
| 地面站 | QGroundControl/MissionPlanner | PC/平板 | 任务规划与监控 |

---

## 仓库结构

```
UAV-Comm-DataLink/
├── README.md                          # 本文件
├── CONTRIBUTING.md                    # 贡献指南
├── LICENSE                            # MIT 许可证
├── docs/
│   ├── 00-导读与学习路线.md
│   ├── 01-通信基础理论/
│   │   ├── 01-无线通信基础.md
│   │   ├── 02-调制与编码技术.md
│   │   ├── 03-天线与射频基础.md
│   │   ├── 04-通信协议栈.md
│   │   └── 05-频谱管理与干扰.md
│   ├── 02-MAVLink协议/
│   │   ├── 01-MAVLink协议架构.md
│   │   ├── 02-核心消息类型.md
│   │   ├── 03-MAVLink通信编程.md
│   │   ├── 04-任务协议与参数协议.md
│   │   └── 05-MAVLink安全与扩展.md
│   ├── 03-数据链设计/
│   │   ├── 01-数据链架构.md
│   │   ├── 02-数传电台选型.md
│   │   ├── 03-图传系统设计.md
│   │   ├── 04-天线与链路优化.md
│   │   └── 05-网络拓扑与多机通信.md
│   ├── 04-遥测与可视化/
│   │   ├── 01-QGroundControl深度使用.md
│   │   ├── 02-MAVSDK开发.md
│   │   ├── 03-MissionPlanner高级功能.md
│   │   ├── 04-自定义地面站开发.md
│   │   └── 05-飞行数据记录与分析.md
│   ├── 05-4G-5G与网联无人机/
│   │   ├── 01-蜂窝网络与无人机.md
│   │   ├── 02-4G-LTE通信链路.md
│   │   ├── 03-5G-NR与URLLC.md
│   │   ├── 04-边缘计算与云协同.md
│   │   └── 05-物联网与无人机集群.md
│   └── 06-论文导读与前沿/
│       ├── 01-无人机通信论文.md
│       ├── 02-网联无人机论文.md
│       └── 03-多机通信论文.md
├── mindmaps/
│   ├── mavlink-protocol-map.md
│   ├── data-link-architecture.md
│   └── comm-tech-comparison.md
└── references/
    ├── paper-list.md
    └── repo-annotations.md
```

---

## 快速开始

### 1. 克隆仓库

```bash
git clone https://github.com/your-username/UAV-Comm-DataLink.git
cd UAV-Comm-DataLink
```

### 2. 推荐阅读工具

- **VS Code** + Markdown Preview Enhanced 插件（支持 Mermaid 图表渲染）
- **Typora** — 所见即所得 Markdown 编辑器
- **GitHub/GitLab** — 在线浏览，自动渲染 Mermaid

### 3. 环境准备（动手实验用）

```bash
# Python 环境
pip install pymavlink mavsdk pyserial

# MAVLink 消息定义
git clone https://github.com/mavlink/mavlink.git

# QGroundControl（地面站）
# 下载地址：https://qgroundcontrol.com/downloads/
```

---

## 核心参考资源

| 资源 | 链接 | 说明 |
|------|------|------|
| MAVLink 官方文档 | https://mavlink.io/ | 协议规范与消息定义 |
| ArduPilot 文档 | https://ardupilot.org/ardupilot/ | 飞控固件文档 |
| PX4 开发者指南 | https://dev.px4.io/ | PX4 飞控开发文档 |
| MAVSDK 文档 | https://mavsdk.mavlink.io/ | MAVSDK API 参考 |
| QGroundControl | https://qgroundcontrol.com/ | 地面站软件 |
| 3GPP UAV 标准 | https://www.3gpp.org/ | 5G 网联无人机标准 |

---

## 贡献

欢迎提交 Issue 和 Pull Request！请先阅读 [CONTRIBUTING.md](CONTRIBUTING.md)。

---

## 相关项目

本项目是 [Qxy661](https://github.com/Qxy661) 无人机教学文档系列之一：

| 项目 | 说明 | GitHub |
|------|------|--------|
| Simulink-UAV-Dynamics-Sim | Simulink无人机动力学仿真+PX4对接 | [Qxy661/Simulink-UAV-Dynamics-Sim](https://github.com/Qxy661/Simulink-UAV-Dynamics-Sim) |
| UAV-Control-Theory | 飞行控制理论 | [Qxy661/UAV-Control-Theory](https://github.com/Qxy661/UAV-Control-Theory) |
| RL-Autonomous-Flight | 强化学习自主飞行 | [Qxy661/RL-Autonomous-Flight](https://github.com/Qxy661/RL-Autonomous-Flight) |
| LLM-Driven-UAV | LLM驱动的无人机系统 | [Qxy661/LLM-Driven-UAV](https://github.com/Qxy661/LLM-Driven-UAV) |

## 许可证

本项目采用 [MIT License](LICENSE) 开源许可证。

---

> **提示：** 本文档中的 Mermaid 图表需要在支持 Mermaid 渲染的 Markdown 查看器中打开（如 VS Code + Mermaid 插件、GitHub 在线浏览等）。
