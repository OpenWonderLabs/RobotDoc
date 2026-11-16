# Onero A1 自研夹爪 CAN 协议

Onero A1 自研夹爪使用标准 11 位 CAN ID，通信波特率为 `1 Mbps`。

## 1. 术语与字节序

| 名称 | 含义 |
|---|---|
| `ESC_ID` | 设备接收 ID / 节点 ID |
| `MST_ID` | 设备反馈 ID |
| `0x7FF` | 设备管理、查询和参数读写的广播请求 ID |

多字节字段采用**小端序**。例如 `0xB562` 在数据区依次发送 `62 B5`。

出厂默认配置：

```text
ESC_ID = 0x008
MST_ID = 0x018
```

注意：夹爪位置控制帧 ID 为 `ESC_ID + 0x100`。在默认配置下，控制帧 ID 是 `0x108`，不是 `0x008`。

## 2. CAN ID 总览

| CAN ID | 方向 | 用途 |
|---|---|---|
| `0x7FF` | Host → Device | 获取/修改 ID、设备查询和参数读写 |
| `ESC_ID` | Host → Device | 校零、使能和失能 |
| `ESC_ID + 0x100` | Host → Device | `GRIP_CONTROL` 夹爪位置、速度限制和夹力限制 |
| `MST_ID` | Device → Host | 设备管理应答、状态和参数应答 |

默认 ID 下：

| 用途 | CAN ID |
|---|---:|
| 设备接收 | `0x008` |
| 夹爪控制 | `0x108` |
| 常规反馈 | `0x018` |

## 3. 设备管理

### 3.1 获取 ID

请求与应答均使用 CAN ID `0x7FF`、DLC 4：

```text
请求：55 00 00 AA
应答：55 ESC_ID MST_ID AA
```

ID 字段在该帧中为单字节，因此本命令仅能表达 `0x00..0xFF`。

### 3.2 修改 ID

请求使用 CAN ID `0x7FF`、DLC 4：

```text
AA NEW_ESC_ID NEW_MST_ID 55
```

保存后重启生效。协议建议立即重启，避免后续参数保存覆盖 Flash。

### 3.3 零点校准

发送至 `ESC_ID`、DLC 8：

```text
FF FF FF FF FF FF FF FA
```

### 3.4 电机使能

发送至 `ESC_ID`、DLC 8：

```text
FF FF FF FF FF FF FF FC
```

### 3.5 电机失能

发送至 `ESC_ID`、DLC 8：

```text
FF FF FF FF FF FF FF FD
```

## 4. 夹爪控制与反馈

### 4.1 `GRIP_CONTROL` 控制帧

控制帧发送至 `ESC_ID + 0x100`，DLC 8：

| 字节 | 字段 | 类型 | 含义 |
|---|---|---|---|
| D0 | `PositionRaw` | `uint8` | 目标开口宽度；`0x00` 全闭，`0xFF` 全开 |
| D1 | `VelocityLimitRaw` | `uint8` | 最大速度限制，不是目标速度 |
| D2 | `ForceLimitRaw` | `uint8` | 最大夹力限制 |
| D3-D7 | `Reserved` | `uint8[5]` | 当前固件忽略，建议填 `0x00` |

三项均取量程一半的示例：

```text
CAN ID: ESC_ID + 0x100 = 0x108
DLC:    8
DATA:   80 80 80 00 00 00 00 00
```

夹爪开度使用归一化值：

```text
0.0 = 全闭 → PositionRaw 0x00
1.0 = 全开 → PositionRaw 0xFF
```

量化为 8 位，存在 ±1 LSB 误差。

### 4.2 `GRIP_CONTROL` 反馈帧

反馈帧由 `MST_ID` 返回。

| 字节 | 字段 | 类型 | 含义 |
|---|---|---|---|
| D0 | `Status` | `uint8` | 状态码，不与 `ESC_ID` 拼接 |
| D1 | `PositionRaw` | `uint8` | 当前开口宽度，映射同控制帧 D0 |
| D2 | `VelocityRaw` | `uint8` | 输出轴速度绝对值 |
| D3 | `ForceRaw` | `uint8` | 当前夹力绝对值 |
| D4 | `MosTemperature` | `int8` | MOS 温度，单位 ℃ |
| D5 | `RotorTemperature` | `int8` | 电机线圈温度，单位 ℃ |
| D6-D7 | `Reserved` | `uint8[2]` | 固定 `0x00` |

速度换算：

```text
VelocityAbs = VelocityRaw × fMaxSpd / 255
```

超过 `VelocityMax` 时钳位为 `0xFF`。夹力物理量换算需要固件量程参数，原文未给出完整公式。

控制帧触发的同步反馈通常反映**发送控制帧之前**的状态，不能作为新目标已执行完毕的证明。若上层需要动作完成语义，应继续轮询并依据位置、速度、状态及超时判断。

### 4.3 状态码

| 值 | 含义 |
|---:|---|
| 0 | 失能且无故障 |
| 1 | 使能 |
| 7 | 张开校准到位开关检测失败 |
| 8 | 过压 |
| 9 | 欠压 |
| 10 | 过流 |
| 11 | MOS 过温 |
| 12 | 电机线圈过温 |
| 13 | CAN 通信超时 |
| 14 | 过载 |
| 15 | 编码器异常 |

