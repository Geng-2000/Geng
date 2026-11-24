# 协议板 USB 接口应用说明
## 1 通信接口
USB 2.0 Full-Speed Device，CDC Class
>协议板为从机

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|---|---|---|---|
| `0` | `SOF` | 2 Byte | 帧头，固定为`0xB4`、`0x7B`，先发`0xB4`，后发`0x7B` |
| `2` | `MsgID` | 1 Byte | 由消息发起者维护的滚动计数器 |
| `3` | `MsgType` | 1 Byte | `0x00`= CMD<br>`0x01`= EVENT<br>`0x02`= ACK<br>`0x03…0xFF`= Reserved |
| `4` | `MsgCode` | 1 Byte | `MsgType = CMD`：MsgCode 表示命令码<br>`MsgType = EVENT`：MsgCode 表示事件码<br>`MsgType = ACK`：MsgCode 表示被确认的原始命令码/事件码 |
| `5` | `Flags/ACK_Flags` | 1 Byte | `MsgType != ACK`：该字段为 Flags<br>`MsgType = ACK`：该字段为 ACK_Flags |
| `6` | `DataLen` | 2 Byte | 业务数据长度，单位：byte，小端序 |
| `8` | `Data` | N Byte | 业务数据 |
| `8 + N` | `CRC16` | 2 Byte | 从 MsgID 到 Data 最后一个字节的 CRC-16/CCITT-FALSE 校验值，小端序 |

### 2.2 字段定义
#### 2.2.1 SOF 定义
帧头，固定为`0xB4`、`0x7B`，先发`0xB4`，后发`0x7B`。
用于在连续字节流中定位帧起点。消息接收者应将 SOF 用于拆包、粘包处理以及通信异常后的重新同步。

#### 2.2.2 MsgID 定义
- 主机和从机各自独立维护一个 MsgID 计数器
- 主机和从机开机或者重启后发送的第一条消息的 MsgID 必须为 `0x00`
- 发送新消息时，除 ACK 外，需要分配新的 MsgID，变化规则为 `MsgID++`，其中 `0xFF` 的下一值为 `0x00`
- 消息接收者需要能够正常处理非连续变化的 MsgID，以应对设备重启等特殊情况
- 设备重传同一条消息时，应保持 MsgID 不变，并将Flags.Retransmit 置1
- 消息接收者回复 ACK 时，ACK 中的 MsgID 应回填所回复消息的原始 MsgID

#### 2.2.3 MsgType 定义
| MsgType | Value | Direction | Description |
|---|---|:---:|---|
| `CMD` | `0x00` | Host → Device | 下发命令 |
| `EVENT` | `0x01` | Device → Host | 异步执行结果或者异步事件上报 |
| `ACK` | `0x02` | 双向 | 命令/事件接收、校验及受理结果 |
| `Reserved` | `0x03…0xFF` | NA | Reserved - 暂未启用 |

典型交互关系: 
```
方式1：主机下发 CMD，从机立即完成并 ACK
Host                                    Device
 │                                         │
 │──────── CMD ───────────────────────────>│
 │<─────────────────────────── ACK ────────│
```

```
方式2：主机下发 CMD，从机先 ACK 表示受理，完成后 EVENT 上报结果，主机再 ACK
Host                                    Device
 │                                         │
 │──────── CMD ───────────────────────────>│
 │<─────────────────────────── ACK ────────│
 │                                         │异步执行
 │<────────────────────────── EVENT ───────│
 │──────── ACK ───────────────────────────>│（Flags.ACK_Required = 1时，才需要回复ACK）
```

```
方式3：从机上报异步事件，主机 ACK
Host                                    Device
 │                                         │
 │<────────────────────────── EVENT ───────│
 │──────── ACK ───────────────────────────>│（Flags.ACK_Required = 1时，才需要回复ACK）
```

#### 2.2.4 MsgCode 定义
MsgCode 的具体含义由 MsgType 决定：
- `MsgType = CMD`：MsgCode 表示命令码，具体定义见后续章节
- `MsgType = EVENT`：MsgCode 表示事件码，具体定义见后续章节
- `MsgType = ACK`：MsgCode 表示被确认的原始命令码/事件码

示例：
```
主机下发 CMD：
Host → Device
MsgID = 0x06 //由主机维护的 MsgID
MsgType = CMD
MsgCode = CMD_XX

从机先 ACK 表示受理：
Device → Host
MsgID = 0x06 //回填原始 MsgID
MsgType = ACK
MsgCode = CMD_XX //回填原始 MsgCode

从机 EVENT 上报结果：
Device → Host
MsgID = 0x00 //由从机维护的 MsgID
MsgType = EVENT
MsgCode = EVENT_XX

主机 ACK：
Host → Device
MsgID = 0x00 //回填原始 MsgID
MsgType = ACK
MsgCode = EVENT_XX //回填原始 MsgCode
```

#### 2.2.5 Flags 定义（MsgType != ACK）
| Bit(s) | Field | Description |
|---|---|---|
| `B0` | `ACK_Required` | 当前消息是否需要对方回复ACK<br>0 = 不需要<br>1 = 需要 |
| `B1` | `Retransmit` | 当前消息是否为重传消息<br>0 = 否<br>1 = 是 |
| `B2…7` | `Reserved` | Reserved - 应设为0 |

(1) ACK_Required
- 主机必须将 CMD 中的 ACK_Required 置1
- 从机忽略 CMD 中的 ACK_Required，收到的所有 CMD 均回复 ACK
- 关键 Event 的 ACK_Required 为1，需要主机回复 ACK
- 非关键 Event 的 ACK_Required 为0，不需要主机回复 ACK

(2) Retransmit
- 非重传消息的 Retransmit 必须设0
- 重传消息的 Retransmit 必须置1，且应保持 MsgID 不变

#### 2.2.6 ACK_Flags 定义（MsgType = ACK）


#### 2.2.7 DataLen 定义
Data 字段的长度，单位为 byte，占用两个字节，小端序。

#### 2.2.8 Data 定义
业务数据，具体定义见后续章节。

#### 2.2.9 CRC16 定义
从 MsgID 到 Data 最后一个字节的 CRC-16/CCITT-FALSE 校验值，占用两个字节，小端序。

代码示例：
```c
uint16_t crc16(const uint8_t* data, uint16_t len) {
    uint16_t crc = 0xFFFF;
    for (uint16_t i = 0; i < len; i++) {
        crc ^= (uint16_t)data[i] << 8;
        for (uint8_t j = 0; j < 8; j++) {
            if (crc & 0x8000) crc = (crc << 1) ^ 0x1021;
            else crc <<= 1;
        }
    }
    return crc;
}
```
