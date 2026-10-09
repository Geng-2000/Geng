# 协议板 USB 接口应用说明
## 1 通信接口
USB 2.0 Full-Speed Device，CDC Class

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|:---:|---|---|---|
| 0 | `SOF` | 2 Byte | 帧头，固定为`0xB4`、`0x7B`，先发`0xB4`，后发`0x7B` |
| 2 | `MsgID` | 1 Byte | 由消息发起者维护的滚动计数器，消息接受者回复ACK时与其跟随 |
| 3 | `MsgType` | 1 Byte | 消息类型<br>`0x00`: CMD<br>`0x01`: EVENT<br>`0x02`: ACK<br>`0x03~0xFF`: Reserved |
| 4 | `MsgCode` | 1 Byte | MsgType = CMD: MsgCode表示命令码<br>MsgType = EVENT: MsgCode表示事件码<br>MsgType = ACK: MsgCode表示被确认的原始命令码/事件码 |
| 5 | `Flags` | 1 Byte | 标志位 |
| 6 | `DataLen` | 2 Byte | 业务数据长度，小端序 |
| 8 | `Data` | N Byte | 业务数据 |
| 8 + N | `CRC16` | 2 Byte | 从MsgID到Data最后一个字节的CRC16校验值，小端序 |

### 2.2 字段定义
#### 2.2.1 SOF定义
Start of Frame‌，帧头，固定为0xB4、0x7B，发送时先发0xB4，后发0x7B。