## 5. 参数读写

参数命令发送至 `0x7FF`，应答由 `MST_ID` 返回。请求和应答的通用结构：

```text
ESC_L ESC_H CMD RID DATA0 DATA1 DATA2 DATA3
```

前两个字节是小端 16 位 `ESC_ID`，`CMD` 位于 D2，寄存器地址 `RID` 位于 D3，D4-D7 是小端 4 字节值。

| 命令 | 名称 | 说明 |
|---:|---|---|
| `0x33` | `READ_PARAM` | 读取 RID，应答返回 4 字节值 |
| `0x55` | `WRITE_PARAM` | 写 RW 寄存器，应答回显实际值 |
| `0xAA` | `SAVE_PARAM` | D3=`0x01` 时保存全部参数到 Flash |
| `0xAF` | `RESET_PARAM` | 恢复出厂参数，保留 `ESC_ID`/`MST_ID` |

写入约束：

- float 拒绝 NaN、Inf 和超出约 ±`3.0e38` 的值；
- `Gr`、`VMAX`、`TMAX`、`PMAX`、`MAX_SPD` 必须严格大于 0；
- RO 寄存器写入会被忽略。

### 5.1 寄存器表

| RID | 名称 | 访问 | 类型 | 范围/说明 |
|---:|---|---|---|---|
| 0 | `UV_Value` | RW | float | `(10.0, 3.4e38]` |
| 2 | `OT_Value` | RW | float | `[80.0, 200)` |
| 3 | `OC_Value` | RW | float | `(0.0, 8.0)` |
| 4 | `ACC` | RW | float | 电机轴加速度，`(0, 3.4e38)` |
| 5 | `DEC` | RW | float | 电机轴减速度，`(0, 3.4e38)` |
| 6 | `MAX_SPD` | RW | float | 最大速度，`(0, 3.4e38]` |
| 7 | `MST_ID` | RW | uint32 | `[0, 0x7FF]` |
| 8 | `ESC_ID` | RW | uint32 | `[0, 0x7FF]` |
| 10 | `CTRL_MODE` | RW | uint32 | `[1, 4]` |
| 14 | `sw_ver` | RO | uint32 | 软件版本号 |
| 16 | `NPP` | RO | uint32 | 电机极对数 |
| 17 | `Rs` | RO | float | 电机相电阻 |
| 18 | `Ls` | RO | float | 电机相电感 |
| 19 | `Flux` | RO | float | 电机磁链值 |
| 20 | `Gr` | RO | float | 齿轮减速比 |
| 21 | `PMAX` | RW | float | 位置映射范围，必须大于 0 |
| 22 | `VMAX` | RW | float | 速度映射范围，必须大于 0 |
| 23 | `TMAX` | RW | float | 扭矩映射范围，必须大于 0 |
| 25 | `KP_ASR` | RW | float | 速度环 Kp，`[0, 3.4e38]` |
| 26 | `KI_ASR` | RW | float | 速度环 Ki，`[0, 3.4e38]` |
| 27 | `KP_APR` | RW | float | 位置环 Kp，`[0, 3.4e38]` |
| 28 | `KI_APR` | RW | float | 位置环 Ki，`[0, 3.4e38]` |
| 29 | `OV_Value` | RW | float | 过压保护，`[0, 60]` |

## 6. 两种控制方式

夹爪固件同时提供两种控制方式，它们属于同一协议体系，控制协议没有发生变化：

### 6.1 电机级 MIT 控制

通过 `ESC_ID` 直接发送位置、速度、Kp、Kd 和力矩等电机控制字段，反馈由 `MST_ID` 返回。校零、使能和失能也使用 `ESC_ID`。

该方式适合底层调试、标定或需要直接控制电机参数的高级应用。

### 6.2 产品级 `GRIP_CONTROL`

通过 `ESC_ID + 0x100` 发送归一化开度、速度限制和夹力限制，由夹爪固件完成底层电机控制。

该方式更接近夹爪业务语义，适合普通开合和抓取应用。同步反馈通常反映命令执行前的状态，上层仍需轮询判断动作完成。

### 6.3 使用约束

- 两种方式均为固件支持的有效实现，不表示协议版本不同；
- 同一控制周期内不应由不同进程或控制器并发发送两种目标，以免后发命令覆盖前一目标；
- 电机级力矩字段与产品级夹力限制含义不同，不能直接等价换算；
- 反馈必须按对应控制方式的帧格式解析；
- 驱动应明确选择一种控制方式，并按对应格式解析反馈。

## 7. 实现安全要求

- 参数写入、修改 ID、保存参数和恢复出厂设置属于高风险维护操作，不应放进普通运动 API；
- 修改 ID 后应避免同总线上出现 ID 冲突；
- 夹爪控制的同步反馈是旧状态，上层不得据此立即宣告动作完成；
- 最大速度和最大夹力首次使用时应从较低值开始，并限制人员接近夹持区域。
