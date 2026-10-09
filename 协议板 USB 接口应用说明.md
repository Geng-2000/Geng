# 协议板 USB 接口应用说明
## 1 通信接口
USB 2.0 Full-Speed Device，CDC Class
>协议板为从机

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|:---:|---|---|---|
| 0 | `SOF` | 2 Byte | 帧头，固定为`0xB4`、`0x7B`，先发`0xB4`，后发`0x7B` |
| 2 | `MsgID` | 1 Byte | 由消息发起者维护的滚动计数器，消息接收者回复 ACK 时与其跟随 |
| 3 | `MsgType` | 1 Byte | 消息类型<br>`0x00`: CMD<br>`0x01`: EVENT<br>`0x02`: ACK<br>`0x03~0xFF`: Reserved |
| 4 | `MsgCode` | 1 Byte | `MsgType = CMD`: MsgCode 表示命令码<br>`MsgType = EVENT`: MsgCode 表示事件码<br>`MsgType = ACK`: MsgCode 表示被确认的原始命令码/事件码 |
| 5 | `Flags` | 1 Byte | 标志位 |
| 6 | `DataLen` | 2 Byte | 业务数据长度，单位：byte，小端序 |
| 8 | `Data` | N Byte | 业务数据 |
| 8 + N | `CRC16` | 2 Byte | 从 MsgID 到 Data 最后一个字节的 CRC16 校验值，小端序 |

### 2.2 字段定义
#### 2.2.1 SOF 定义
Start of Frame‌，帧头，固定为 0xB4、0x7B，发送时先发 0xB4，后发 0x7B。<br>
用于定位帧起点，可帮助消息接收者在遇到拆包、粘包或者通信异常等问题后重新同步。

#### 2.2.2 MsgID 定义
MsgID 是消息发起者维护的一个滚动计数器
```
取值范围：0x00 ~ 0xFF
初始值：0x00
回绕规则：0xFF 的下一值为 0x00
```
(1) 主机发送给从机的消息（ACK 除外）
- 主机单独维护一个 MsgID
- 主机每发送一条新的消息（ACK 除外）后，执行`MsgID++`
- 从机收到主机消息若需回复 ACK 时，填写所回复的主机消息的原始 MsgID
- 主机重传同一条消息时，应保持 MsgID 不变

(2) 从机发送给主机的消息（ACK 除外）
- 从机单独维护一个 MsgID
- 从机每发送一条新的消息（ACK 除外）后，执行`MsgID++`
- 主机收到从机消息若需回复 ACK 时，填写所回复的从机消息的原始 MsgID
- 从机重传同一条消息时，应保持 MsgID 不变

#### 2.2.3 MsgType 定义
| MsgType | Value | Direction | Description |
|---|:---:|:---:|---|
| `CMD` | `0x00` | Host -> Device | 下发命令 |
| `EVENT` | `0x01` | Device -> Host | 异步执行结果或者异步事件上报 |
| `ACK` | `0x02` | 双向 | 命令/事件接收、校验及受理结果 |
| Reserved | `0x03~0xFF` | NA | Reserved |

#### 2.2.4 MsgCode 定义

#### 2.2.5 Flags 定义

#### 2.2.6 DataLen 定义

#### 2.2.7 Data 定义

#### 2.2.8 CRC16 定义
