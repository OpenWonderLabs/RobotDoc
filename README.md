# onero 机械臂及关节电机资料

本目录的文档是机械臂或关节电机资料，按中英文分类存放。

## 目录结构

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
├── cad/                                 # CAD 图纸
│   ├── arm/                             # 机械臂（待补充）
│   └── actuator/                        # 关节模组
│       ├── XD5746-1_2d.pdf
│       ├── XD5746-1_3d.stp
│       ├── XD5746-2_2d.pdf
│       ├── XD5746-2_3d.stp
│       ├── XD5757-1_2d.pdf
│       ├── XD5757-1_3d.stp
│       ├── XD5757-2_2d.pdf
│       └── XD5757-2_3d.stp
│
└── tools/                               # 调试上位机软件
    └── motor_debug_tool_v1.0.2.zip
```

## 机械臂开发工程

- C/C++、Python 工程：https://github.com/OpenWonderLabs/ArmApi
- ROS2 工程：https://github.com/OpenWonderLabs/ArmRos
