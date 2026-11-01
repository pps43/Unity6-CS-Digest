# 资源导入管线（C# 侧）

资源数据库、导入调度、缓存和工件存储都在原生侧。C# 侧负责两类扩展：**后处理器**（`AssetPostprocessor`），用于在导入前后修改资源；以及**脚本化导入器**（`ScriptedImporter`），用于让 C# 类型接管某种文件扩展名的导入。本页说明它们如何被发现、排序和调用，以及版本如何影响资源重新导入。

父页面：[编辑器架构](index.md) · 相关：[编辑器应用与回调](editor-application.md)、[样式、布局与渲染](../ui-toolkit/styles-layout-rendering.md)（USS/UXML 导入器示例）

## 相关源码文件

- [`Editor/Mono/AssetPostprocessor.cs`](../../Editor/Mono/AssetPostprocessor.cs)
- [`Modules/AssetPipelineEditor/Public/ScriptedImporter.cs`](../../Modules/AssetPipelineEditor/Public/ScriptedImporter.cs)
- [`Editor/Mono/AssetPipeline/AssetImporter.bindings.cs`](../../Editor/Mono/AssetPipeline/AssetImporter.bindings.cs)、[`AssetImportContext.bindings.cs`](../../Editor/Mono/AssetPipeline/AssetImportContext.bindings.cs)
- [`Modules/AssetDatabase/Editor/ScriptBindings/AssetDatabase.bindings.cs`](../../Modules/AssetDatabase/Editor/ScriptBindings/AssetDatabase.bindings.cs)：`AssetDatabase` 公共 API（约 96 KB，几乎全是绑定）

## 1. `AssetPostprocessor`

### 1.1 两类回调

