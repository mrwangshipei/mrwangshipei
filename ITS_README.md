# Integrated Test System (ITS) - 集成测试系统

Integrated Test System (ITS) 是一个面向电子产品制造行业的综合性测试解决方案，集成了功能测试、视觉检测、OCR识别、MES对接等多种能力。项目包含两个主要技术栈：**VTS (Vision Test System)** - 新一代视觉测试系统和 **OPPO Test System** - OPPO产品专用测试系统。

## 技术栈

### .NET Framework 4.8
- Windows Forms 应用程序
- C# 11.0 (preview)
- x86/x64 平台支持

### .NET 8.0 (VTS New Stack)
- .NET 8.0 Windows Forms
- 现代化跨平台支持

### 开发工具
- **Visual Studio 2022** (主要开发环境)
- MSBuild 构建系统

## 使用的开源项目

### 主要依赖

| 库名 | 版本 | 用途 |
|------|------|------|
| **UPPERIOC** | 2.0.4.103 | 自研IoC容器，基于依赖注入的模块化架构 |
| **Newtonsoft.Json** | 13.0.3 | JSON序列化/反序列化 |
| **Serilog** | 3.0.1 / 9.0.0 | 结构化日志，支持Seq输出 |
| **PetaPoco** | - | 轻量级ORM，SQLite数据库访问 |
| **OpenCvSharp4** | 4.x | OpenCV .NET封装，图像处理 |
| **SQLitePCLRaw** | 3.0.2 | SQLite原生绑定 |
| **M2Mqtt** | - | MQTT客户端，MES通信 |
| **FrmControl** | 1.0.2.49 | UI控件库 |

### 第三方SDK

| SDK | 用途 |
|-----|------|
| MvCameraControl.Net | 明远视觉相机驱动 (v3.4.0) |
| Zebra.Printer.SDK | Zebra打印机SDK (v3.0.3337) |
| HaiKangCamera SDK | 海康相机驱动 |
| PaddleOCR SDK | OCR文字识别 |

## 源代码统计

| 项目 | 统计数据 |
|------|----------|
| C# 源文件数 | 1,381 个 |
| 代码总行数 | 52,851 行 |
| 项目数量 | 60+ 个 |

> 统计范围：排除依赖库(yilai/)、备份文件(Backup*)、迁移备份(MigrationBackup/)、使用说明书/和驱动程序(DriverDll/)

## 入口点

### VTS (Vision Test System)
```
VTSUI/Program.cs
命名空间: VTSUI
主窗体: VTSMainForm
入口方法: static void Main()
```

**入口流程**:
1. 检查管理员权限（如非管理员则提权重启）
2. 依赖注入容器初始化 (UPPERIOC)
3. IoC模块注册 (数据库、文件模型、序列化等)
4. 启动主窗体 `VTSMainForm`

### OPPO Test System
```
OPPOYeWu/FangDanTongDao/Program.cs
命名空间: FangDanTongDao
主窗体: OppoForm / A181Form (根据配置)
入口方法: static void Main()
```

**入口流程**:
1. 检查 Administrator 权限
2. 包互斥体检查（防止多实例）
3. IoC容器初始化
4. 动态加载设备模型和MES实现
5. 启动主窗体和辅助窗体（ AssistantForm）

## 技术实现概览

### 架构设计

```
┌─────────────────────────────────────────────────────────────┐
│                    UI Layer (VTSUI / FangDanTongDao)        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ VTSMainForm  │  │  OppoForm    │  │  AssistantForm   │  │
│  └──────────────┘  └──────────────┘  └──────────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│               Business Logic Layer (VTSBll)                 │
│  图像处理 │ 数据校验 │ 测试逻辑 │ 报表生成                   │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                  Work Node Pipeline (VTSWorkNode)           │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────────┐  │
│  │ProcessNode│ │ OCRNode  │ │PrintNode │ │UpdateDbNode  │  │
│  └──────────┘ └──────────┘ └──────────┘ └──────────────┘  │
└─────────────────────────────────────────────────────────────┘
                            │
┌─────────────────────────────────────────────────────────────┐
│                   Data Access Layer (VTS Dao)               │
│  SQLite │ PetaPoco ORM │ Database Connection                │
└─────────────────────────────────────────────────────────────┘
```

### 核心模块

#### 1. VTS.Flow - 工作流引擎
- 基于JSON的流程定义文件
- `VTSFlowController` 管理流程生命周期
- 支持流程缓存优化

