# 阅读与导航指南

本页说明如何在本地有效阅读这份参考源码：用什么工具打开，哪些命令有用，以及如何沿着托管/原生边界追踪一个功能。它不讨论如何使用 Unity。

父页面：[仓库概览](overview.md) · 相关：[物理组织](physical-layout.md)、[绑定层](runtime/bindings.md)

## 1. 前置条件与现实预期

- README 只给出了解决方案位置 `Projects/CSharp/UnityReferenceSource.sln` [`README.md:27`](../README.md#L27)，没有构建说明。
- 本 Wiki **没有执行**构建。工程引用了仓库中不存在的源生成器路径和 BannedSymbols 文件，也需要支持 `LangVersion 14.0` 的 SDK，详见 [物理组织 §7](physical-layout.md#7-构建状态与缺失依赖)。
- 即使编译成功，代码也不能独立运行：大量方法是 `extern`，需要 Unity 原生运行时提供实现（例如 [`UnityEngineObject.bindings.cs:692-704`](../Runtime/Export/Scripting/UnityEngineObject.bindings.cs#L692-L704)）。

因此建议把解决方案当作**索引工程**：在支持 SDK 风格工程的 IDE 中打开，用于跳转定义、查找引用；IDE 报出的缺失分析器或文件错误一般不影响导航。这条建议是推断，未在本环境中验证。

## 2. 推荐的命令行检索

以下命令基于 ripgrep，均在仓库根目录执行；它们是本 Wiki 研究时实际使用的检索方式。

| 目的 | 命令 |
| --- | --- |
| 找某个 C# API 的原生实现名 | `rg -n "FreeFunction\|NativeMethod\|NativeHeader" Runtime/Export/Scripting/GameObject.bindings.cs` |
| 找原生会回调的托管入口 | `rg -n "RequiredByNativeCode" -A 2 Editor/Mono/EditorApplication.cs` |
| 找编辑器如何发现用户类型 | `rg -n "TypeCache.GetTypesWithAttribute<\|TypeCache.GetTypesDerivedFrom<" --glob *.cs` |
| 找代码重载时的初始化/清理钩子 | `rg -n "\[OnCodeLoaded\]\|\[OnCodeUnloading\]\|AutoStaticsCleanupOnCodeReload" --glob *.cs` |
| 判断文件属于哪个程序集 | `rg -n "<文件名>" Projects/CSharp/UnityEngine.csproj Projects/CSharp/UnityEditor.csproj` |

## 3. 追踪一个功能的通用方法

### 3.1 从公共 API 追到原生

1. 找到 C# 声明，通常在 `*.bindings.cs` 文件中。
2. 读类型上的 `[NativeHeader]`，它给出 C++ 头文件路径，例如 `Runtime/Export/Scripting/GameObject.bindings.h` [`GameObject.bindings.cs:24`](../Runtime/Export/Scripting/GameObject.bindings.cs#L24)。
3. 读方法上的 `[FreeFunction]`/`[NativeMethod]`，它给出 C++ 函数名以及是否带隐式 `this`、是否线程安全、是否会抛异常，例如 `GetComponent` 映射到 `GameObjectBindings::GetComponentFromType` [`GameObject.bindings.cs:42-43`](../Runtime/Export/Scripting/GameObject.bindings.cs#L42-L43)。
4. 到这里 C# 侧的线索就结束了。特性语义见 [绑定层](runtime/bindings.md)。

### 3.2 从原生回调追到托管

原生代码调用托管代码时，目标方法几乎都标了 `[RequiredByNativeCode]`。它们通常是 `private static` 或 `internal static`，在 IDE 中“查找引用”找不到调用方。例如编辑器每帧调用 `EditorApplication.Internal_CallUpdateFunctions` [`EditorApplication.cs:503-511`](../Editor/Mono/EditorApplication.cs#L503-L511)。看到没有托管调用方的 `[RequiredByNativeCode]` 方法时，应把它当作运行时入口，而不是死代码。

### 3.3 从编辑器扩展点追到发现逻辑

编辑器大量通过特性或接口发现用户代码，常见入口是 `TypeCache`。例如：

- `[CustomEditor]`：[`CustomEditorAttributes.cs:169-200`](../Editor/Mono/CustomEditorAttributes.cs#L169-L200)
- `[InitializeOnLoadMethod]`：[`EditorAssemblies.cs:154-178`](../Editor/Mono/EditorAssemblies.cs#L154-L178)
- `[ScriptedImporter]`：[`ScriptedImporter.cs:60-138`](../Modules/AssetPipelineEditor/Public/ScriptedImporter.cs#L60-L138)
- 构建回调接口 `IOrderedCallback`：[`BuildPipelineInterfaces.cs:878-904`](../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L878-L904)

## 4. 首次阅读路线示例：`EditorWindow.Update` 是怎么被调用的

下面这条真实路径可以作为练习，每一步都能在仓库中找到：

1. 原生编辑器循环调用 `EditorApplication.Internal_CallUpdateFunctions`，它遍历 `EditorApplication.update` 委托 [`EditorApplication.cs:503-511`](../Editor/Mono/EditorApplication.cs#L503-L511)。
2. 当某个 `EditorWindow` 成为 `HostView` 的当前视图时，如果该窗口类型声明了 `Update` 方法，`HostView.RegisterSelectedPane` 会把 `SendUpdate` 挂到 `EditorApplication.update` 上 [`HostView.cs:617-632`](../Editor/Mono/HostView.cs#L617-L632)。
3. `SendUpdate` 调用通过反射创建的 `m_Update` 委托 [`HostView.cs:736-738`](../Editor/Mono/HostView.cs#L736-L738)；委托在 `CreateDelegates` 中按方法名 `"Update"` 创建 [`HostView.cs:154-167`](../Editor/Mono/HostView.cs#L154-L167)。

完整背景见 [窗口系统](editor/window-system.md)。

## 5. 常见陷阱

| 陷阱 | 说明 |
| --- | --- |
| 把编辑器分支当成播放器行为 | 源码已按编辑器配置预处理（推断），播放器专用代码可能缺失。见 [仓库概览 §3.2](overview.md#32-源码已按编辑器配置预处理推断)。 |
| 认为 `internal` 可随意跨模块访问 | 真实构建按模块拆分，跨模块可见性由 `VisibleToOtherModules` 控制。见 [物理组织 §3](physical-layout.md#3-模块化与单体程序集)。 |
| 期待找到测试 | 仓库不包含测试；文档注释里的 `Tests/...` 路径（如 [`IJob.cs:43`](../Modules/ManagedKernel/Managed/Jobs/IJob.cs#L43)）在仓库中不存在。 |
| 认为静态字段会跨代码重载保留 | 许多静态字段标了 `[AutoStaticsCleanupOnCodeReload]`，会在重载时被清理，例如 `EditorApplication.update` [`EditorApplication.cs:207-211`](../Editor/Mono/EditorApplication.cs#L207-L211)。见 [生命周期作用域](runtime/lifecycle-scopes.md)。 |
| 用 `== null` 判断 `UnityEngine.Object` 时只想到 C# 引用 | `==` 被重载为检查原生对象是否存活。见 [对象模型](runtime/object-model.md)。 |
