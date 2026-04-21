# FCT

FCT（Factory Test）是一款面向工厂的 Windows Forms 测试系统，支持多工厂切换、MES 系统对接、串口协议通信和软件授权管理。

## 项目结构

| 项目 | 说明 |
|---|---|
| **FCT** | 主程序 - Windows Forms 界面、串口通信、程序入口 |
| **HZH_Controls** | 自定义 WinForms 控件库（表格、图表、工厂可视化控件） |
| **UPPERIOC.MutiFactoryMoudle** | 自定义 IOC/DI 框架模块 |
| **WenJianFanWen** | 软件注册授权模块 |
| **HuaFengBLL** | 华丰工厂 MES 对接（HTTP API） |
| **TaoLongBLL** | 陶隆工厂 MES 对接（HTTP API） |
| **Model** | 共享模型、枚举、接口（FctFactory、FctMethod） |
| **Mig** | FluentMigrator 数据库迁移 |

## 编译运行

```bash
# 编译整个解决方案
dotnet build FCT.sln

# 编译主项目
dotnet build FCT.csproj

# 运行（从 bin/Debug 或 bin/Release 输出目录）
FCT.exe
```

## 架构概览

### UPPERIOC 依赖注入

UPPERIOC 是一套自定义 IOC 框架（以编译后的 DLL 形式存在）。核心用法：

- `UPPERIOCApplication.RunInstance(config => { ... })` - 引导容器
- `config.AddModule<T>()` - 注册模块（含 `OnPostInitialize()` 生命周期钩子）
- `config.Provider.Rigister<T>()` - 注册配置对象
- `UPPERIOCApplication.Container.GetInstance(typeof(T))` - 解析实例

主要单例访问入口：`M.I`（MutiFactoryCenter）和 `F.I`（UFileModelCenter）

### 多工厂模式

通过 `FctFactory` 枚举（`TaoLong`、`HuaFeng`）在运行时切换行为。每个工厂通过以下方式注册处理器：

```csharp
M.I.RigisterFunc(FctFactory.X, FctMethod.Y, handler);
M.I.InvokeFunc(FctMethod.Y, param);
```

`FctMethod` 取值：`InitFirstWindow`、`CanIStart`、`TestFinish`、`ChangeMes`、`ResultHandler`

### 启动流程（Program.cs）

1. 单实例检查（进程名 "FCT"）
2. `UPPERIOCApplication.RunInstance()` - 注册模块和配置
3. `Mig.Center.InitDb()` - 在 SQLite `FCT.db` 上运行 FluentMigrator 迁移
4. `Application.Run(new MainForm())`

### MainForm 主界面

- 20+ 个 PCB 输入面板（双击输入 SN）
- TabControl 中包含 `Test_list` DataGridView（Min/Max/Value/Rule 列）
- 状态栏显示授权状态（"正式机"、"试用机器"、"软件过期"）
- 通过 `M.I` 调用工厂特定操作（登录、SN 验证、结果上传）

### 数据库

- **SQLite** 数据库：`bin/Debug/FCT.db`
- 通过 `DbConnect.cs` 单例直接查询（`GetLocalDataTable()`、`RunSqlLocal()`）
- 两个连接：`SQLiteConn1`（本地）、`SQLiteConn2`（网络/共享）
- 通过 `Mig` 项目中的 **FluentMigrator** 管理 Schema 迁移

### 串口协议（CurrentSerialPort.cs）

- 封装 `System.IO.Ports.SerialPort`
- 35ms 帧结束定时器（以 35ms 间隔作为帧边界）
- 同步 `Write`/`Read` 带超时
- 通过 `LogCenter` 记录十六进制和 ASCII 日志

### MES 对接

各 BLL（`HuaFengMesCenter`、`TaoLongMesCenter`）均实现：

- `Login()` - 弹出登录窗口，获取认证 Token
- `CheckSN()` - 测试前验证条码是否在 MES 中登记
- `UploadResult()` - 上传测试结果（通过/失败）及参数

### 关键配置类

- `FCTUFileConfiguation` - 日志配置（`FCTlog/` 目录，48 小时保留）
- `UFileModelConfigration` - XML 模型持久化路径（`xml/` 目录）
- `ILockConfiguation` - 软件授权锁配置

### 日志

日志输出到 `FCTlog/` 目录，按日期分文件（Debug/、Error/、Info/）。调用方式：

```csharp
LogCenter.Log(LogType.X, message);
```
