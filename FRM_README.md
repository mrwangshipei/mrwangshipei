# FrmControl

一个 WinForms 控件库，基于 .NET Framework 4.8，用于快速开发桌面应用程序。

## 构建

```bash
dotnet build        # Debug 构建
dotnet build -c Release  # Release 构建
```

## 目录结构

| 目录 | 说明 |
|------|------|
| `FrmBase_/` | 基础窗体类，提供拖拽移动、渐变背景、图标等功能 |
| `C/` | 自定义控件（按钮、面板、复选框、进度条、加载、指示灯、提示等） |
| `Frm/` | 对话框窗体（输入框、选择框、密码框、加载遮罩、Toast 提示等） |
| `FrmPremission/` | 权限管理模块（用户、角色、权限编辑） |
| `Animation/` | 动画基类 |
| `Util/` | 工具类 |
| `thame/` | 主题颜色配置 |

## 核心组件

### 基础窗体

- `FrmBaseForm` - 带标题栏拖拽、渐变背景、图标支持的窗体基类
- `CBaseControl` - 提供 `DoInvoke()` 方法用于线程安全调用

### 控件

- `CBtn_` - 自定义按钮系列（Apple3Btn, FrmLineBtn, IndustrialTierButton, FrmLinerGidentBtn 等）
- `CPanel_` - 自定义面板
- `CCheckBox_` - 自定义复选框
- `ProgressBar/` - 进度条系列（GradientProgressBar, VerticalProgressBar, TextProgressBar, YieldProgressBar）
- `Loading/` - 加载控件
- `Light/` - 状态指示灯（StatusLamp, StatusLampLabel）
- `tip/` - 提示服务（IndustrialTipService, TipForm, TipInvoker）

### 对话框

- `LoadingOverlay` - 加载遮罩，支持动画显示/隐藏
- `WarningOverlay` - 警告遮罩
- `FrmLitleScreenToast` - 小屏幕 Toast 提示
- `FrmDialog` - 自定义对话框
- `FrmInput` / `FrmSelect` / `FrmPwd` - 输入/选择/密码对话框

## 使用示例

### 显示加载遮罩

```csharp
LoadingOverlay.Show(this, "加载中...");
LoadingOverlay.Hide(this);
```

### 使用基础窗体

```csharp
public partial class MyForm : FrmBaseForm
{
    // 自动支持拖拽移动、渐变背景
}
```

### 线程安全调用

```csharp
this.DoInvoke(() => {
    label.Text = "更新内容";
});
```

## 依赖

- .NET Framework 4.8
- System.ValueTuple 4.5.0
- UPPERIOC 2.0.4.103
