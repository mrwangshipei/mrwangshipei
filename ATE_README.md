\# AXATE (Android eXtended Automatic Test Equipment)



基于 WinForms 开发的自动化测试解决方案，用于 Android 芯片/设备的功能与性能测试。



\[!\[Windows](https://img.shields.io/badge/OS-Windows-blue)](https://www.microsoft.com/windows)

\[!\[.NET Framework](https://img.shields.io/badge/.NET-Framework-5C2D91?logo=.net)](https://dotnet.microsoft.com/framework)

\[!\[Visual Studio](https://img.shields.io/badge/IDE-Visual%20Studio-5C2D91?logo=visual-studio)](https://visualstudio.microsoft.com)



\---



\## 目录



\- \[技术栈](#技术栈)

\- \[使用的开源项目](#使用的开源项目)

\- \[源代码行数](#源代码行数)

\- \[项目结构](#项目结构)

\- \[入口点](#入口点)

\- \[技术实现概览](#技术实现概览)

\- \[构建要求](#构建要求)



\---



\## 技术栈



| 类别 | 技术 |

|------|------|

| \*\*核心框架\*\* | .NET Framework 4.8 |

| \*\*UI 框架\*\* | Windows Forms, Microsoft.Ink (触控笔支持) |

| \*\*语言\*\* | C# 10.0 (preview) |

| \*\*数据库\*\* | SQLite 3 (通过 System.Data.SQLite) |

| \*\*硬件通信\*\* | USB HID, Serial Port, I²C, SPI |

| \*\*远程服务\*\* | WCF (Windows Communication Foundation) |

| \*\*消息队列\*\* | MQTT (第三方库集成) |

| \*\*数据处理\*\* | ADO.NET, PetaPoco ORM |



\### 项目架构



解决方案采用多项目结构，包含以下核心模块：



| 项目 | 说明 |

|------|------|

| `TKATS` | 主程序入口 (AX-ATS.exe) |

| `Common` | 共享类库 (工具类、模型、状态机) |

| `Protocol` | 通讯协议实现 (AC/DC/ 示波器/ 高压等) |

| `Dao` | 数据访问层 (PetaPoco 映射) |

| `DataUpload` | 数据上传模块 (HTTP/WCF) |

| `ATEService` | WCF 服务接口定义 |

| `SqliteHelper` | SQLite 数据库辅助类 |

| `SerailDriver` | 串口驱动封装 |

| `ModelOrShiJian` | 模型与事件定义 |

| `UPPERIOC.MutlSourceManager` | IOC 容器与多源工厂管理 |



\---



\## 使用的开源项目



| 名称 | 版本 | 用途 |

|------|------|------|

| \*\*PetaPoco.Compiled\*\* | 6.0.684-beta | 轻量级 ORM，用于 SQLite 数据操作 |

| \*\*Serilog\*\* | 4.3.0 | 结构化日志记录 (文件/Seq) |

| \*\*Serilog.Sinks.Seq\*\* | 9.0.0 | 日志聚合与可视化 |

| \*\*Serilog.Sinks.File\*\* | 7.0.0 | 日志文件持久化 |

| \*\*Newtonsoft.Json\*\* | 13.0.1 | JSON 序列化/反序列化 |

| \*\*UPPERIOC\*\* | 2.0.4.103 | 自研 IOC 容器框架 |

| \*\*FrmControl\*\* | 1.0.2.49 | UI 控件库 (第三方) |

| \*\*System.Data.SQLite\*\* | 1.0.103.0 | SQLite ADO.NET 提供程序 |

| \*\*Microsoft.Web.WebView2\*\* | 1.0.3719.77 | WebView2 容器集成 |



\### 外部依赖 DLL (WaiBuDLL)



\- `GongJuJiHe.dll` - 工具集合

\- `Microsoft.Ink.dll` - 触控笔输入

\- `System.Data.SQLite.dll` - SQLite 驱动

\- `UIChuanTi.dll` - UI 传输库

\- `ZiDingYiKongJian.dll` - 自定义控件



\---



\## 源代码行数



```bash

Total: \~71,000 lines of C# source code

```



\### 主要模块代码量 (估算)



| 模块 | 行数 | 说明 |

|------|------|------|

| `TKATS` (主程序) | \~4,000 | UI 界面与主流程 |

| `Common` | \~2,500 | 核心类库与工具 |

| `Protocol` | \~1,500 | 通讯协议实现 |

| `Dao` | \~500 | 数据访问层 |

| `DataUpload` | \~800 | 数据上传逻辑 |

| `ATEService` | \~1,000 | WCF 服务实现 |

| 其他模块 | \~60,000 | 各功能模块 (测试项、设备驱动等) |



> 注：包含设计器生成代码 (.Designer.cs) 和资源文件代码。



\---



\## 项目结构



```

AXATE/

├── TKATS/                    # 主程序

│   ├── Program.cs           # 入口点 (STAThread Main)

│   ├── TKATS.cs             # 主窗体类 TKATSClass

│   ├── BLL/                 # 业务逻辑层

│   ├── Pages/               # 页面窗体

│   ├── KongJian/            # 自定义控件

│   ├── Util/                # 工具类

│   └── View/                # 视图层

├── Common/                   # 共享类库

│   ├── ATS/                 # ATS 核心逻辑

│   ├── Factory/             # 工厂模式实现

│   ├── TestLifecycle/       # 测试生命周期

│   ├── protocol/            # 协议模型

│   ├── Language/            # 多语言支持

│   └── utils/               # 工具类

├── Protocol/                 # 通讯协议

│   ├── ACSource\_\*.cs       # AC 电源协议

│   ├── HighVoltage\_\*.cs    # 高压测试协议

│   ├── Oscilloscope\_\*.cs   # 示波器协议

│   └── TestBox\_TK.cs       # 测试盒协议

├── Dao/                      # 数据访问层

├── DataUpload/               # 数据上传

├── ATEService/               # WCF 服务

├── SQLiteHelper/             # SQLite 辅助

├── SerailDriver/             # 串口驱动

├── ModelOrShiJian/           # 模型与事件

├── UPPERIOC.MutlSourceManager/ # IOC 容器

└── WaiBuDLL/                 # 外部依赖 DLL

```



\---



\## 入口点



\### 主程序入口



\*\*文件\*\*: `TKATS/Program.cs`



```csharp

\[STAThread]

static void Main()

{

&#x20;   // 全局异常处理注册

&#x20;   AppDomain.CurrentDomain.UnhandledException += GlobalExceptionHandler;

&#x20;   TaskScheduler.UnobservedTaskException += GlobalTaskExceptionHandler;



&#x20;   // 单实例检查

&#x20;   Mutex mutex = new Mutex(false, string.Format("{0} - By UZ", Application.ProductName));

&#x20;   Variables.IsAllReadyRun = !mutex.WaitOne(100, false);



&#x20;   // 管理员权限检查

&#x20;   WindowsPrincipal principal = new WindowsPrincipal(WindowsIdentity.GetCurrent());

&#x20;   if (principal.IsInRole(WindowsBuiltInRole.Administrator))

&#x20;   {

&#x20;       // 初始化数据库

&#x20;       Center.InitDb();



&#x20;       // IOC 容器配置

&#x20;       UPPERIOCApplication.RunInstance(modcfg => {

&#x20;           modcfg.UPPERFileModelMoudle(new UFileMConfig());

&#x20;           modcfg.AddModule<UPPERMutiFacotyMoudle<ATEFactory>>();

&#x20;           // 注册状态机与数据库

&#x20;           modcfg.Provider.Rigister(new TestStateMachine());

&#x20;           modcfg.Provider.Rigister(IDatabase db);

&#x20;       });



&#x20;       // 启动主窗口

&#x20;       Application.Run(new TKATSClass(channelType, ini));

&#x20;   }

}

```



\### 主窗口类



\*\*文件\*\*: `TKATS/TKATS.cs`



```csharp

public partial class TKATSClass : BaseForm, IATEForm

{

&#x20;   // 实现 IATEForm 接口

&#x20;   // 提供测试流程控制、数据采集、结果处理等核心功能

}

```



\---



\## 技术实现概览



\### 1. 状态机驱动的测试流程



采用\*\*状态机模式\*\*管理测试流程，确保线程安全：



```

None → PrepareTest → Testing → \[ Pause ] → (Completed | Failed | Aborted) → None

```



\*\*实现\*\*: `Common/status/TestStateMachine.cs`



\- 支持状态转换回调 (OnEnter/OnExit)

\- 线程安全的状态切换

\- 事件通知机制 (StatusChanged)



\### 2. 多工厂模式



支持不同测试设备的插件化集成：



```csharp

public enum ATEFactory

{

&#x20;   ZhuoNeng,     // 卓能测试设备

}



public enum ATEFunc

{

&#x20;   BeforeChangePN,

&#x20;   FirstLoadPN,

&#x20;   FirstShown

}

```



\*\*特点\*\*:

\- 通过 IOC 容器注册不同的工厂实现

\- 支持运行时切换测试设备类型



\### 3. 测试生命周期管理



\*\*实现\*\*: `Common/TestLifecycle/`



```

Cycle Model:

&#x20; Lifecycle → Event → Message → COM Port → Protocol

```



支持的生命周期类型:

\- 基础测试循环

\- 老化测试 (Aging)

\- 多通道并行测试



\### 4. 多语言支持



通过 ResX 资源文件实现多语言切换：



```

Language/Resouces/

&#x20; ├── Resources\_ch.resx   # 中文

&#x20; ├── Resources\_en.resx   # 英文

&#x20; └── FanYich\_en.resx     # 翻译资源

```



程序启动时根据配置切换 UI 语言：



```csharp

Thread.CurrentThread.CurrentUICulture = new CultureInfo(language);

```



\### 5. 数据持久化



使用 \*\*PetaPoco ORM\*\* 操作 SQLite 数据库：



```csharp

// 结果数据库

var db = DatabaseConfiguration.Build()

&#x20;   .UsingConnectionString("Data Source=Result.db;Password=12345;")

&#x20;   .UsingProvider<SQLiteDatabaseProvider>()

&#x20;   .Create();



// 配置数据库

var dbData = DatabaseConfiguration.Build()

&#x20;   .UsingConnectionString("Data Source=db/Data.db;Password=12345;")

&#x20;   .Create();

```



\*\*表结构\*\*:

\- `test\_results` - 测试结果

\- `material\_version\_history` - 物料版本历史



\### 6. 通讯协议



支持多种测试设备协议:



| 设备类型 | 协议实现 |

|----------|----------|

| AC 电源 | `ACSource\_APS5000A`, `ACSource\_PAG1010` |

| 高压测试仪 | `HighVoltage\_JK7122`, `HighVoltage\_U9053` |

| 示波器 | `Oscilloscope\_DS1102E` |

| 测试盒 | `TestBox\_TK` |



\### 7. 数据上传



通过 HTTP/WCF 将测试数据上传至服务器：



```

DataUploadCenter

&#x20; ├── HttpCommunication (HTTP REST API)

&#x20; └── ATEService (WCF)

```



支持的数据上报:

\- 卓能系统对接

\- 物料版本同步

\- 测试结果上传



\### 8. 安全机制



\- \*\*单实例运行\*\*: 通过 Mutex 实现

\- \*\*管理员权限\*\*: 自动提权运行

\- \*\*互斥体保护\*\*: 防止多实例冲突



\---



\## 构建要求



\### 开发环境



| 软件 | 版本 |

|------|------|

| \*\*Visual Studio\*\* | 2022 |

| \*\*.NET Framework\*\* | 4.8 |

| \*\*Windows SDK\*\* | 最新 |



\### 构建配置



解决方案支持以下构建配置：



| 配置 | 目标平台 | 说明 |

|------|----------|------|

| Debug | AnyCPU/x86/x64 | 调试版本 |

| Release | x86/x64 | 发布版本 (x86 默认) |



\### 输出目录



```

Release/

&#x20; ├── AX-ATS.exe           # 主程序

&#x20; ├── Common.dll           # 共享类库

&#x20; ├── Dao.dll              # 数据访问层

&#x20; ├── Protocol.dll         # 协议库

&#x20; └── buffer-logs/         # 缓冲日志

```



\---



\## 许可证



&#x20;proprietary - 本项目为商业项目，仅供内部使用。



\---



\## 作者



AXATE Development Team



\---



\## 版本历史



| 版本 | 日期 | 说明 |

|------|------|------|

| 1.0.x | - | 初始版本 |

| 1.0.0.\* | - | 持续迭代中 |



\---



\*本文档根据项目源代码 自动生成\*



