<div align="center">

![logo](asset/UPPERIOC.png)

</div>

<h1 align="center">UPPERIOC</h1>

<p align="center">
  <a href="https://github.com/mrwangshipei/UPPERIOC">
    <img src="https://badgen.net/badge/Github/mrwangshipei/21D789?icon=github">
  </a>
  <img src="https://img.shields.io/badge/NetStandard-2.0-blue">
  <a href="https://github.com/mrwangshipei/UPPERIOC/blob/master/LICENSE">
    <img alt="GitHub" src="https://img.shields.io/github/license/mrwangshipei/UPPERIOC?style=flat-square">
  </a>
  <img alt="GitHub last commit" src="https://img.shields.io/github/last-commit/mrwangshipei/UPPERIOC?style=flat-square">
  <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/mrwangshipei/UPPERIOC?style=social">
</p>

<p align="center"><strong>一个为 WinForm 应用设计的轻量级 IOC 容器和模块化框架</strong></p>

---

## 📑 目录

- [简介](#-简介)
- [安装](#-安装)
- [快速入门](#-快速入门)
- [核心功能](#-核心功能)
  - [IOC 容器](#ioc-容器)
  - [日志系统](#日志系统)
  - [消息通信](#消息通信-sendor)
  - [文件配置模型](#文件配置模型-model)
  - [应用锁](#应用锁-mlock)
  - [权限系统](#权限系统)
  - [翻译模块](#翻译模块)
- [模块开发指南](#-模块开发指南)
- [最佳实践](#-最佳实践)
- [常见问题](#-常见问题)
- [贡献](#-贡献)

---

## 📖 简介

UPPERIOC 是一个为 WinForm 应用设计的 IOC 容器和插件集合框架，旨在帮助开发者快速搭建标准化、模块化的单体应用程序。

### 核心特性

- ✅ **轻量级 IOC 容器** - 支持构造函数注入、属性注入、命名绑定
- ✅ **模块化架构** - 可插拔的模块设计，灵活扩展
- ✅ **事件驱动** - 基于应用生命周期的事件系统
- ✅ **开箱即用** - 内置日志、配置、消息通信等常用功能
- ✅ **WinForm 友好** - 专门为 WinForm 应用优化设计

**适合场景：** 个人学习、小型项目、原型系统、快速开发

> ⚠️ **注意：** 建议在小型到中型项目中使用，使用前请确保理解项目架构和设计原理。

---

## 📦 安装

### NuGet 安装

```bash
# Package Manager Console
Install-Package UPPERIOC

# .NET CLI
dotnet add package UPPERIOC
```

### 预发布版本

```bash
# MyGet Pre-release feed
# 添加源：https://www.myget.org/F/upperioc/api/v3/index.json
```

### 系统要求

- .NET Framework 4.6.2+
- .NET Standard 2.0+
- WinForms 应用（推荐）

---

## 🚀 快速入门

### 最小化配置

```csharp
using UPPERIOC;
using UPPERIOC2.UPPER.UIOC.Center;

static void Main()
{
    // 初始化框架
    UPPERIOCApplication.RunInstance(md =>
    {
        // 可选：配置模块
    });

    Application.Run(new MainForm());
}
```

### 完整配置示例

```csharp
using UPPERIOC;
using UPPERIOC.UPPER.IOC.Center.Configuation;
using UPPERIOC2.UPPER.UIOC.Center;

static void Main()
{
    UPPERIOCApplication.RunInstance(md =>
    {
        // 日志模块配置
        md.UPPERLogFileMoudle(new FileLogConfig
        {
            DirectoryName = "logs",
            DefaultExt = ".log",
            WhichTypePrint = new List<LogType>
            {
                LogType.Debug,
                LogType.Info,
                LogType.Warn,
                LogType.Error
            },
            FileNameTimeFormat = "yyyy-MM-dd",
            HowManyHourSave = 48,  // 保留 48 小时日志
            PrintMs = true
        });

        // 文件模型配置
        md.UPPERFileModelMoudle(new FileModelConfig
        {
            SaveModelPath = "config"
        });

        // 应用锁配置
        md.UPPERMLockMoudle(new MLockConfiguation
        {
            Solt = "MyApp_Slot",
            Listenaddr = "127.0.0.1:8080",
            LockName = "MyAppInstance"
        });
    });

    Application.Run(new MainForm());
}
```

---

## 🧩 核心功能

### IOC 容器

UPPERIOC 提供了轻量级的依赖注入容器，支持多种注入方式。

#### 1. 特性标记

```csharp
using UPPERIOC.UPPER.IOC.Annaiation;

// 标记为可注入的对象
[IOCObject]
public class UserService
{
    public string GetUserName() => "Admin";
}

// 构造函数注入
[IOCObject]
public class UserController
{
    private readonly UserService _userService;

    [IOCConstructor]
    public UserController(UserService userService)
    {
        _userService = userService;
    }
}

// 属性注入
[IOCObject]
public class OrderController
{
    [IOCPorpeties]
    public UserService UserService { get; set; }

    [IOCConstructor]
    public OrderController() { }
}
```

#### 2. 注册与解析

```csharp
using UPPERIOC2.UPPER.UIOC.Center;

// 自动注册（扫描 [IOCObject] 标记的类）
var service = U.C.GetInstance<UserService>();

// 手动注册
U.C.Rigister<ILogger>(new FileLogger());

// 命名注册
U.C.Rigister<ILogger>("FileLogger", new FileLogger());
U.C.Rigister<ILogger>("DatabaseLogger", new DatabaseLogger());

// 按名称解析
var fileLogger = U.C.GetInstance<ILogger>("FileLogger");

// 获取所有实现
var allLoggers = U.C.GetAllInstance<ILogger>();

// 获取接口实现（自动查找子类）
var logger = U.C.GetInstanceAndSub<ILogger>();
```

#### 3. 生命周期管理

```csharp
// 单例模式（默认）
U.C.Rigister<CacheService>(SingleBean: true);

// 瞬态模式（每次获取新实例）
U.C.Rigister<ConnectionService>(SingleBean: false);
```

---

### 日志系统

内置文件日志实现，支持日志类型过滤、自动清理等功能。

#### 配置示例

```csharp
using UPPERIOC.UPPER.UFileLog.IConfiguation;

internal class AppLogConfig : IFileLogConfiguation
{
    public string DirectoryName => "ApplicationLogs";
    public string DefaultExt => ".log";
    public List<LogType> WhichTypePrint => new()
    {
        LogType.Debug,
        LogType.Info,
        LogType.Warn,
        LogType.Error
    };
    public string FileNameTimeFormat => "yyyyMMdd_HHmmss";
    public int HowManyHourSave => 72;  // 保留 72 小时
    public bool PrintMs => true;        // 打印毫秒
}

// 注册日志模块
UPPERIOCApplication.RunInstance(md =>
{
    md.UPPERLogFileMoudle(new AppLogConfig());
});
```

#### 使用方式

```csharp
using UPPERIOC.UPPER;

// 记录日志
LogCenter.Log("应用程序启动");
LogCenter.Debug("调试信息");
LogCenter.Info("用户登录成功");
LogCenter.Warn("警告：磁盘空间不足");
LogCenter.Error("错误：数据库连接失败");

// 带异常信息的日志
try
{
    // 可能抛出异常的代码
}
catch (Exception ex)
{
    LogCenter.Error("操作失败", ex);
}
```

---

### 消息通信 (Sendor)

基于发布/订阅模式的消息总线，用于模块间解耦通信。

#### 基本用法

```csharp
using UPPERIOC.UPPER.USendor.Center;

// 注册订阅者
SendorCenter.Register<string>(msg =>
{
    LogCenter.Info($"收到消息：{msg}");
});

// 带条件的订阅
SendorCenter.Register<UserEvent>(e =>
{
    if (e.EventType == "Login")
    {
        LogCenter.Info($"用户 {e.UserName} 登录");
    }
});

// 发布消息
SendorCenter.Publish<string>("Hello World");
SendorCenter.Publish(new UserEvent
{
    EventType = "Login",
    UserName = "Admin"
});
```

#### 实践场景

```csharp
// 定义事件类型
public class DataChangedEvent
{
    public string TableName { get; set; }
    public string Operation { get; set; }
    public object Data { get; set; }
}

// 在数据仓库中发布事件
public class Repository
{
    public void Save<T>(T entity)
    {
        // 保存逻辑
        SendorCenter.Publish(new DataChangedEvent
        {
            TableName = typeof(T).Name,
            Operation = "Save",
            Data = entity
        });
    }
}

// 在 UI 中订阅并刷新界面
SendorCenter.Register<DataChangedEvent>(e =>
{
    if (e.TableName == "User")
    {
        RefreshUserGrid();
    }
});
```

---

### 文件配置模型 (Model)

持久化配置和数据模型的存储系统。

#### 配置示例

```csharp
using UPPERIOC2.UPPER.UFileModel.IConfiguaion;

internal class AppConfig : IUFileModelConfiguation
{
    public string SaveModelPath => "AppConfig";
}

// 注册模块
UPPERIOCApplication.RunInstance(md =>
{
    md.UPPERFileModelMoudle(new AppConfig());
});
```

#### 使用方式

```csharp
using UPPERIOC2.UPPER.UFileModel.Center;
using UPPERIOC.UPPER.UFileModel.Model;

// 定义模型
[Serializable]
public class AppSettings
{
    public string Theme { get; set; } = "Light";
    public int MaxConnections { get; set; } = 100;
    public bool AutoSave { get; set; } = true;
}

// 保存配置
var settings = new AppSettings
{
    Theme = "Dark",
    MaxConnections = 200
};
UFileModelCenter.SaveModel(settings);

// 读取配置
var loadedSettings = UFileModelCenter.GetModel<AppSettings>();

// 获取或创建（如果不存在则创建默认值）
var config = UFileModelCenter.GetOrCreate<AppSettings>(() =>
    new AppSettings { Theme = "Default" });
```

---

### 应用锁 (MLock)

确保应用程序单实例运行。

#### 配置示例

```csharp
using UPPERIOC2.UPPER.MLOCK.IConfiguation;

public class MyAppLockConfig : MLockConfiguation
{
    public override string Solt => "MyApp_SingleInstance";
    public override string Listenaddr => "127.0.0.1:9090";
    public override string LockName => "MyApplication";

    public override void Noregister()
    {
        MessageBox.Show("程序已运行，请勿重复启动！");
        Environment.Exit(0);
    }
}

// 注册模块
UPPERIOCApplication.RunInstance(md =>
{
    md.UPPERMLockMoudle(new MyAppLockConfig());
});
```

---

### 权限系统

基于角色的权限控制模块。

#### 配置与使用

```csharp
using UPPERIOC.UPPER._Premission.UAttribute;
using UPPERIOC.UPPER._Premission.Center;

// 定义需要权限的方法
[PremissionRequired("User.Manage")]
public void DeleteUser(string userId)
{
    // 删除用户逻辑
}

// 检查权限
if (PremissionCenter.HasPermission("Report.View"))
{
    ShowReport();
}

// 获取当前用户权限
var permissions = PremissionCenter.GetCurrentUserPermissions();
```

---

### 翻译模块

多语言支持模块。

```csharp
using UPPERIOC.UPPER.Translate.IConfigration;
using UPPERIOC.UPPER.Translate.Center;

public class AppTranslateConfig : ITranslateConfig
{
    public string APPKey => "your_api_key";
    public string APPSeret => "your_secret";
    public string FromLanguage => "zh";
    public string ToLanguage => "en";
}

// 注册并设置
TranslateCenter.Instance.SetRootWindow(mainForm);
```

---

## 🛠️ 模块开发指南

### 创建自定义模块

```csharp
using UPPERIOC.UPPER.Moudle_;
using UPPERIOC.UPPER.IOC.Center.IProvider;

// 实现模块接口
public class CustomModule : IUPPERModule
{
    // 模块优先级（数字越小优先级越高）
    public int Priority => 100;

    // 模块名称
    public string ModuleName => "CustomModule";

    // 模块版本
    public VersionModel Version => new VersionModel(1, 0, 0);

    // 初始化
    public void Initialize(IContainerProvider container)
    {
        // 注册服务
        container.Rigister<ICustomService, CustomService>();
    }

    // 后初始化（所有模块初始化完成后调用）
    public void PostInitialize(IContainerProvider container)
    {
        // 执行依赖于其他模块的初始化逻辑
    }

    // 模块卸载前
    public void OnPreDestroy(IContainerProvider container)
    {
        // 清理资源
    }
}

// 注册模块
UPPERIOCApplication.RunInstance(md =>
{
    md.AddModule<CustomModule>();
});
```

### 应用生命周期事件

```csharp
using UPPERIOC.UPPER.Event.AppEvent;
using UPPERIOC.UPPER.Event.AppEvent.Impl;
using UPPERIOC.UPPER.Event.Attri;

// 订阅应用事件
public class ApplicationEventListener : IUPPERApplicationListener
{
    [Order(100)]
    public void OnApplicationPreInitialization(ApplicationPreInitializationEvent e)
    {
        // 应用预初始化
    }

    [Order(200)]
    public void OnApplicationStopping(ApplicationStoppingEvent e)
    {
        // 应用停止前
    }

    [Order(300)]
    public void OnApplicationStopped(ApplicationStoppedEvent e)
    {
        // 应用已停止
    }
}
```

---

## 💡 最佳实践

### 1. 模块化设计

将功能按业务领域拆分为独立模块，每个模块负责特定功能：

```csharp
// 用户模块
public class UserModule : IUPPERModule { }

// 订单模块
public class OrderModule : IUPPERModule { }

// 报表模块
public class ReportModule : IUPPERModule { }
```

### 2. 依赖注入优先

避免直接使用静态方法获取依赖，优先使用注入：

```csharp
// ❌ 不推荐
public class UserController
{
    public void GetUser()
    {
        var service = U.C.GetInstance<UserService>();
    }
}

// ✅ 推荐
[IOCObject]
public class UserController
{
    private readonly UserService _userService;

    [IOCConstructor]
    public UserController(UserService userService)
    {
        _userService = userService;
    }
}
```

### 3. 事件驱动架构

使用 SendOr 进行模块间通信，降低耦合：

```csharp
// 订单完成后发布事件
SendorCenter.Publish(new OrderCompletedEvent { OrderId = orderId });

// 库存模块订阅并处理
SendorCenter.Register<OrderCompletedEvent>(e =>
{
    UpdateStock(e.OrderId);
});
```

### 4. 日志分级使用

根据场景选择合适的日志级别：

```csharp
LogCenter.Debug("详细调试信息，生产环境可关闭");
LogCenter.Info("正常业务流程记录");
LogCenter.Warn("警告信息，不影响系统运行");
LogCenter.Error("错误信息，需要关注和处理");
```

### 5. 配置集中管理

将所有配置集中在启动时配置：

```csharp
UPPERIOCApplication.RunInstance(md =>
{
    md.UPPERLogFileMoudle(new LogConfig());
    md.UPPERFileModelMoudle(new ModelConfig());
    md.UPPERMLockMoudle(new LockConfig());
    md.AddModule<BusinessModule>();
});
```

---

## 🧯 常见问题

### IOCObject 失效

**问题：** CSC warning CS9057 编译器版本不兼容

**解决方法：** 更新 `Microsoft.CodeAnalysis.CSharp` 到 `4.10.0+`

```xml
<PackageReference Include="Microsoft.CodeAnalysis.CSharp" Version="4.10.0" />
```

### Configuration 找不到

**问题：** 配置接口无法自动注入

**解决方法 1：** 手动注入配置

```csharp
md.Provider.Rigister<MyConfig>(new MyConfig());
```

**解决方法 2：** 开启默认容器 Provider

```csharp
md.UPPERIOCMoudle();
```

### WinForm 设计器类无法显示

**问题：** 设计器无法加载窗体

**解决方法：**

1. 确保已生成所有项目
2. 重启 Visual Studio
3. 清理解决方案并重新生成

### 模块加载顺序问题

**问题：** 模块依赖导致初始化失败

**解决方法：** 使用 `Priority` 属性设置模块加载顺序

```csharp
public class DatabaseModule : IUPPERModule
{
    public int Priority => 10;  // 优先加载
}

public class UIModule : IUPPERModule
{
    public int Priority => 100;  // 后加载
}
```

---

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

### 开发环境搭建

```bash
# 克隆仓库
git clone https://github.com/mrwangshipei/UPPERIOC.git

# 还原依赖
dotnet restore

# 运行测试
dotnet test
```

### 联系

- 交流群：816781059
- QQ：364003656
- Email：contact@upperioc.dev

---

## 📄 许可证

MIT License

---

如果这个项目对你有帮助，欢迎 **Star** 🌟 支持一下！
