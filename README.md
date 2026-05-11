# onero Robotic Arm & Actuator Documentation

This repository contains the official documentation, CAD drawings, and debugging tools for the **onero Robotic Arm A1** and its **actuator modules** (XD5746 / XD5757), developed by OneRobotics (Shenzhen) Co., Ltd. All documents are provided in both English and Chinese.

本仓库包含 **onero A1 机械臂**及其**关节模组**（XD5746 / XD5757）的产品文档、CAD 图纸和调试工具，由卧安机器人（深圳）股份有限公司开发。文档按中英文分类存放。

## Product Overview / 产品简介

**onero Robotic Arm A1** is a 7-DOF (7 degrees of freedom) modular robotic arm designed for education, research, and industrial automation, with a maximum payload of 3 kg. It is available in three configurations: A1L (left arm), A1R (right arm), and A1S (dual arm).

**onero A1 机械臂**是一款 7 自由度模块化机械臂，适用于教育科研和工业自动化场景，最大负载 3 kg。提供三种型号：A1L（左臂）、A1R（右臂）和 A1S（双臂）。

**Actuator Modules / 关节模组：**

| Model | Rated Torque | Gear Ratio | Dimensions | Weight |
|-------|-------------|------------|------------|--------|
| XD5746-1 (radial cable) | 3 N.m | 10:1 | Φ57 × 46 mm | 320 g |
| XD5746-2 (axial cable) | 3 N.m | 10:1 | Φ57 × 46 mm | 320 g |
| XD5757-1 (radial cable) | 15 N.m | 48.19:1 | Φ57 × 56.5 mm | 390 g |
| XD5757-2 (axial cable) | 15 N.m | 48.19:1 | Φ57 × 56.5 mm | 390 g |

Key features: 21-bit dual encoder, CAN 2.0 communication (1 Mbps), 20–55 V power input, position / velocity / torque / MIT hybrid control modes.

主要特性：21 位双编码器、CAN 2.0 通信（1 Mbps）、20–55 V 宽电压供电、支持位置/速度/力矩/MIT 混合控制模式。

## Directory Structure / 目录结构

```
├── zh/                                  # 中文文档
│   ├── arm/                             # 机械臂
│   │   ├── A1_产品说明书.pdf
│   │   └── A1_快速入门手册.pdf
│   ├── actuator/                        # 关节模组
│   │   ├── 关节说明书.pdf
│   │   └── 通信协议.pdf
│   └── tools/                           # 调试工具说明
│       └── 调试工具使用说明.pdf
│
├── en/                                  # English docs
│   ├── arm/                             # Robotic Arm
│   │   ├── A1_Product_Manual.pdf
│   │   └── A1_User_Guide.pdf
│   ├── actuator/                        # Actuator
│   │   ├── Actuator_User_Guide.pdf
│   │   └── Actuator_Communication_Protocol.pdf
│   └── tools/                           # Debugging Tool
│       └── Debug_Tool_Quick_Guide.pdf
│
├── cad/                                 # CAD drawings / CAD 图纸
│   ├── arm/                             # Robotic Arm (TBD / 待补充)
│   └── actuator/                        # Actuator / 关节模组
│       ├── XD5746-1_2d.pdf
│       ├── XD5746-1_3d.stp
│       ├── XD5746-2_2d.pdf
│       ├── XD5746-2_3d.stp
│       ├── XD5757-1_2d.pdf
│       ├── XD5757-1_3d.stp
│       ├── XD5757-2_2d.pdf
│       └── XD5757-2_3d.stp
│
└── tools/                               # Debug software / 调试上位机软件
    └── motor_debug_tool_v1.0.2.zip
```

## Development Resources / 开发工程

- C/C++, Python SDK: https://github.com/OpenWonderLabs/ArmApi
- ROS2: https://github.com/OpenWonderLabs/ArmRos
