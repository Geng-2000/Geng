# 协议板 USB 接口应用说明
## 1 通信接口
USB 2.0 Full-Speed Device，CDC Class

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|:---:|---|---|---|
| 0 | `SOF` | 2 Byte | 帧头，固定为0xB4、0x7B，先发0xB4，后发0x7B |
| 2 | `MsgType` | 1 Byte | 消息类型<br>`0x00`: CMD<br>`0x01`: EVENT<br>`0x02`: ACK<br>`0x03~0xFF`: Reserved |
| 3 | `MsgCode` | 1 Byte | MsgType=CMD：MsgCode表示命令码<br>MsgType=EVENT：MsgCode表示事件码<br>MsgType=ACK：MsgCode表示被确认的原始命令码/事件码 |