#### 2. VTSWorkNode - 处理节点
| 节点类型 | 功能描述 |
|----------|----------|
| `VTSProcessNode` | 主处理节点，支持并发执行 |
| `VTSOCRNode` | OCR文字识别 (PaddleOCR) |
| `VTSPrintNode` | 标签打印 (Zebra) |
| `VTSUpdateDbNode` | 数据库更新 |
| `VTSYingTongMesNode` | 鹰龙MES集成 |

#### 3. VTSOpenCvBll - 图像处理
- 图像预处理（去噪、增强）
- 边缘检测
- 模板匹配
- YOLO目标检测

#### 4. VTSPaddleOCR - OCR引擎
- 基于PaddleOCR的文本识别
- 支持多语言
- 自定义训练模型

#### 5. VTSHaiKangCamera - 相机 SDK
- 海康相机驱动集成
- 图像采集控制
- 参数配置管理

### 消息传递机制
- **事件驱动**: 扫描器、扫码设备通过事件通信
- **SerailDriver**: 串口通信驱动
- **MQTTC**: MQTT协议封装，用于MES通信
- **LibWinUsb**: USB设备监控

### 数据库
- **SQLite** 作为嵌入式数据库
- 数据库位置:
  - VTS: `VTSUI/db/lite.db`
  - OPPO: `OPPOYeWu/db/data.db`
- 使用 **PetaPoco.Compiled** 进行数据访问

### MES 集成
| MES类型 | 实现类 |
|---------|--------|
| 小灵MES | `XiaoLingMesDefine` |
| 多维MES | `DuweiMesDefine` |
| 鹰龙MES | `YingtongMesDefine` |

### 项目结构约定

```
ITS_master_new/
├── OPPOYeWu/              # OPPO测试系统
│   ├── FangDanTongDao/    # 主应用程序
│   ├── ZongGongChangBLL/  # 通用工厂业务逻辑 (8通道测试)
│   ├── ATEBLL/           # ATE测试业务逻辑
│   └── TKSpeedTest/      # 读卡速度测试
├── VTS*                   # 视觉测试系统栈
│   ├── VTSUI/            # UI层
│   ├── VTS.Flow/         # 工作流引擎
│   ├── VTSBll/           # 业务逻辑层
│   ├── VTSWorkNode/      # 处理节点
│   ├── VTS.Dao/          # 数据访问层
│   ├── VTSPaddleOCR/     # OCR引擎
│   ├── VTSOpenCvBll/     # OpenCV处理
│   ├── VTSHaiKangCamera/ # 相机驱动
│   └── VTSZebraPrinter/  # 打印机驱动
├── Common/                # 共享模型和工具
├── GongKongCanShu/        # 供应商驱动
│   ├── SheBei/           # 设备控制
│   ├── SaoMiaoSheBei/    # 扫描设备接口
│   ├── SerailDriver/     # 串口驱动
│   └── MesParam/         # MES参数模型
├── Upper.serilog/         # Serilog日志扩展
└── UPPERIOC/              # IoC容器
```

### 关键设计模式

| 模式 | 应用场景 |
|------|----------|
| **Singleton** | BLL类单例访问 |
| **Factory** | 设备工厂、MES工厂 |
| **Event-driven** | 扫描器/扫码事件 |
| **Pipeline** | 处理节点链式调用 |
| **IoC/DI** | 模块化依赖注入 |

### 权限管理
- 程序以管理员权限运行
- Windows账号集成
- 基于角色的访问控制

## 构建与运行

```bash
# 构建解决方案
msbuild ITS.sln /p:Configuration=Release

# 构建特定项目
msbuild OPPOYeWu/FangDanTongDao/OppOFrom.csproj /p:Configuration=Release
msbuild VTSUI/VTS.UI.csproj /p:Configuration=Release
```

## 注意事项

1. **管理员权限**: 程序需要管理员权限运行，非管理员启动时会自动提权
2. **单实例**: 通过Mutex和进程检查确保单实例运行
3. **.NET Framework 4.8**: OPPO系统需要安装.NET Framework 4.8
4. **数据库**: 首次运行时会自动创建数据库文件

## 技术特性

- 支持多语言界面 (中文/英文)
- 触摸屏手势解锁 (HHGestureUnlocking)
- 虚拟键盘集成
- 批量配置管理
- 实时数据上传MES
- 离线数据缓存
- 断点续传支持
