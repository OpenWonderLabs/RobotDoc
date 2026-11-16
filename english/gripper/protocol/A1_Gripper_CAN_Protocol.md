# Onero A1 Gripper CAN Protocol

This document summarizes the Onero A1 self-developed gripper CAN protocol. CAN uses standard 11-bit identifiers at `1 Mbps`. Multi-byte fields use little-endian order.

## 1. Identifiers

| Name | Default | Purpose |
|---|---:|---|
| `ESC_ID` | `0x008` | Device receive / motor maintenance ID |
| `MST_ID` | `0x018` | Device response ID |
| `0x7FF` | fixed | Device management and parameter access |
| `ESC_ID + 0x100` | `0x108` | Product-level gripper control |

The gripper protocol has two complementary control layers:

- motor-level maintenance frames on `ESC_ID` for zeroing, enable, and disable;
- product-level `GRIP_CONTROL` on `ESC_ID + 0x100` for opening, velocity limit, and force limit.

## 2. Device management

### Read IDs

Send and receive on `0x7FF`, DLC 4:

```text
request:  55 00 00 AA
response: 55 ESC_ID MST_ID AA
```

### Change IDs

Send on `0x7FF`, DLC 4:

```text
AA NEW_ESC_ID NEW_MST_ID 55
```

The new IDs take effect after reboot. Reboot immediately to avoid later parameter-save operations overwriting Flash.

### Zero, enable, and disable

Send to `ESC_ID`, DLC 8:

```text
zero:    FF FF FF FF FF FF FF FA
enable:  FF FF FF FF FF FF FF FC
disable: FF FF FF FF FF FF FF FD
```

## 3. Gripper control

Send `GRIP_CONTROL` to `ESC_ID + 0x100`, DLC 8:

| Byte | Field | Meaning |
|---|---|---|
| D0 | `PositionRaw` | `0x00` fully closed, `0xFF` fully open |
| D1 | `VelocityLimitRaw` | maximum velocity limit, not target velocity |
| D2 | `ForceLimitRaw` | maximum gripping-force limit |
| D3-D7 | reserved | send `0x00` |

Example at half scale:

```text
CAN ID: 0x108
DATA:   80 80 80 00 00 00 00 00
```

The public normalized opening convention is:

```text
0.0 = fully closed
1.0 = fully open
```

### Feedback

The normal response is reported on `MST_ID`:

| Byte | Field | Meaning |
|---|---|---|
| D0 | `Status` | state or fault code |
| D1 | `PositionRaw` | current opening |
| D2 | `VelocityRaw` | absolute output-shaft velocity |
| D3 | `ForceRaw` | absolute gripping force |
| D4 | `MosTemperature` | signed MOS temperature in °C |
| D5 | `RotorTemperature` | signed winding temperature in °C |
| D6-D7 | reserved | `0x00` |

A response returned together with a control frame normally describes the state **before** the new target is executed. Applications must poll and apply their own completion tolerance and timeout.

### Status codes

| Value | Meaning |
|---:|---|
| 0 | disabled, no fault |
| 1 | enabled |
| 7 | open-limit calibration switch failure |
| 8 | overvoltage |
| 9 | undervoltage |
| 10 | overcurrent |
| 11 | MOS overtemperature |
| 12 | winding overtemperature |
| 13 | CAN timeout |
| 14 | overload |
| 15 | encoder fault |

## 4. Parameter access

Requests use `0x7FF`; responses use `MST_ID`:

```text
ESC_L ESC_H CMD RID DATA0 DATA1 DATA2 DATA3
```

| Command | Name | Purpose |
|---:|---|---|
| `0x33` | `READ_PARAM` | read a register |
| `0x55` | `WRITE_PARAM` | write an RW register |
| `0xAA` | `SAVE_PARAM` | save all parameters when D3=`0x01` |
| `0xAF` | `RESET_PARAM` | factory reset while preserving IDs |

Important registers include undervoltage (0), overtemperature (2), overcurrent (3), acceleration (4), deceleration (5), maximum speed (6), response ID (7), receive ID (8), control mode (10), software version (14), position range (21), velocity range (22), torque range (23), speed-loop gains (25/26), position-loop gains (27/28), and overvoltage (29).

ID changes, parameter writes, saving, and factory reset are maintenance operations. They should not be exposed through ordinary motion commands without explicit safeguards.

## 5. Two control methods

The gripper firmware supports two control methods within the same protocol family. This does not indicate a protocol-version change.

### 5.1 Motor-level MIT control

Commands are sent directly to `ESC_ID` with motor position, velocity, Kp, Kd, and torque fields. Responses use `MST_ID`; zero, enable, and disable also use `ESC_ID`.

This method is suitable for low-level diagnostics, calibration, and advanced applications that require direct motor parameters.

### 5.2 Product-level `GRIP_CONTROL`

Commands are sent to `ESC_ID + 0x100` with normalized opening, velocity limit, and gripping-force limit. The gripper firmware performs the underlying motor control.

This method matches application-level gripper semantics and is suitable for ordinary open, close, and grasp operations. The immediate response normally describes the state before execution, so applications still need polling, tolerance, and timeout logic.

### 5.3 Constraints

- Both methods are valid firmware-supported implementations.
- Do not issue targets through both methods concurrently; a later command may overwrite an earlier target.
- Motor torque and product-level gripping-force limit are different quantities and are not directly interchangeable.
- Parse feedback according to the selected control method.
- A driver should explicitly select one control method and parse its corresponding feedback format.

## 6. Safety notes

- Start with conservative velocity and force limits.
- Avoid CAN-ID conflicts when changing IDs.
- Do not report motion completion from the immediate control response; poll state with tolerance and timeout.
