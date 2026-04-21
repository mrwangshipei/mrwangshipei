## SerialPortHelper 串口通信工具类



`SerialPortHelper` 是一个专为自定义串口协议设计的辅助类，适用于需要识别完整帧（如帧头/帧尾/校验）的通信场景，典型用于工业设备、嵌入式系统、仪器通讯等。



---



### ✅ 核心功能



* **完整帧判断机制**：通过用户传入的判断逻辑识别是否已收到完整数据帧。

* **自适应延迟触发机制**：避免数据包截断或早读。

* **线程安全的读写控制**：多线程环境下可靠通信。

* **支持缓存查看（Peek）与清除（ReadAndClear）**：灵活管理串口缓冲区。

* **支持发送后等待响应（SendAndWaitResult）**：封装典型收发流程。

---

# Upper.SerialPortHelper

本项目采用 [Mozilla Public License 2.0 (MPL-2.0)](https://www.mozilla.org/en-US/MPL/2.0/) 开源。

## 主要授权条款说明

- 允许商业用途，包括闭源项目。
- 只要你修改了本项目的源码文件（如`.cs`文件），必须将修改后的这些文件开源。
- 你可以将本项目与闭源代码一同分发，未修改本项目文件的代码不受MPL约束。

如需了解详细条款，请参阅 [LICENSE](./LICENSE) 文件或官方协议说明。

---



### 🧱 典型用法



```csharp

using System.IO.Ports;

using Upper.SerialPortHelper;



SerialPort sp = new SerialPort("COM3", 9600);

var helper = new SerialPortHelper(sp);



helper.IsFullFrame += (data) =>

{

   // 示例：完整帧以 0xAA 开头，0xFF 结尾，且长度大于等于5

   return data.Length >= 5 && data[0] == 0xAA && data[^1] == 0xFF;

};



if (helper.Open())

{

   byte[] cmd = new byte[] { 0xAA, 0x01, 0x02, 0x03, 0xFF };

   byte[] response = helper.SendAndWaitResult(cmd, 100, onlyread: false);

   Console.WriteLine(BitConverter.ToString(response));

}

```



---



### 📌 方法说明



| 方法                                                                   | 描述               |
| -------------------------------------------------------------------- | ---------------- |
| `SerialPortHelper(SerialPort serial, int MsgsWaitTime = 35)`         | 初始化帮助类，绑定外部串口实例  |
| `bool Open()` / `bool Close()`                                       | 打开 / 关闭串口连接      |
| `void Send(byte[] msg)`                                              | 发送数据，不等待响应       |
| `byte[] SendAndWaitResult(byte[] data, int waittime, bool onlyread)` | 发送数据并等待接收完整帧     |
| `byte[] ReadAndClear(int waitms)`                                    | 读取数据并清空缓存区       |
| `byte[] Peek(int waitms)`                                            | 只查看缓存区数据，不清除     |
| `byte[] Receive(int waitms, bool onlyread = false)`                  | 通用读取函数，支持只读或清除缓存 |



---



### 🧠 注意事项



* 本类 **仅支持单个串口实例管理**，如需多串口通信，可自行封装字典管理：`Dictionary<string, SerialPortHelper>`。

* `IsFullFrame` 是关键逻辑，必须由用户提供完整帧的识别方式，否则延迟机制会退化为定时释放。

* `SendAndWaitResult` 推荐用于典型“请求-响应”结构的协议封装。

* 所有手动 `Receive` 的操作，应在确认线程安全的前提下调用。



---



### ♻️ 资源释放



使用结束后请及时释放资源避免线程挂起或句柄泄漏：



```csharp

helper.Dispose();

```



---



如需英文文档、单元测试示例、或多串口封装样例，可联系补充。

---

## BERT 串口误码测试（外层协议）

解决方案已新增控制台项目 `Upper.SerialPortHelper.Bert`，用于通过 BERT 协议（长度+协议标识+PRBS负载+CRC16）对串口链路进行误码率测试（BER）。

### 协议帧格式
- `[len16][0xB1][payload...][crc16]`
  - `len16`：后续字节总数（含协议标识、负载、CRC16），低字节在前
  - `payload`：PRBS 伪随机序列（支持 7/9/15/23/31 阶）
  - `crc16`：CCITT(0x1021, init=0xFFFF) 对 `[len16|0xB1|payload]` 的校验

期望被测端回环原样返回，或使用相同协议返回数据，工具会按照 PRBS 期望序列逐位对比统计误码。

### 运行示例
```bash
dotnet run --project ./Upper.SerialPortHelper.Bert \
  --port COM3 --baud 115200 \
  --duration 30 --rate 100 --payload 128 --prbs 15 \
  --frame-wait 35 --timeout-read 1000 --timeout-write 1000 --verbose
```

### 参数说明
- `--port`：串口名（如 `COM3`）
- `--baud`：波特率（默认 115200）
- `--duration`：测试时长秒（默认 30）
- `--rate`：发送帧率（帧/秒，默认 100）
- `--payload`：每帧负载字节数（默认 128）
- `--prbs`：PRBS 阶数（7/9/15/23/31，默认 15）
- `--frame-wait`：帧聚合等待毫秒（默认 35）
- `--timeout-read/--timeout-write`：读/写超时（默认 1000ms）
- `--verbose`：打印串口收发日志

### 兼容性
- 库 `Upper.SerialPortHelper` 目标 `netstandard2.0`
- 控制台 `Upper.SerialPortHelper.Bert` 目标 `net8.0`
- 库工程已排除编译 BERT 子目录，互不影响



