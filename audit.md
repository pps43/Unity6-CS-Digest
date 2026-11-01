# 覆盖与一致性审计

本页记录本 Wiki 的覆盖结论、关键结论台账、已发现并修正的问题，以及验证状态。它的作用是让读者判断哪些内容可信、哪些只是推断、哪些区域没有分析。它**不**表示仓库已被完整分析。

返回：[首页](index.md)

## 1. 快照与范围

| 项 | 内容 |
| --- | --- |
| 源码快照 | `6974f873c0dd24dd88840492397e3a7ce3b817ab` |
| 审计日期 | 2026-10-31 |
| 范围 | 见 [首页“快照与范围”](index.md#快照与范围)；没有用户指定的特定区域，范围由本 Wiki 按“核心运行路径 + 编辑器核心 + 全量目录盘点”确定 |
| 页面数 | 25 个 Markdown 页面（含本页） |
| 审计方式 | 在撰写所用的同一上下文中执行；**没有**使用独立代理复核 |

## 2. 执行过的命令与机械检查

| 目的 | 命令（概要） | 结果 |
| --- | --- | --- |
| 快照 | `git log -1` | 提交如上；工作区原本没有未提交修改 |
| 文件类型统计 | `Get-ChildItem -Recurse -File \| Group-Object Extension` | 8692 个 `.cs`、12 个 `.dll`、5 个 `.csproj`、2 个 `.md`、1 个 `.sln`、1 个 `.txt`（统计时 Wiki 尚未创建） |
| 工程归属 | 解析两个主 `.csproj` 的 `Compile Include`，与磁盘 `.cs` 文件做集合比较 | 8689/8692 被主工程引用；其余 3 个是 IncrementalBuildPipeline 的 `Data.cs`；同时属于两个工程的只有 2 个文件；模块归属 78 个 Engine、88 个 Editor，无交集 |
| 预处理指令 | `rg "^\s*#\s*(if\|elif\|else\|endif\|define\|undef)\b" --glob *.cs` | `#if` 为 0 条；另有 7 条 `#endif`/`#define`，都位于注释块中（`AdaptivePerformanceAnalytics.cs`、`TreeEditor/Includes/Perlin.cs`） |
| 测试 | `rg -l "\[Test\]\|\[UnityTest\]"`、查找名为 `Test`/`Tests` 的目录 | 0 个测试用例或测试目录；只有 `TestRunnerHelpers.cs` 引用 NUnit 名称 |
| 绑定规模 | 按文件名、`[NativeHeader(`、`RequiredByNativeCode`、`class BindingsMarshaller` 统计 | 869 个 `*.bindings.cs`、27 个 `*.binding.cs`；902 个文件含 `[NativeHeader(`；555 个文件含 `RequiredByNativeCode`；144 处 `BindingsMarshaller`；1 个文件含 `MethodImplOptions.InternalCall` |
| SDK | `dotnet --list-sdks` | 6.0.321、9.0.316 |
| 缺失依赖 | `Test-Path` 检查 BannedSymbols 文件和 `artifacts/MonolithicEditorDlls/Debug` | 都不存在 |
| 模块统计 | 遍历每个模块，统计 `.cs` 数、命名空间和工程归属 | 166 个模块，结果见 [模块目录](module-catalog.md) |
| 链接与行号 | 自写脚本：提取所有 Markdown 链接，检查目标文件存在；检查 `#Lx-Ly` 中 1 ≤ x ≤ y ≤ 文件行数；检查 `.md` 锚点与标题 slug 是否一致 | 最后一次运行见 §8 |

**未执行**：没有构建、运行或测试代码，也没有运行 IDE 分析。原因见 [物理组织 §7](physical-layout.md#7-构建状态与缺失依赖)。

## 3. 正向覆盖：计划页面 → 证据

| 计划页面 | 目的 | 状态 | 说明 |
| --- | --- | --- | --- |
| [overview](overview.md) | 仓库形态与能力地图 | 已覆盖 | 高层架构图中的每条边都附有证据 |
| [getting-started](getting-started.md) | 阅读和追踪方法 | 已覆盖 | 命令来自研究时实际使用的检索；构建步骤未执行，已明确说明 |
| [physical-layout](physical-layout.md) | 工程、程序集、模块化 | 已覆盖 | 基于 `.csproj` 和文件统计 |
| [module-catalog](module-catalog.md) | 全量模块盘点 | 已覆盖（目录级） | 分类由本 Wiki 整理 |
| [runtime/bindings](runtime/bindings.md) | 托管/原生互操作 | 已覆盖 | “生成代码”相关描述为推断，已标注 |
| [runtime/object-model](runtime/object-model.md) | `Object` 身份与空值语义 | 已覆盖 | 销毁时序属于原生行为，未覆盖 |
| [runtime/lifecycle-scopes](runtime/lifecycle-scopes.md) | 作用域状态机与静态清理 | 已覆盖 | 没有逐行分析 `ActiveLifecycleScopesTracker` |
| [runtime/player-loop-and-async](runtime/player-loop-and-async.md) | 帧循环表示、协程、同步上下文、Awaitable | 已覆盖 | 默认执行顺序属于原生行为 |
| [runtime/serialization](runtime/serialization.md) | 序列化特性与 V2 执行器 | 部分 | 没有逐个分析操作码；v1/v2 的选择条件未知 |
| [runtime/jobs-and-collections](runtime/jobs-and-collections.md) | `IJob`、`NativeArray` | 部分 | 只分析了两条代表性路径 |
| [editor/editor-application](editor/editor-application.md) | 编辑器回调与初始化 | 已覆盖 | 与生命周期回调的先后顺序未确认 |
| [editor/window-system](editor/window-system.md) | 视图层级与窗口后端 | 已覆盖 | 布局保存和恢复未追踪 |
| [editor/imgui](editor/imgui.md) | IMGUI 周期与两遍布局 | 已覆盖 | 没有分析具体控件 |
| [editor/inspector](editor/inspector.md) | 编辑器类型解析、Inspector 构建、属性绘制器 | 已覆盖 | 没有分析 `ActiveEditorTracker`（原生）和 `PropertyHandler` |
| [editor/script-compilation](editor/script-compilation.md) | 程序集模型与 Bee 编译 | 已覆盖 | MSBuild 路径只提及 |
| [editor/asset-pipeline](editor/asset-pipeline.md) | 后处理器和脚本化导入器 | 已覆盖 | 版本哈希的用途为推断 |
| [editor/build-pipeline](editor/build-pipeline.md) | 构建入口与回调 | 已覆盖 | 原生阶段顺序未知 |
| [editor/package-manager](editor/package-manager.md) | UPM 请求与 UI 服务容器 | 部分 | UI 只分析了两个机制 |
| [ui-toolkit/index](ui-toolkit/index.md) | 面板与更新阶段、驱动方 | 已覆盖 | PlayerLoop 阶段与原生入口的对应关系为推断 |
| [ui-toolkit/events](ui-toolkit/events.md) | 分发器与传播 | 已覆盖 | 没有逐个分析各事件类型的 `PreDispatch`/`PostDispatch` |
| [ui-toolkit/styles-layout-rendering](ui-toolkit/styles-layout-rendering.md) | 导入、样式、布局、渲染 | 部分 | UIR 内部和样式属性生成代码未分析 |

## 4. 反向覆盖：从仓库到页面

盘点单位是“需要理解的职责或运行入口”，不是文件。

| 对象 | 类型 | 发现来源 | 重要性 | 对应页面 | 状态 | 说明 |
| --- | --- | --- | --- | --- | --- | --- |
| `UnityReferenceSource.sln` 与 5 个工程 | 工程 | `.sln` | 高 | physical-layout | 已覆盖 | — |
| `Runtime/Export/Scripting` | 核心 API | 目录 | 高 | object-model、player-loop-and-async、bindings | 已覆盖 | 未覆盖文件包括 `GameObject.deprecated.cs`、`TagHandle.cs` 等小文件 |
| `Runtime/Export/PlayerLoop` | 帧循环 | 目录 | 高 | player-loop-and-async | 已覆盖 | — |
| `Runtime/Export/Serialization`、`Runtime/Serialize` | 序列化 | 目录 | 高 | serialization | 部分 | 见 §3 |
| `Runtime/Export/Application` | 应用 API | 目录 | 中 | lifecycle-scopes | 部分 | 只分析了作用域入口 |
| `Runtime/Export` 其余约 65 个子目录（Graphics 66、RenderPipeline 44、Director 31、Math 16、Camera 11 等） | 领域 API | 目录 | 中 | — | 有意排除 | 多为绑定声明，机制见 bindings |
| `Runtime/Scripting` | 生命周期集成 | 目录 | 高 | lifecycle-scopes | 已覆盖 | `APIUpdaterRuntimeHelpers.cs` 未分析 |
| `Runtime/Profiler`、`Transform`、`2D`、`GI`、`Input`、`Jobs`、`Misc`、`NameFormatter`、`VirtualFileSystem` | 运行时子系统 | 目录 | 中/低 | — | 有意排除 | `Runtime/Jobs` 只在 jobs-and-collections 中提及 |
| `External/ScriptingCore` | 生命周期基础设施 | 目录 | 高 | lifecycle-scopes | 已覆盖 | — |
| `External` 其余 15 个子目录 | 第三方或共享代码 | 目录 | 低 | physical-layout | 不适用 | 只说明用途，不分析第三方实现 |
| `artifacts/` | 生成代码与工具 DLL | 目录 | 中 | physical-layout | 已覆盖 | — |
| `Tools/` | 构建工具 | 目录 | 低 | physical-layout | 已覆盖 | — |
| `Editor/IncrementalBuildPipeline` | 数据契约 | `.sln` | 中 | physical-layout、script-compilation | 已覆盖 | `PlayerBuildProgramLibrary.Data` 只列出类型 |
| `Editor/Mono` 根目录核心类型（`EditorApplication`、`EditorWindow`、`HostView`、`GUIView`、`ContainerWindow`、`View`、`AssetPostprocessor`、`TypeCache`、`SerializedObject/Property`、`CustomEditorAttributes`、`EditorAssemblies`、`AssemblyReloadEvents`） | 编辑器核心 | 目录 | 高 | editor/* | 已覆盖 | 根目录共 260 个文件，其余例如 `Undo.cs`、`EditorGUI.cs`、`SceneHierarchy*.cs`、`PlayerSettings*.cs`、`ConsoleWindow.cs` 未分析 |
| `Editor/Mono/Inspector` | Inspector | 目录 | 高 | inspector | 部分 | 200 多个内置编辑器未分析 |
| `Editor/Mono/GUI` | 停靠和分割视图、编辑器控件 | 目录 | 中 | window-system | 部分 | 只覆盖类型层级 |
| `Editor/Mono/Scripting` | 编译和 API 更新 | 目录 | 高 | script-compilation | 部分 | APIUpdater 和 RestrictedApis 未分析 |
| `Editor/Mono/BuildPipeline` | 构建 | 目录 | 高 | build-pipeline | 部分 | 见页面局限 |
| `Editor/Mono/AssetPipeline`、`AssetDatabase` | 导入器绑定 | 目录 | 中 | asset-pipeline | 部分 | 各内置导入器未分析 |
| `Editor/Mono/UIElements` | 编辑器 UI Toolkit 集成 | 目录 | 中 | inspector、styles-layout-rendering | 部分 | — |
| `Editor/Mono` 其余子目录（SceneView、Overlays、Animation、Audio、BuildProfile、VersionControl、Handles、Prefabs、SceneManagement、Search、Tools、GI、ProjectBrowser 等，共 78 个子目录中的大部分） | 编辑器功能 | 目录 | 中 | — | 有意排除 | 范围限制 |
| `Modules/*`（166 个） | 模块 | 目录 | 混合 | module-catalog | 7 个已覆盖、2 个部分、10 个仅提及、147 个有意排除（目录级） | 见下 |
| `[RequiredByNativeCode]` 入口（555 个文件） | 运行入口 | `rg` | 高 | 各页面 | 部分 | 已覆盖的入口族：对象绑定、异常封送、协程、同步上下文、Awaitable、生命周期作用域、PlayerLoop 类型、序列化 V2、IMGUI Begin/End、GUIView 事件、EditorApplication 回调、InitializeOnLoad、AssemblyReloadEvents、编译接口、资源后处理、ScriptedImporter、自定义编辑器解析、构建回调、UI Toolkit 驱动入口 |
| CI/CD、打包、发布脚本 | 交付 | 仓库根目录 | — | — | 不适用 | 仓库中没有这些内容 |
| 测试 | 契约 | 搜索 | 高 | — | 证据不足 | 仓库不含测试，所有契约只能来自实现和注释 |

**模块的有意排除理由**：147 个目录级模块（例如 Physics、Animation、Audio、Terrain、ProfilerEditor、QuickSearch、ShortcutManagerEditor、GraphToolkitEditor、MultiplayerEditor、ProjectAuditorEditor、TimelineFoundation、LocalizationRuntime）都是领域功能。它们的跨边界机制已由 bindings、lifecycle-scopes 和 editor/index 中的扩展点模式覆盖；分析每个模块的领域逻辑超出了本次范围。

**状态计数**（按上表行数，不代表语义覆盖率）：已覆盖 9 行，部分 9 行，有意排除 3 行，不适用 2 行，证据不足 1 行，“遗漏”0 行；另有 “Modules/*” 1 行单独按模块计数（7 个已覆盖、2 个部分、10 个仅提及、147 个目录级）。

## 5. 关键结论台账

| ID | 规范化结论 | 条件或范围 | 证据 | 页面位置 | 状态 |
| --- | --- | --- | --- | --- | --- |
| C1 | 仓库只用于参考，不允许修改或再分发，不接受 PR | — | `README.md:12-17` | overview、index | 已验证 |
| C2 | 源码可能已按编辑器配置展开条件编译 | 全仓库 | `#if` 为 0 条，加上 `UnityEngineObject.bindings.cs:341-359` 的注释与代码不一致 | overview、physical-layout、object-model、serialization、getting-started | 推断 |
| C3 | 真实构建按模块拆分程序集，参考工程把它们合并为两个 | — | `InternalsVisibleTo.cs:5-25`、`AssemblyInfo.cs:5-10`、`UnityEngine.csproj:13` | overview、physical-layout、getting-started | 已验证（真实构建形态来自间接证据） |
| C4 | `extern` 方法体在生产构建中由 IL 注入器改写 | 生产构建 | `SerializationBackendManagedCommands.bindings.cs:111-118` 注释 | bindings | 文档声明（注释），未见实现 |
| C5 | `UnityEngine.Object` 的 `==`/`bool` 判断原生对象是否存活；MonoBehaviour/ScriptableObject 在指针为空时直接判为死亡 | — | `UnityEngineObject.bindings.cs:318-360` | object-model、runtime/index、getting-started | 已验证 |
| C6 | 作用域嵌套为 AssemblyLoaded ⊃ CodeLoaded ⊃ CodeInitialized ⊃ {PlayMode, EditMode} | — | 各 Scope 构造函数 | lifecycle-scopes | 已验证 |
| C7 | 作用域转换只能在主线程发起，否则报错并返回 | — | `LifecycleController.cs:214-237` | lifecycle-scopes、runtime/index | 已验证 |
| C8 | 生命周期回调中的异常被逐个捕获，不中断其它回调 | 作用域转换 | `ScopeTransitionHelper.cs:70-103` | lifecycle-scopes | 已验证 |
| C9 | `PlayerLoopSystemInternal.numSubSystems` 是后代总数 | `SetPlayerLoop` 拍平时 | `PlayerLoop.bindings.cs:400-421` | player-loop-and-async | 已验证 |
| C10 | `UnitySynchronizationContext.Exec` 只执行调用开始时已入队的工作 | — | `UnitySynchronizationContext.cs:74-89` | player-loop-and-async | 已验证 |
| C11 | `Awaitable` 被池化，只能 await 一次；结果被取走后归还池中 | — | `Awaitable.cs:17-37`、`217-254` | player-loop-and-async、runtime/index | 已验证（“只能 await 一次”是设计约定） |
| C12 | `delayCall` 是一次性队列，回调中新加入的项进入下一轮 | — | `EditorApplication.cs:519-526` | editor-application、editor/index | 已验证 |
| C13 | `update`/`delayCall` 分发在 C# 侧没有逐回调的异常捕获 | — | `EditorApplication.cs:503-526` | editor-application | 已验证（后果为推断） |
| C14 | `[InitializeOnLoad]` 由原生排序后运行静态构造函数；类型加载和初始化异常会被记录，然后继续 | — | `EditorAssemblies.cs:121-152` | editor-application、editor/index | 已验证 |
| C15 | `EditorWindow.Show()` 创建 ContainerWindow → SplitView → DockArea → 标签页 | 窗口没有父视图时 | `EditorWindow.cs:1459-1475` | window-system | 已验证 |
| C16 | 默认窗口后端由 UIElementsEditor 在 `[OnCodeLoaded]` 时注入，每个 GUIView 的内容运行在面板根部的 `IMGUIContainer` 中 | — | `RetainedMode.cs:25-43`、`DefaultWindowBackend.cs:23-41` | window-system、imgui、ui-toolkit/index | 已验证 |
| C17 | 自定义编辑器解析先做常规匹配，再做 fallback 匹配，并沿基类链向上查找 | — | `CustomEditorAttributes.cs:57-85` | inspector、editor/index | 已验证 |
| C18 | Inspector 先尝试 `CreateInspectorGUI`，返回 null 时回退到 IMGUI | 自定义编辑器 | `InspectorElement.cs:733-734` | inspector | 已验证 |
| C19 | 有编译设置错误时不启动编译；编译通过 Bee 异步执行并由 tick 轮询 | Bee 路径 | `EditorCompilation.cs:879-972`、`1156-1265` | script-compilation | 已验证 |
| C20 | 编译接口的异常处理因入口而异 | — | `EditorCompilationInterface.cs:41-69`、`89-154` | script-compilation | 已修正（见 §6） |
| C21 | 多个 ScriptedImporter 自动选择同一扩展名时，全部被拒绝 | — | `ScriptedImporter.cs:103-119` | asset-pipeline | 已验证 |
| C22 | 构建回调按 `callbackOrder` 合并执行；顺序相同时第二个列表先执行；严格模式下第一个异常会停止后续回调 | `InvokeCallbackInterfacesPair` | `BuildPipelineInterfaces.cs:1078-1132` | build-pipeline | 已验证 |
| C23 | UI Toolkit 面板在编辑器中由 RetainedMode 驱动，在运行时由 `UIElementsRuntimeUtilityNative` 回调驱动 | — | `RetainedMode.cs:78-93`、`UIElementsUtility.bindings.cs:12-60` | ui-toolkit/index | 已验证；与 PlayerLoop 阶段的对应关系为推断 |
| C24 | 事件处理期间产生的新事件会入队，并在当前事件完成后、剩余队列之前处理 | — | `EventDispatcher.cs:337-420` | events | 已验证 |
| C25 | 布局最多正常循环 10 遍，之后开启诊断记录 | — | `VisualTreeLayoutUpdater.cs:24-30`、`346-360` | styles-layout-rendering | 已验证（超限后的进一步行为未追踪） |
| C26 | 布局计算由原生 Yoga 执行 | 默认处理器 | `LayoutProcessor.cs:32`、`LayoutNative.bindings.cs:13-17` | styles-layout-rendering、ui-toolkit/index | 已验证 |

## 6. 跨页与对抗审计发现

停止补充正文后，审计从源码重新检查了高风险结论，并搜索“所有、总是、从不、全部、不会”等容易过度概括的措辞。发现并处理如下：

| 严重度 | 发现 | 处理 |
| --- | --- | --- |
| 重要 | 脚本编译页原先写“异常不会传回原生”。复查发现 `SetAllScripts`、`RequestScriptCompilation` 等入口只是直接转发，没有捕获异常 | 已修正为“因入口而异”，并补充证据（C20） |
| 重要 | 生命周期页原先写“所有转换都由原生触发”，但 `[OnCodeInitializing]` 回调和 `EditModeScopePostprocessor` 也会从托管侧发起转换 | 已修正 |
| 重要 | 编辑器父页面原先写“已有构建时不能 BuildPlayer”，但这项检查只在 `BuildPlayer(BuildPlayerOptions)` 中，`BuildPlayerFromUI` 本身没有 | 已补充条件 |
| 一般 | 窗口系统页原先写“没有 `Update` 的窗口不会给 `update` 增加任何开销”，忽略了 `CheckForWindowRepaint` 订阅 | 已修正 |
| 一般 | 对象模型页开头把“可被序列化的对象”都归为 `UnityEngine.Object`，与 `[SerializeReference]` 的普通类相矛盾 | 已修正 |
| 一般 | UI Toolkit 页原先写“修改元素时不会立即重新计算任何东西” | 已改为“通常通过递增版本通知”，并补充 `elementPanel` 为空时的行为 |
| 一般 | Inspector 页把首次构建描述为“只创建 2 个元素”，遗漏了“填满视口”和“恢复滚动位置” | 已修正 |
| 一般 | 物理组织页把“92 个 AssemblyInfo.cs 文件”写成“92 个模块” | 已修正为 92 个文件、80 个模块 |
| 一般 | 编辑器父页面的扩展点表原先把所有入口都标为“原生调用”，但属性绘制器查询由托管代码发起 | 已修正表头和说明 |
| 一般 | `EditorApplication` 层级和项目事件的引用范围截断 | 已修正行号 |
| 一般 | 窗口系统事件时序图中的一步原先没有直接证据 | 已补充 `DefaultWindowBackend.ProcessEvent` 证据，其余步骤标为推断 |
| 一般 | 预定义程序集的深度相同时，选择结果依赖字典枚举顺序 | 已作为推断写入脚本编译页 |

同一结论在多个页面出现时（C5、C11、C12、C16、C17），已逐页核对措辞，没有发现仍未解决的矛盾。

## 7. 未决与证据不足

- **原生行为**：PlayerLoop 默认顺序、对象销毁时序、`InitializeOnLoad` 与生命周期回调的相对顺序、构建主流程阶段、UI Toolkit PlayerLoop 阶段与原生入口的对应关系，都只能推断。
- **生成工具**：绑定生成器、IL 注入器、AutoStaticsCleanup 源生成器和 UI Toolkit 源生成器都不在仓库中。C2、C4 以及静态清理在核心程序集中是否生效，都无法验证。
- **播放器行为**：由于 C2，播放器专用分支可能缺失，本 Wiki 描述的是编辑器配置下的代码。
- **大文件**：`ReadExecutor.cs`/`WriteExecutor.cs`（各约 46 KB）、`ActiveLifecycleScopesTracker.cs`（约 34 KB）、`EditorCompilation.cs`（约 99 KB）、`GUI.cs`/`GUILayout.cs`、`MsBuildCompilation.cs` 只分析了与本 Wiki 结论相关的片段。

## 8. 验证状态

- **链接与行号**：脚本检查了全部页面中的相对链接、`#Lx-Ly` 行号范围和 `.md` 锚点。最后一次检查了 1084 个链接，错误为 0。这只说明目标存在且行号在范围内，不说明引用内容在语义上正确。
- **语义抽查**：§5 中全部 26 条关键结论都在撰写时或审计时回到源码复核过。一般性描述只做了代表性抽查，没有逐条复核。
- **图示**：每张 Mermaid 图下方都列出了节点和边的证据；无法从 C# 证据确认的部分已在图下注明为推断。环境中没有 Mermaid 渲染器，图的语法**没有经过渲染验证**，只人工检查过泛型尖括号等容易出错的字符。
- **代码命令**：[阅读与导航指南](getting-started.md) 中的 `rg` 命令都是研究时使用过的检索形式。构建、运行和测试命令一律没有执行。
