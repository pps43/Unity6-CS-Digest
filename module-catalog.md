# 模块目录

`Modules/` 下共有 166 个模块目录。本页列出全部模块的程序集归属、文件数和主要命名空间，并标明本 Wiki 是否深入分析。它用于定位代码，也是 [audit.md](audit.md) 反向覆盖检查的依据。

父页面：[物理组织](physical-layout.md)

## 数据来源与口径

- **程序集**：如果模块下任一文件出现在 `UnityEngine.csproj` 的 `Compile Include` 中，记为 `UnityEngine`，否则记为 `UnityEditor`。统计中没有模块同时属于两个工程（见 [物理组织 §1](physical-layout.md#1-解决方案与工程)）。
- **`.cs` 数**：模块目录下递归统计的 `.cs` 文件数。
- **主要命名空间**：对模块内所有文件的第一条 `namespace` 声明计数，取出现次数最多的两个。只有一个文件、且没有命名空间声明的模块显示为“—”。
- **分类**：分类标题由本 Wiki 根据模块名和命名空间整理，**不是**仓库中的官方分组。
- **Wiki 覆盖**：
  - 链接页面：该页面分析了模块的核心机制。
  - “部分”：只分析了一部分职责。
  - “仅提及”：页面中出现，但没有展开分析。
  - “目录级”：只在本表中出现，没有分析其行为。

以下表格由统计命令生成后整理，统计方法记录在 [audit.md](audit.md)。

## 运行时：脚本基础、序列化与数据

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [Scripting](../Modules/Scripting/) | UnityEngine | 36 | `UnityEngine.Bindings`、`UnityEngine` | [绑定层](runtime/bindings.md) |
| [Marshalling](../Modules/Marshalling/) | UnityEngine | 4 | `UnityEngine` | [绑定层](runtime/bindings.md)（仅提及） |
| [ManagedKernel](../Modules/ManagedKernel/) | UnityEngine | 93 | `Unity.Collections`、`Unity.Collections.LowLevel.Unsafe` | [Jobs 与原生容器](runtime/jobs-and-collections.md) |
| [Burst](../Modules/Burst/) | UnityEngine | 52 | `Unity.Burst.Intrinsics`、`Unity.Burst` | [Jobs 与原生容器](runtime/jobs-and-collections.md)（仅提及） |
| [Mathematics](../Modules/Mathematics/) | UnityEngine | 88 | `Unity.Mathematics`、`Unity.Mathematics.Geometry` | 目录级 |
| [Properties](../Modules/Properties/) | UnityEngine | 51 | `Unity.Properties`、`Unity.Properties.Internal` | 目录级 |
| [JSONSerialize](../Modules/JSONSerialize/) | UnityEngine | 1 | `UnityEngine` | [序列化](runtime/serialization.md)（仅提及） |
| [Identifiers](../Modules/Identifiers/) | UnityEngine | 1 | `UnityEngine.Identifiers` | 目录级 |
| [Hierarchy](../Modules/Hierarchy/) | UnityEngine | 28 | `Unity.Hierarchy` | 目录级 |
| [HierarchyCore](../Modules/HierarchyCore/) | UnityEngine | 33 | `Unity.Hierarchy` | 目录级 |
| [Subsystems](../Modules/Subsystems/) | UnityEngine | 19 | `UnityEngine`、`UnityEngine.SubsystemsImplementation` | 目录级 |
| [Insights](../Modules/Insights/) | UnityEngine | 7 | `UnityEngine.ManagedCapture.Internal`、`UnityEngine.ManagedCapture` | 目录级 |
| [Streaming](../Modules/Streaming/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [CrashReporting](../Modules/CrashReporting/) | UnityEngine | 1 | `UnityEngine.CrashReportHandler` | 目录级 |
| [PerformanceReporting](../Modules/PerformanceReporting/) | UnityEngine | 1 | `UnityEngine.Analytics` | 目录级 |
| [Localization](../Modules/Localization/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [LocalizationRuntime](../Modules/LocalizationRuntime/) | UnityEngine | 65 | `Unity.Localization`、`Unity.Localization.Components` | 目录级 |
| [SmartStrings](../Modules/SmartStrings/) | UnityEngine | 75 | `Unity.SmartStrings.Extensions`、`Unity.SmartStrings.Core.Parsing` | 目录级 |
| [UnityConsent](../Modules/UnityConsent/) | UnityEngine | 3 | `UnityEngine.UnityConsent` | 目录级 |
| [ImageConversion](../Modules/ImageConversion/) | UnityEngine | 2 | `UnityEngine` | 目录级 |
| [ScreenCapture](../Modules/ScreenCapture/) | UnityEngine | 2 | `UnityEngine` | 目录级 |
| [AssetBundle](../Modules/AssetBundle/) | UnityEngine | 9 | `UnityEngine` | 目录级 |
| [ContentLoad](../Modules/ContentLoad/) | UnityEngine | 9 | `Unity.Loading`、`Unity.Loading.LowLevel` | 目录级 |

## 运行时：图形、2D、文本与特效

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [Grid](../Modules/Grid/) | UnityEngine | 3 | `UnityEngine` | 目录级 |
| [Tilemap](../Modules/Tilemap/) | UnityEngine | 9 | `UnityEngine.Tilemaps`、`UnityEngine` | 目录级 |
| [SpriteMask](../Modules/SpriteMask/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [SpriteShape](../Modules/SpriteShape/) | UnityEngine | 2 | `UnityEngine.U2D` | 目录级 |
| [U2DRuntime](../Modules/U2DRuntime/) | UnityEngine | 11 | `UnityEngine.U2D.UTess`、`UnityEngine.U2D.Runtime` | 目录级 |
| [RenderAs2D](../Modules/RenderAs2D/) | UnityEngine | 3 | `UnityEngine` | 目录级 |
| [VectorGraphics](../Modules/VectorGraphics/) | UnityEngine | 15 | `Unity.VectorGraphics` | 目录级 |
| [VFX](../Modules/VFX/) | UnityEngine | 9 | `UnityEngine.VFX`、`UnityEngine.Experimental.VFX` | 目录级 |
| [VirtualTexturing](../Modules/VirtualTexturing/) | UnityEngine | 2 | `UnityEngine.Rendering` | 目录级 |
| [Terrain](../Modules/Terrain/) | UnityEngine | 11 | `UnityEngine`、`UnityEngine.TerrainTools` | 目录级 |
| [TerrainPhysics](../Modules/TerrainPhysics/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [ParticleSystem](../Modules/ParticleSystem/) | UnityEngine | 10 | `UnityEngine`、`UnityEngine.ParticleSystemJobs` | 目录级 |
| [Wind](../Modules/Wind/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [Cloth](../Modules/Cloth/) | UnityEngine | 2 | `UnityEngine` | 目录级 |
| [MetalFX](../Modules/MetalFX/) | UnityEngine | 2 | `UnityEngine.Rendering.MetalFX` | 目录级 |
| [AMD](../Modules/AMD/) | UnityEngine | 1 | `UnityEngine.AMD` | 目录级 |
| [NVIDIA](../Modules/NVIDIA/) | UnityEngine | 1 | `UnityEngine.NVIDIA` | 目录级 |
| [ClusterRenderer](../Modules/ClusterRenderer/) | UnityEngine | 2 | `UnityEngine` | 目录级 |
| [TextRendering](../Modules/TextRendering/) | UnityEngine | 5 | `UnityEngine` | 目录级 |
| [TextCoreFontEngine](../Modules/TextCoreFontEngine/) | UnityEngine | 9 | `UnityEngine.TextCore.LowLevel`、`UnityEngine.TextCore` | 目录级 |
| [TextCoreTextEngine](../Modules/TextCoreTextEngine/) | UnityEngine | 72 | `UnityEngine.TextCore.Text`、`UnityEngine.TextCore` | 目录级 |

## 运行时：物理、动画、音视频与 AI

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [Physics](../Modules/Physics/) | UnityEngine | 37 | `UnityEngine`、`UnityEngine.LowLevelPhysics` | 目录级 |
| [Physics2D](../Modules/Physics2D/) | UnityEngine | 3 | `UnityEngine`、`UnityEngine.LowLevelPhysics2D` | 目录级 |
| [PhysicsCore2D](../Modules/PhysicsCore2D/) | UnityEngine | 76 | `Unity.U2D.Physics` | 目录级 |
| [Vehicles](../Modules/Vehicles/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [Animation](../Modules/Animation/) | UnityEngine | 52 | `UnityEngine.Animations`、`UnityEngine` | 目录级 |
| [Director](../Modules/Director/) | UnityEngine | 9 | `UnityEngine.Playables` | 目录级 |
| [TimelineFoundation](../Modules/TimelineFoundation/) | UnityEngine | 153 | `Unity.Timeline.Foundation.ViewModel`、`Unity.Timeline.Foundation.Model` | 目录级 |
| [Audio](../Modules/Audio/) | UnityEngine | 32 | `UnityEngine.Audio`、`UnityEngine.Experimental.Audio` | 目录级 |
| [DSPGraph](../Modules/DSPGraph/) | UnityEngine | 8 | `Unity.Audio` | 目录级 |
| [Video](../Modules/Video/) | UnityEngine | 7 | `UnityEngine.Experimental.Video`、`UnityEngine.Video` | 目录级 |
| [AI](../Modules/AI/) | UnityEngine | 17 | `UnityEngine.AI`、`Unity.AI.Navigation.LowLevel` | 目录级 |

## 运行时：输入与 UI

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [Input](../Modules/Input/) | UnityEngine | 5 | `UnityEngineInternal.Input` | 目录级 |
| [InputLegacy](../Modules/InputLegacy/) | UnityEngine | 2 | `UnityEngine` | 目录级 |
| [InputForUI](../Modules/InputForUI/) | UnityEngine | 18 | `UnityEngine.InputForUI` | [UI Toolkit 事件](ui-toolkit/events.md)（仅提及） |
| [ClusterInput](../Modules/ClusterInput/) | UnityEngine | 1 | `UnityEngine` | 目录级 |
| [IMGUI](../Modules/IMGUI/) | UnityEngine | 35 | `UnityEngine` | [IMGUI](editor/imgui.md) |
| [UIElements](../Modules/UIElements/) | UnityEngine | 665 | `UnityEngine.UIElements`、`UnityEngine.UIElements.UIR` | [UI Toolkit](ui-toolkit/index.md) 及子页面 |
| [UI](../Modules/UI/) | UnityEngine | 8 | `UnityEngine`、`UnityEngine.UI` | 目录级 |
| [UICommon](../Modules/UICommon/) | UnityEngine | 5 | `UnityEngine` | 目录级 |
| [Accessibility](../Modules/Accessibility/) | UnityEngine | 14 | `UnityEngine.Accessibility` | 目录级 |

## 运行时：网络、服务与平台

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [UnityWebRequest](../Modules/UnityWebRequest/) | UnityEngine | 10 | `UnityEngine.Networking`、`UnityEngine` | 目录级 |
| [UnityWebRequestAssetBundle](../Modules/UnityWebRequestAssetBundle/) | UnityEngine | 2 | `UnityEngine.Networking` | 目录级 |
| [UnityWebRequestAudio](../Modules/UnityWebRequestAudio/) | UnityEngine | 2 | `UnityEngine.Networking` | 目录级 |
| [UnityWebRequestTexture](../Modules/UnityWebRequestTexture/) | UnityEngine | 2 | `UnityEngine.Networking` | 目录级 |
| [UnityWebRequestWWW](../Modules/UnityWebRequestWWW/) | UnityEngine | 2 | `UnityEngine`、`UnityEngine.Networking` | 目录级 |
| [UnityCurl](../Modules/UnityCurl/) | UnityEngine | 2 | `Unity.Curl` | 目录级 |
| [UnityAnalytics](../Modules/UnityAnalytics/) | UnityEngine | 7 | `UnityEngine.Analytics`、`UnityEngine` | 目录级 |
| [UnityAnalyticsCommon](../Modules/UnityAnalyticsCommon/) | UnityEngine | 15 | `UnityEditor.Analytics`、`UnityEngine.Analytics` | 目录级 |
| [UnityConnect](../Modules/UnityConnect/) | UnityEngine | 2 | `UnityEngine.Advertisements`、`UnityEngine.Connect` | 目录级 |
| [GameCenter](../Modules/GameCenter/) | UnityEngine | 2 | `UnityEngine`、`UnityEngine.SocialPlatforms.Impl` | 目录级 |
| [AndroidJNI](../Modules/AndroidJNI/) | UnityEngine | 32 | `UnityEngine.Android`、`UnityEngine` | 目录级 |
| [XR](../Modules/XR/) | UnityEngine | 15 | `UnityEngine.XR`、`UnityEngine.XR.Provider` | 目录级 |
| [AdaptivePerformance](../Modules/AdaptivePerformance/) | UnityEngine | 60 | `UnityEngine.AdaptivePerformance`、`UnityEngine.AdaptivePerformance.Basic` | 目录级 |
| [Multiplayer](../Modules/Multiplayer/) | UnityEngine | 8 | `Unity.Multiplayer`、`Unity.Multiplayer.PlayMode` | 目录级 |

## 编辑器：资源、包、构建与工作流

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [AssetDatabase](../Modules/AssetDatabase/) | UnityEditor | 17 | `UnityEditor`、`UnityEditor.Experimental` | [资源导入](editor/asset-pipeline.md)（部分） |
| [AssetPipelineEditor](../Modules/AssetPipelineEditor/) | UnityEditor | 28 | `UnityEditor`、`UnityEditor.AssetImporters` | [资源导入](editor/asset-pipeline.md) |
| [AssetPackageEditor](../Modules/AssetPackageEditor/) | UnityEditor | 10 | `UnityEditor.AssetPackage`、`UnityEditor` | 目录级 |
| [AssetPackageUIEditor](../Modules/AssetPackageUIEditor/) | UnityEditor | 11 | `UnityEditor.AssetPackage`、`UnityEditor` | 目录级 |
| [PackageManager](../Modules/PackageManager/) | UnityEditor | 76 | `UnityEditor.PackageManager`、`UnityEditor.PackageManager.Requests` | [包管理器](editor/package-manager.md) |
| [PackageManagerUI](../Modules/PackageManagerUI/) | UnityEditor | 395 | `UnityEditor.PackageManager.UI.Internal`、`UnityEditor.PackageManager.UI` | [包管理器](editor/package-manager.md)（部分） |
| [QuickInstall](../Modules/QuickInstall/) | UnityEditor | 8 | `UnityEditor.QuickInstall`、`UnityEditor.InAppPurchasing` | 目录级 |
| [BuildProfileEditor](../Modules/BuildProfileEditor/) | UnityEditor | 57 | `UnityEditor.Build.Profile`、`UnityEditor.Build.Profile.Elements` | [播放器构建](editor/build-pipeline.md)（仅提及） |
| [BuildReportingEditor](../Modules/BuildReportingEditor/) | UnityEditor | 23 | `UnityEditor.Build.Reporting`、`UnityEditor.Build` | [播放器构建](editor/build-pipeline.md)（仅提及） |
| [BuildAnalysis](../Modules/BuildAnalysis/) | UnityEditor | 52 | `UnityEditor.Build.Analysis` | 目录级 |
| [ContentBuild](../Modules/ContentBuild/) | UnityEditor | 31 | `UnityEditor.Build.Content`、`UnityEditor.Build` | 目录级 |
| [ContentLoadEditor](../Modules/ContentLoadEditor/) | UnityEditor | 4 | `UnityEditor` | 目录级 |
| [PlayModeEditor](../Modules/PlayModeEditor/) | UnityEditor | 17 | `Unity.PlayMode.Editor` | 目录级 |
| [SafeMode](../Modules/SafeMode/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [UMPE](../Modules/UMPE/) | UnityEditor | 9 | `UnityEditor.MPE`、`Unity.MPE` | 目录级 |
| [Progress](../Modules/Progress/) | UnityEditor | 2 | `UnityEditor` | 目录级 |
| [ShortcutManagerEditor](../Modules/ShortcutManagerEditor/) | UnityEditor | 31 | `UnityEditor.ShortcutManagement` | 目录级 |
| [QuickSearch](../Modules/QuickSearch/) | UnityEditor | 246 | `UnityEditor.Search`、`UnityEditor.Search.Providers` | 目录级 |
| [EditorToolbar](../Modules/EditorToolbar/) | UnityEditor | 43 | `UnityEditor.Toolbars`、`UnityEditor.Search` | 目录级 |
| [SceneView](../Modules/SceneView/) | UnityEditor | 3 | `UnityEditor`、`UnityEditor.Toolbars` | 目录级 |
| [GridAndSnap](../Modules/GridAndSnap/) | UnityEditor | 3 | `UnityEditor.Snap` | 目录级 |
| [PresetsEditor](../Modules/PresetsEditor/) | UnityEditor | 9 | `UnityEditor.Presets` | 目录级 |
| [PresetsUIEditor](../Modules/PresetsUIEditor/) | UnityEditor | 4 | `UnityEditor.Presets` | 目录级 |
| [SceneTemplateEditor](../Modules/SceneTemplateEditor/) | UnityEditor | 18 | `UnityEditor.SceneTemplate`、`UnityEditor.Search` | 目录级 |
| [HierarchyEditor](../Modules/HierarchyEditor/) | UnityEditor | 28 | `Unity.Hierarchy.Editor`、`Unity.Hierarchy.Editor.Tests` | 目录级 |
| [DiagnosticsEditor](../Modules/DiagnosticsEditor/) | UnityEditor | 2 | `UnityEditor` | 目录级 |
| [Licensing](../Modules/Licensing/) | UnityEditor | 52 | `UnityEditor.Licensing.UI.Events.Windows`、`UnityEditor.Licensing.UI.Events.Buttons` | 目录级 |
| [CloudServicesSettingsEditor](../Modules/CloudServicesSettingsEditor/) | UnityEditor | 6 | `UnityEditor.Analytics`、`UnityEditor.CrashReporting` | 目录级 |
| [UnityConnectEditor](../Modules/UnityConnectEditor/) | UnityEditor | 42 | `UnityEditor.Connect`、`UnityEditor.Connect.Fallback` | 目录级 |
| [UnityEditorAnalyticsEditor](../Modules/UnityEditorAnalyticsEditor/) | UnityEditor | 9 | `UnityEditor`、`UnityEditor.EditorAnalyticsDebugger` | 目录级 |
| [AcceleratorClient](../Modules/AcceleratorClient/) | UnityEditor | 3 | `Unity.AcceleratorClient` | 目录级 |
| [GenericRemoteEditor](../Modules/GenericRemoteEditor/) | UnityEditor | 1 | `UnityEditor.Remote` | 目录级 |
| [DeviceSimulatorEditor](../Modules/DeviceSimulatorEditor/) | UnityEditor | 24 | `UnityEditor.DeviceSimulation` | 目录级 |
| [MultiplayerEditor](../Modules/MultiplayerEditor/) | UnityEditor | 312 | `Unity.Multiplayer.PlayMode.Editor`、`Unity.Multiplayer.Center.Editor` | 目录级 |
| [UIAutomationEditor](../Modules/UIAutomationEditor/) | UnityEditor | 7 | `UnityEditor.UIAutomation` | 目录级 |

## 编辑器：UI 框架、图形编辑器框架与本地化

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [UIElementsEditor](../Modules/UIElementsEditor/) | UnityEditor | 151 | `UnityEditor.UIElements`、`UnityEditor.UIElements.Debugger` | [UI Toolkit](ui-toolkit/index.md)、[窗口系统](editor/window-system.md) |
| [UIBuilder](../Modules/UIBuilder/) | UnityEditor | 195 | `Unity.UI.Builder`、`UnityEditor.UIElements` | [UI Toolkit](ui-toolkit/index.md)（仅提及） |
| [UIToolkitAuthoringEditor](../Modules/UIToolkitAuthoringEditor/) | UnityEditor | 406 | `Unity.UIToolkit.Editor`、`UnityEditor.UIElements` | [UI Toolkit](ui-toolkit/index.md)（仅提及） |
| [UIElementsSamplesEditor](../Modules/UIElementsSamplesEditor/) | UnityEditor | 59 | `UnityEditor.UIElements.Samples` | 目录级 |
| [StyleSheetsEditor](../Modules/StyleSheetsEditor/) | UnityEditor | 13 | `UnityEditor.StyleSheets` | [UI Toolkit](ui-toolkit/index.md)（仅提及） |
| [GraphViewEditor](../Modules/GraphViewEditor/) | UnityEditor | 74 | `UnityEditor.Experimental.GraphView`、`UnityEditor.Experimental.UIElements.GraphView` | 目录级 |
| [GraphToolkitEditor](../Modules/GraphToolkitEditor/) | UnityEditor | 842 | `Unity.GraphToolkit.Editor`、`Unity.GraphToolkit.Editor.Implementation` | 目录级 |
| [PropertiesEditor](../Modules/PropertiesEditor/) | UnityEditor | 2 | `Unity.Properties.Internal` | 目录级 |
| [TextCoreFontEngineEditor](../Modules/TextCoreFontEngineEditor/) | UnityEditor | 2 | `UnityEditor.TextCore.LowLevel` | 目录级 |
| [TextCoreTextEngineEditor](../Modules/TextCoreTextEngineEditor/) | UnityEditor | 45 | `UnityEditor.TextCore.Text`、`UnityEngine.TextCore.Text` | 目录级 |
| [TextRenderingEditor](../Modules/TextRenderingEditor/) | UnityEditor | 4 | `UnityEditor` | 目录级 |
| [LocalizationEditor](../Modules/LocalizationEditor/) | UnityEditor | 6 | `UnityEditor` | 目录级 |
| [LocalizationRuntimeEditor](../Modules/LocalizationRuntimeEditor/) | UnityEditor | 70 | `Unity.Localization.Editor`、`Unity.Localization.Editor.Search` | 目录级 |
| [SmartStringsEditor](../Modules/SmartStringsEditor/) | UnityEditor | 7 | `Unity.SmartStrings.Editor` | 目录级 |
| [AccessibilityEditor](../Modules/AccessibilityEditor/) | UnityEditor | 8 | `UnityEditor.Accessibility` | 目录级 |

## 编辑器：分析与性能工具

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [ProfilerEditor](../Modules/ProfilerEditor/) | UnityEditor | 233 | `Unity.Profiling.Editor.UI`、`UnityEditorInternal.Profiling` | 目录级 |
| [JobsProfilerEditor](../Modules/JobsProfilerEditor/) | UnityEditor | 18 | `UnityEditor.JobsProfiling` | 目录级 |
| [NetworkProfilerEditor](../Modules/NetworkProfilerEditor/) | UnityEditor | 9 | `UnityEditor.Networking` | 目录级 |
| [InsightsEditor](../Modules/InsightsEditor/) | UnityEditor | 17 | `UnityEditor.InsightsEditor`、`UnityEditorInternal` | 目录级 |
| [ProjectAuditorEditor](../Modules/ProjectAuditorEditor/) | UnityEditor | 170 | `Unity.ProjectAuditor.Editor.Core`、`Unity.ProjectAuditor.Editor` | 目录级 |
| [BurstEditor](../Modules/BurstEditor/) | UnityEditor | 22 | `Unity.Burst.Editor` | [脚本编译](editor/script-compilation.md)（仅提及） |
| [MathematicsEditor](../Modules/MathematicsEditor/) | UnityEditor | 4 | `Unity.Mathematics.Editor` | 目录级 |

## 编辑器：内容领域编辑器

| 模块 | 程序集 | `.cs` 数 | 主要命名空间 | Wiki 覆盖 |
| --- | --- | --- | --- | --- |
| [AnimationWindow](../Modules/AnimationWindow/) | UnityEditor | 61 | `UnityEditorInternal`、`UnityEditor` | 目录级 |
| [AudioEditor](../Modules/AudioEditor/) | UnityEditor | 5 | `UnityEditor.Audio` | 目录级 |
| [ParticleSystemEditor](../Modules/ParticleSystemEditor/) | UnityEditor | 38 | `UnityEditor` | 目录级 |
| [PhysicsEditor](../Modules/PhysicsEditor/) | UnityEditor | 34 | `UnityEditor`、`UnityEditor.IMGUI.Controls` | 目录级 |
| [Physics2DEditor](../Modules/Physics2DEditor/) | UnityEditor | 41 | `UnityEditor` | 目录级 |
| [PhysicsCore2DEditor](../Modules/PhysicsCore2DEditor/) | UnityEditor | 28 | `Unity.U2D.Physics.Editor`、`UnityEditor.U2D.PhysicsCore2D.Profiler` | 目录级 |
| [ClothEditor](../Modules/ClothEditor/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [TerrainEditor](../Modules/TerrainEditor/) | UnityEditor | 30 | `UnityEditor.TerrainTools`、`UnityEditor` | 目录级 |
| [TreeEditor](../Modules/TreeEditor/) | UnityEditor | 18 | `TreeEditor` | 目录级 |
| [TilemapEditor](../Modules/TilemapEditor/) | UnityEditor | 8 | `UnityEditor` | 目录级 |
| [GridEditor](../Modules/GridEditor/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [SpriteMaskEditor](../Modules/SpriteMaskEditor/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [SpriteShapeEditor](../Modules/SpriteShapeEditor/) | UnityEditor | 1 | `UnityEditor.U2D` | 目录级 |
| [U2DEditor](../Modules/U2DEditor/) | UnityEditor | 16 | `UnityEditor.U2D.Profiling`、`UnityEditor.U2D` | 目录级 |
| [VectorGraphicsEditor](../Modules/VectorGraphicsEditor/) | UnityEditor | 6 | `Unity.VectorGraphics.Editor` | 目录级 |
| [VFXEditor](../Modules/VFXEditor/) | UnityEditor | 12 | `UnityEditor.VFX` | 目录级 |
| [VideoEditor](../Modules/VideoEditor/) | UnityEditor | 7 | `UnityEditor`、`UnityEditor.Video` | 目录级 |
| [MediaEditor](../Modules/MediaEditor/) | UnityEditor | 3 | `UnityEditor.Media`、`UnityEditorInternal.Media` | 目录级 |
| [Lighting](../Modules/Lighting/) | UnityEditor | 18 | `UnityEditor.Lighting.LightingSearch`、`UnityEditor.Lighting` | 目录级 |
| [EmbreeEditor](../Modules/EmbreeEditor/) | UnityEditor | 1 | `UnityEditor.Embree` | 目录级 |
| [MeshLODGenerator](../Modules/MeshLODGenerator/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [SketchUpEditor](../Modules/SketchUpEditor/) | UnityEditor | 4 | `UnityEditor` | 目录级 |
| [ShaderApiReflectionEditor](../Modules/ShaderApiReflectionEditor/) | UnityEditor | 9 | `UnityEditor.ShaderApiReflection` | 目录级 |
| [ShaderBuildSettingsEditor](../Modules/ShaderBuildSettingsEditor/) | UnityEditor | 1 | — | 目录级 |
| [ShaderLabParserEditor](../Modules/ShaderLabParserEditor/) | UnityEditor | 10 | `UnityEditor.ShaderApiReflection`、`UnityEditor` | 目录级 |
| [VirtualTexturingEditor](../Modules/VirtualTexturingEditor/) | UnityEditor | 1 | `UnityEngine.Rendering` | 目录级 |
| [XREditor](../Modules/XREditor/) | UnityEditor | 3 | `UnityEditor.XR` | 目录级 |
| [AIEditor](../Modules/AIEditor/) | UnityEditor | 7 | `UnityEditor.AI`、`UnityEditor` | 目录级 |
| [JSONSerializeEditor](../Modules/JSONSerializeEditor/) | UnityEditor | 1 | `UnityEditor` | 目录级 |
| [AdaptivePerformanceEditor](../Modules/AdaptivePerformanceEditor/) | UnityEditor | 40 | `UnityEditor.AdaptivePerformance.Editor`、`UnityEditor.AdaptivePerformance.Simulator.Editor` | 目录级 |
| [TimelineFoundationEditor](../Modules/TimelineFoundationEditor/) | UnityEditor | 131 | `Unity.Timeline.Foundation.View`、`Unity.Timeline.Foundation.Widgets` | 目录级 |
