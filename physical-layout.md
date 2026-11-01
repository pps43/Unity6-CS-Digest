# 物理组织：工程、程序集与模块

本页说明仓库的物理结构：有哪些工程、每个目录被编译进哪个程序集、`artifacts/` 和 `External/` 里有什么，以及“模块”在这个仓库中的含义。逻辑职责见各子系统页面；全部模块清单见 [模块目录](module-catalog.md)。

父页面：[仓库概览](overview.md)

## 相关源码文件

- [`Projects/CSharp/UnityReferenceSource.sln`](../Projects/CSharp/UnityReferenceSource.sln)
- [`Projects/CSharp/UnityEngine.csproj`](../Projects/CSharp/UnityEngine.csproj)、[`Projects/CSharp/UnityEditor.csproj`](../Projects/CSharp/UnityEditor.csproj)
- [`artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs`](../artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs)
- [`Runtime/Export/AssemblyInfo.cs`](../Runtime/Export/AssemblyInfo.cs)
- [`Modules/Scripting/Attributes/BindingsAttributes.cs`](../Modules/Scripting/Attributes/BindingsAttributes.cs)
- [`Editor/IncrementalBuildPipeline/`](../Editor/IncrementalBuildPipeline/)

## 1. 解决方案与工程

解决方案包含 5 个工程 [`UnityReferenceSource.sln:6-15`](../Projects/CSharp/UnityReferenceSource.sln#L6-L15)：

| 工程 | 产物 | 内容 | 关键依赖 |
| --- | --- | --- | --- |
| `UnityEngine` | `UnityEngine` [`UnityEngine.csproj:8`](../Projects/CSharp/UnityEngine.csproj#L8) | `Runtime/`、引擎模块、部分 `External/`、`artifacts/generated` | NuGet 包引用 [`UnityEngine.csproj:8111-8121`](../Projects/CSharp/UnityEngine.csproj#L8111-L8121) |
| `UnityEditor` | `UnityEditor` [`UnityEditor.csproj:8`](../Projects/CSharp/UnityEditor.csproj#L8) | `Editor/Mono`、编辑器模块 | 引用 `UnityEngine` 工程与三个数据工程 [`UnityEditor.csproj:18061-18064`](../Projects/CSharp/UnityEditor.csproj#L18061-L18064)，以及 Bee、Cecil、ExCSS、Newtonsoft.Json 等预编译 DLL [`UnityEditor.csproj:18049-18060`](../Projects/CSharp/UnityEditor.csproj#L18049-L18060) |
| `ScriptCompilationBuildProgram.Data.gen` | 数据契约 | 编辑器传给脚本编译构建程序的数据结构 | 引用 `BeeBuildProgramCommon.Data` [`ScriptCompilationBuildProgram.Data.gen.csproj:30`](../Editor/IncrementalBuildPipeline/ScriptCompilationBuildProgram.Data/ScriptCompilationBuildProgram.Data.gen.csproj#L30) |
| `PlayerBuildProgramLibrary.Data.gen` | 数据契约 | 播放器构建程序的数据结构，如 `PlayerBuildConfig`、`Il2CppConfig` | — |
| `BeeBuildProgramCommon.Data.gen` | 数据契约 | 两类构建程序共享的数据 | — |

两个主工程的共同设置：`netstandard2.1`、`LangVersion 14.0`、允许 unsafe、关闭隐式包含（`EnableDefaultItems=false`），并且所有源文件都通过显式 `Compile Include` 链接 [`UnityEngine.csproj:8-17`](../Projects/CSharp/UnityEngine.csproj#L8-L17)。

### 每个 `.cs` 文件属于哪个工程

对两个主工程的 `Compile Include` 与磁盘文件做集合比较（命令见 [audit.md](audit.md)）：8692 个 `.cs` 文件中有 8689 个被两个主工程引用；剩下 3 个是 `Editor/IncrementalBuildPipeline/*/Data.cs`，由各自的数据工程编译。只有两个文件同时被两个主工程编译：`Runtime/Export/AssemblyInfo.cs` 与 `artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs`。

| 顶层目录 | `.cs` 数 | 编译进 | 说明 |
| --- | --- | --- | --- |
| `Runtime/` | 532 | `UnityEngine`（`Runtime/Export/AssemblyInfo.cs` 两边都编译） | 引擎核心 API：`Export`（460）、`Serialize`（26）、`Profiler`、`Transform` 等 |
| `Modules/` | 6542 | 按模块拆分到两个工程 | 约 170 个模块，见 [模块目录](module-catalog.md) |
| `Editor/Mono/` | 1520 | `UnityEditor` | 编辑器核心：窗口、Inspector、编译、构建、场景视图等 |
| `Editor/IncrementalBuildPipeline/` | 3 | 三个数据工程 | 构建程序数据契约 |
| `External/` | 88 | 两个工程都有 | 第三方或 Unity 内部共享代码，见下文 |
| `artifacts/` | 6 | 两个工程 | 生成代码与预编译工具 DLL |
| `Tools/` | 1 | `UnityEditor` | `SharedWithEditor.cs` 与 Bee DLL |

目录计数来自对仓库的文件统计；工程归属来自对 `Compile Include` 路径前缀的分组统计。

## 2. 预定义符号与“编辑器视角”

`UnityEngine.csproj` 的 `DefineConstants` 同时包含 `UNITY_EDITOR`、`COMBINED_ASSEMBLIES`、`ALLOW_IMPLICIT_INTERNAL_VISIBILITY`，以及多个平台的 `*_API` 符号 [`UnityEngine.csproj:13`](../Projects/CSharp/UnityEngine.csproj#L13)；`UnityEditor.csproj` 类似，另含 `UNITY_EDITOR_LINUX` 等 [`UnityEditor.csproj:13`](../Projects/CSharp/UnityEditor.csproj#L13)。

源码中没有任何 `#if` 指令（全仓库搜索为 0 条）。结合上面的符号，可以推断源码在发布前已按“编辑器内”配置做过条件编译展开。详见 [仓库概览 §3.2](overview.md#32-源码已按编辑器配置预处理推断)。

另有两个与代码生成相关的设置：

- `UnityEnableAutoStaticsCleanupCodeGen=false`，并通过 `CompilerVisibleProperty` 暴露给生成器 [`UnityEngine.csproj:24`](../Projects/CSharp/UnityEngine.csproj#L24)、[`UnityEngine.csproj:8094-8096`](../Projects/CSharp/UnityEngine.csproj#L8094-L8096)。这与 [生命周期作用域](runtime/lifecycle-scopes.md) 中的静态字段自动清理生成器有关。
- 两个工程都引用了一个位于 `/tmp/uitoolkit_gen_zua5apap/` 的 UI Toolkit 源生成器 [`UnityEngine.csproj:8108-8110`](../Projects/CSharp/UnityEngine.csproj#L8108-L8110)，该路径不在仓库中。

## 3. 模块化与“单体”程序集

在真实 Unity 构建中，`Modules/<Name>` 会编译成独立程序集，例如 `UnityEngine.PhysicsModule`。本仓库能找到三类证据：

1. 生成的 `InternalsVisibleTo.cs` 把内部成员开放给一长串模块程序集，如 `UnityEngine.CoreModule`、`UnityEngine.IMGUIModule`、`UnityEngine.PhysicsModule` [`InternalsVisibleTo.cs:5-25`](../artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs#L5-L25)。
2. `Runtime/Export/AssemblyInfo.cs` 注明“构建系统会把此文件包含进所有模块（Runtime 和 Editor）”，并为每个模块程序集打上 `UnityEngineModuleAssembly` 标记 [`AssemblyInfo.cs:5-10`](../Runtime/Export/AssemblyInfo.cs#L5-L10)。
3. `VisibleToOtherModulesAttribute` 用于控制模块之间可见的内部类型和成员 [`BindingsAttributes.cs:9-22`](../Modules/Scripting/Attributes/BindingsAttributes.cs#L9-L22)。代码中经常写明可见范围，例如 `EditorApplication.CallDelayed` 只对 Burst 和 ProjectAuditor 模块可见 [`EditorApplication.cs:253-254`](../Editor/Mono/EditorApplication.cs#L253-L254)。

参考工程则通过 `COMBINED_ASSEMBLIES` 把所有模块合并成两个程序集。工程的 `GetTargetPath` 目标指向 `artifacts/MonolithicEditorDlls/Debug/ManagedMonolithic/BoundToNative/UnityEngine.dll` [`UnityEngine.csproj:8100-8107`](../Projects/CSharp/UnityEngine.csproj#L8100-L8107)，从名字看是“已绑定到原生”的单体 DLL；仓库中不存在这个目录。

**阅读影响：** `internal` 成员在参考工程里几乎处处可见，但在真实构建中跨模块访问受 `VisibleToOtherModules` 限制。判断一个内部 API 能否被某个模块使用时，应以特性参数为准，不要以参考工程能编译为准。

### 模块目录约定

模块内部子目录没有统一模板。按一级子目录名出现的频率统计，最常见的是 `Managed`（40 个模块）、`Public`（29）、`ScriptBindings`（25）、`Editor`（14）。`Modules/` 下共有 92 个 `AssemblyInfo.cs`，分布在 80 个模块中，主要用于声明 `InternalsVisibleTo`，例如物理模块向其测试程序集开放内部成员 [`Modules/Physics/AssemblyInfo.cs:7-10`](../Modules/Physics/AssemblyInfo.cs#L7-L10)。

编辑器模块通常以 `Editor` 结尾，例如 `PhysicsEditor`；但也有例外，如 `PackageManager`、`QuickSearch`、`UIBuilder` 属于编辑器，`AssetDatabase` 也属于编辑器。实际归属以 [模块目录](module-catalog.md) 中按工程统计的结果为准。

## 4. `artifacts/`：生成代码与工具

| 路径 | 内容 | 证据 |
| --- | --- | --- |
| `artifacts/generated/UnityEvent/UnityEvent_0..4.cs` | 由模板生成的 `UnityEvent<T0..T3>` 泛型类，注释说明通过 `./jam GenerateUnityEventClasses` 重新生成 | [`UnityEvent_1.cs:6-9`](../artifacts/generated/UnityEvent/UnityEvent_1.cs#L6-L9) |
| `artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs` | 生成的程序集友元声明，两个工程都编译 | [`InternalsVisibleTo.cs:5`](../artifacts/MonolithicEditorDlls/InternalsVisibleTo.cs#L5) |
| `artifacts/Stevedore/...` | `Unity.CompilationPipeline.Common.dll`、`UnityEditor.MSBuild.dll`、`AsmDefToCSProjLib.dll` 等预编译依赖 | [`UnityEditor.csproj:18049-18052`](../Projects/CSharp/UnityEditor.csproj#L18049-L18052) |

## 5. `External/` 与 `Tools/`

| 路径 | 编译进 | 作用 |
| --- | --- | --- |
| `External/ScriptingCore/Unity.Scripting` | `UnityEngine` | 代码生命周期管理（作用域、静态清理）、诊断。见 [生命周期作用域](runtime/lifecycle-scopes.md) |
| `External/baselib/.../CSharp` | `UnityEngine` | baselib 的 C# 绑定（线程、Socket、文件 IO 等），文件名带 `.gen.binding.cs` |
| `External/unitytls` | `UnityEngine` | TLS 绑定 |
| `External/LibTessDotNet`、`External/ClipperManaged` | `UnityEngine` | 第三方几何库（曲面细分、多边形裁剪） |
| `External/AMDUnityPlugin`、`External/NVUnityPlugin` | `UnityEngine` | GPU 厂商插件脚本 |
| `External/CoreCLR/SourceCopiesToRemove` | `UnityEngine` | `DynamicDependencyAttribute` 等 BCL 特性的源码副本 |
| `External/il2cpp`、`External/NiceIO`、`External/JsonParsers` | `UnityEditor` | IL2CPP/链接器集成共享代码、路径库、MiniJSON |
| `External/Unity.Cecil`、`External/ExCSS`、`External/nugets/...` | 以 DLL 引用 | Cecil（IL 读写）、ExCSS（USS 解析，见 [样式、布局与渲染](ui-toolkit/styles-layout-rendering.md)）、Newtonsoft.Json |
| `Tools/Bee/build/Distribution/*` | 以 DLL 引用 | Bee 构建驱动，见 [脚本编译](editor/script-compilation.md) |
| `Tools/Unity.BuildSystem/SharedWithEditor` | `UnityEditor` | 构建系统程序与播放器构建程序共享的代码 [`SharedWithEditor.cs:5`](../Tools/Unity.BuildSystem/SharedWithEditor/SharedWithEditor.cs#L5) |

归属来自 `Compile Include` 统计；DLL 引用见 [`UnityEditor.csproj:18049-18060`](../Projects/CSharp/UnityEditor.csproj#L18049-L18060)。

## 6. IncrementalBuildPipeline 数据契约

编辑器不在进程内直接调用编译器，而是把描述数据交给外部“构建程序”。`ScriptCompilationData` 描述一次脚本编译：程序集列表、代码生成程序集、dotnet/Roslyn 路径、输出目录、是否提取 `RuntimeInitializeOnLoad` 等 [`Data.cs:14-32`](../Editor/IncrementalBuildPipeline/ScriptCompilationBuildProgram.Data/Data.cs#L14-L32)。`AssemblyData` 描述单个程序集的源文件、宏定义、引用（以整数索引表示）、分析器和语言版本 [`Data.cs:40-60`](../Editor/IncrementalBuildPipeline/ScriptCompilationBuildProgram.Data/Data.cs#L40-L60)。

构建程序本身（`ScriptCompilationBuildProgram.exe`）不在仓库中；编辑器只知道它的位置 [`EditorCompilation.cs:977-982`](../Editor/Mono/Scripting/ScriptCompilation/EditorCompilation.cs#L977-L982)。数据流见 [脚本编译](editor/script-compilation.md)。

## 7. 构建状态与缺失依赖

本 Wiki **没有执行**任何构建命令。可以从工程文件直接确认的缺失项包括：

- `AdditionalFiles` 引用的 `Tools/Unity.BuildSystem/Unity.BuildSystem/BannedSymbolsFiles/BannedSymbols.RuntimeModules.txt` 不存在 [`UnityEngine.csproj:8086-8088`](../Projects/CSharp/UnityEngine.csproj#L8086-L8088)。
- 源生成器路径 `/tmp/uitoolkit_gen_zua5apap/...` 不存在 [`UnityEngine.csproj:8108-8110`](../Projects/CSharp/UnityEngine.csproj#L8108-L8110)。
- `GetTargetPath` 指向的 `artifacts/MonolithicEditorDlls/Debug/...` 不存在 [`UnityEngine.csproj:8100-8107`](../Projects/CSharp/UnityEngine.csproj#L8100-L8107)。

此外，`LangVersion 14.0` 需要较新的 .NET SDK；分析环境只安装了 6.0 和 9.0 SDK（`dotnet --list-sdks` 输出）。即使能编译，产物也无法运行，因为所有 `extern` 方法都需要 Unity 原生运行时。把这个解决方案当作**代码导航工程**使用更现实，见 [阅读与导航指南](getting-started.md)。