| 类型 | 何时调用 | 实例 | 证据 |
| --- | --- | --- | --- |
| 单资源回调（`OnPreprocessAsset`、`OnPostprocessTexture`、`OnPreprocessAudio` 等） | 某个资源导入期间 | 每次导入为每个匹配的后处理器类型新建实例，并设置 `assetPath` 和 `context` | [`AssetPostprocessor.cs:524-553`](../../Editor/Mono/AssetPostprocessor.cs#L524-L553) |
| `OnPostprocessAllAssets`（静态方法） | 一批自动导入完成后 | 不创建实例，直接调用静态方法 | [`AssetPostprocessor.cs:340-374`](../../Editor/Mono/AssetPostprocessor.cs#L340-L374) |

单资源回调通过方法名反射查找：`InvokeMethodIfAvailable` 在实例类型上查找公共或非公共实例方法，存在时就在性能标记中调用 [`AssetPostprocessor.cs:1133-1145`](../../Editor/Mono/AssetPostprocessor.cs#L1133-L1145)。例如 `PreprocessAsset` 对当前导入的所有处理器调用 `OnPreprocessAsset` [`AssetPostprocessor.cs:699-706`](../../Editor/Mono/AssetPostprocessor.cs#L699-L706)。

### 1.2 发现与排序

- 对给定导入器类型，先沿导入器类型链汇总“可能相关的方法名”，再在所有后处理器类型中筛选实现了其中任意方法的类型。没有无参构造函数的类型会被报错并跳过。结果按导入器类型缓存 [`AssetPostprocessor.cs:656-687`](../../Editor/Mono/AssetPostprocessor.cs#L656-L687)。
- 排序依次比较 `GetPostprocessOrder()`、类型全名、程序集限定名，保证结果确定 [`AssetPostprocessor.cs:376-399`](../../Editor/Mono/AssetPostprocessor.cs#L376-L399)。`GetPostprocessOrder` 在设置 `assetPath` 后只读取一次，所以实现可以按资源返回不同值，但同一次导入中不会变化 [`AssetPostprocessor.cs:71-73`](../../Editor/Mono/AssetPostprocessor.cs#L71-L73)。
- 支持重入，例如在导入中立即导入另一个资源：`InitPostprocessors` 把当前处理器集合压栈，`CleanupPostprocessors` 出栈 [`AssetPostprocessor.cs:549-561`](../../Editor/Mono/AssetPostprocessor.cs#L549-L561)。

### 1.3 版本与重新导入

后处理器分为“静态依赖”和“动态”两类。实现的方法中只要有一个不属于 `k_NonAutomaticDependencyMethods`，`PostprocessorInfo` 就被视为静态依赖，并在构造时记录 `GetVersion()` 和 `GetPostprocessOrder()` [`AssetPostprocessor.cs:46-59`](../../Editor/Mono/AssetPostprocessor.cs#L46-L59)。原生会为不同导入器调用 `Get*ProcessorsHashString`；这些方法把所有静态依赖后处理器的“程序集限定名 + 版本”拼接成字符串 [`AssetPostprocessor.cs:401-416`](../../Editor/Mono/AssetPostprocessor.cs#L401-L416)、[`AssetPostprocessor.cs:689-697`](../../Editor/Mono/AssetPostprocessor.cs#L689-L697)。

推断：这个字符串参与导入结果的依赖哈希，所以修改后处理器的 `GetVersion()` 或增删后处理器，会触发对应类型资源重新导入。原生侧如何使用它不在仓库中。

### 1.4 `OnPostprocessAllAssets`

- 只有声明了 `didDomainReload` 参数的版本，才会在“本批没有任何资源变化”时仍被调用；不带该参数的版本在空批次中被跳过 [`AssetPostprocessor.cs:344-364`](../../Editor/Mono/AssetPostprocessor.cs#L344-L364)。
- 每个处理器的异常都会被捕获并记录，不影响其它处理器 [`AssetPostprocessor.cs:366-369`](../../Editor/Mono/AssetPostprocessor.cs#L366-L369)。
- 最后调用 `CodeEditorProjectSync.PostprocessSyncProject`，同步 IDE 工程文件 [`AssetPostprocessor.cs:372-373`](../../Editor/Mono/AssetPostprocessor.cs#L372-L373)。

编辑器自身也通过这个机制实现功能，例如在域重载后重新进入 EditMode 作用域 [`EditModeScopePostprocessor.cs:12-25`](../../Editor/Mono/Scripting/EditModeScopePostprocessor.cs#L12-L25)，以及在 UXML/USS 变化后刷新 UI Toolkit [`RetainedMode.cs:95-123`](../../Modules/UIElementsEditor/RetainedMode.cs#L95-L123)。

## 2. `ScriptedImporter`

### 2.1 契约

用户继承 `ScriptedImporter` 并实现 `OnImportAsset(AssetImportContext ctx)`。原生通过 `[RequiredByNativeCode] GenerateAssetData` 调用它 [`ScriptedImporter.cs:23-33`](../../Modules/AssetPipelineEditor/Public/ScriptedImporter.cs#L23-L33)。`[ScriptedImporter]` 特性声明 `version`、`importQueuePriority`、`fileExtensions`、`overrideFileExtensions` 和 `AllowCaching` [`ScriptedImporter.cs:234-270`](../../Modules/AssetPipelineEditor/Public/ScriptedImporter.cs#L234-L270)。

### 2.2 注册

`RegisterScriptedImporters`（原生调用）[`ScriptedImporter.cs:60-138`](../../Modules/AssetPipelineEditor/Public/ScriptedImporter.cs#L60-L138)：

1. 用 `TypeCache.GetTypesWithAttribute<ScriptedImporterAttribute>()` 获取候选类型；不继承 `ScriptedImporter` 的类型报错。
2. 计算每个导入器处理的扩展名；没有任何有效扩展名时报错。
3. **冲突处理**：如果多个导入器把同一个扩展名声明为自动选择，所有相关导入器都会失去这个扩展名，并输出错误。规则不会选出“胜者”。
4. 检查导入器是否声明了 `GatherDependenciesFromSourceFile` 静态方法（会沿父类查找），决定是否支持导入依赖提示。
5. 对每个剩余扩展名调用 `AssetImporter.RegisterImporter(...)`，传入版本、优先级、扩展名、依赖提示、是否自动选择和是否允许缓存。

内置示例：USS 导入器 `[ScriptedImporter(version: 24, ext: "uss", importQueueOffset: 1100)]` [`StyleSheetImporter.cs:18-20`](../../Editor/Mono/UIElements/StyleSheets/StyleSheetImporter.cs#L18-L20)；UXML 导入器 `[ScriptedImporter(version: 36, ext: "uxml", importQueueOffset: 1102)]` [`UIElementsViewImporter.cs:29-31`](../../Modules/UIElementsEditor/UIElementsViewImporter.cs#L29-L31)。UXML 的队列偏移更大，从数值推断，它在 USS 之后导入。

## 3. 局限

- `AssetDatabase` 的刷新、导入工作进程（`AssetImportWorkerPostProcessorHelper.cs`）、工件差异报告（`ArtifactDifferenceReporter.cs`）和导入活动窗口（`ImportActivityWindow.cs`，约 151 KB）没有分析。
- 原生导入器（纹理、模型、音频）的 C# 侧主要是设置项绑定和 Inspector，例如 `Modules/AssetPipelineEditor/ImportSettings`，本页没有展开。
