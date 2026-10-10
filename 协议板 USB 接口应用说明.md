# 协议板 USB 接口应用说明
## 1 通信接口
USB 2.0 Full-Speed Device，CDC Class
>协议板为从机

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|:---:|---|---|---|
| `0` | `SOF` | 2 Byte | 帧头，固定为`0xB4`、`0x7B`，先发`0xB4`，后发`0x7B` |
| `2` | `MsgID` | 1 Byte | 由消息发起者维护的滚动计数器，消息接收者回复 ACK 时回填相同的 MsgID |
| `3` | `MsgType` | 1 Byte | 消息类型<br>`0x00`: CMD<br>`0x01`: EVENT<br>`0x02`: ACK<br>`0x03~0xFF`: Reserved |
| `4` | `MsgCode` | 1 Byte | 当`MsgType = CMD`时：MsgCode 表示命令码<br>当`MsgType = EVENT`时：MsgCode 表示事件码<br>当`MsgType = ACK`时：MsgCode 表示被确认的原始命令码/事件码 |
| `5` | `Flags` | 1 Byte | 标志位 |
| `6` | `DataLen` | 1 Byte | 业务数据长度，单位：byte |
| `7` | `Data` | N Byte | 业务数据 |
| `7 + N` | `CRC16` | 2 Byte | 从 MsgID 到 Data 最后一个字节的 CRC16 校验值，小端序 |

### 2.2 字段定义
#### 2.2.1 SOF 定义
Start of Frame‌，帧头，固定为`0xB4`、`0x7B`，发送时先发`0xB4`，后发`0x7B`。<br>
用于在连续字节流中定位帧起点，消息接收者可将帧头用于处理拆包、粘包等情况以及用于发生通信异常后的重新同步。

#### 2.2.2 MsgID 定义
MsgID 是由消息发起者维护的滚动计数器，主机与从机各自维护一个独立的 MsgID，发送消息时分配新的 MsgID，ACK 不产生新的 MsgID。
```
取值范围：0x00 ~ 0xFF
初始值：0x00
变化规则：MsgID++
回绕规则：0xFF 的下一值为 0x00
```
可分以下两种情况：

(1) 主机发送给从机的消息（ACK 除外）
- 主机单独维护一个 MsgID
- 主机每发送一条新的消息（ACK 除外）后，执行`MsgID++`
- 从机收到主机消息若需回复 ACK 时，填写所回复主机消息的原始 MsgID
- 主机重传同一条消息时，应保持 MsgID 不变

(2) 从机发送给主机的消息（ACK 除外）
- 从机单独维护一个 MsgID
- 从机每发送一条新的消息（ACK 除外）后，执行`MsgID++`
- 主机收到从机消息若需回复 ACK 时，填写所回复从机消息的原始 MsgID
- 从机重传同一条消息时，应保持 MsgID 不变

#### 2.2.3 MsgType 定义
| MsgType | Value | Direction | Description |
|---|:---:|:---:|---|
| `CMD` | `0x00` | Host -> Device | 下发命令 |
| `EVENT` | `0x01` | Device -> Host | 异步执行结果或者异步事件上报 |
| `ACK` | `0x02` | 双向 | 命令/事件接收、校验及受理结果 |
| `Reserved` | `0x03…0xFF` | NA | Reserved - 不得使用 |

基本交互关系如下: 
```
主机下发命令
Host                                    Device
 │                                         │
 │──────── CMD ───────────────────────────>│
 │<─────────────────────────── ACK ────────│
 │                                         │异步执行（如有）
 │<────────────────────────── EVENT ───────│
 │──────── ACK ───────────────────────────>│
```
```
从机异步事件上报
Host                                    Device
 │                                         │
 │<────────────────────────── EVENT ───────│
 │──────── ACK ───────────────────────────>│
```

#### 2.2.4 MsgCode 定义
MsgCode 的含义由消息的 MsgID 决定。
- 当`MsgType = CMD`时：MsgCode 表示命令码，命令码的具体定义见后续章节
- 当`MsgType = EVENT`时：MsgCode 表示事件码，事件码的具体定义见后续章节
- 当`MsgType = ACK`时：MsgCode 表示被确认的原始命令码/事件码

#### 2.2.5 Flags 定义
| Bit(s) | Field | Description |
|:---:|---|---|
| `B0` | `ACK_Required` | 当前消息是否需要对方回复ACK<br>0 = 不需要<br>1 = 需要 |
| `B1` | `Retransmit` | 当前消息是否为重传消息<br>0 = 否<br>1 = 是 |
| `B2…7` | `Reserved` | Reserved - 应设为0 |

#### 2.2.6 DataLen 定义

#### 2.2.7 Data 定义

#### 2.2.8 CRC16 定义
