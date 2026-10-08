# 协议板USB通信接口协议
## 1 通信接口与传输方式
USB 2.0 Full-Speed Device，CDC Class

## 2 协议说明
### 2.1 消息帧格式
| Offset | Field | Size | Description |
|:---:|---|---|---|
| 0 | `SOF` | 2 Byte | 帧头，固定为0xB4、0x7B，先发0xB4，后发0x7B |
| 2 | `MsgType` | 1 Byte | 消息类型：CMD、EVENT、ACK |
| 3 | `Msg`
