# 播放器构建管线（C# 侧）

构建播放器时，C# 侧负责公共 API 参数校验、构建配置（Build Profile）转换，以及在各个阶段发现并调用用户构建回调；资源打包、脚本处理和平台后处理的主流程在原生 `BuildPlayer` 中。本页说明入口校验、回调发现规则和回调调用顺序。

父页面：[编辑器架构](index.md) · 相关：[脚本编译](script-compilation.md)、[资源导入](asset-pipeline.md)

## 相关源码文件

- [`Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs)
- [`Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs)
- [`Editor/Mono/BuildPipeline/BuildPlayerContext.cs`](../../Editor/Mono/BuildPipeline/BuildPlayerContext.cs)
- [`Editor/Mono/BuildPlayerWindowBuildMethods.cs`](../../Editor/Mono/BuildPlayerWindowBuildMethods.cs)
- 构建配置：[`Modules/BuildProfileEditor`](../../Modules/BuildProfileEditor/)、[`Editor/Mono/BuildProfile`](../../Editor/Mono/BuildProfile/)

## 1. 入口与校验

| 入口 | 行为 | 证据 |
| --- | --- | --- |
| `BuildPlayer(BuildPlayerWithProfileOptions)` | 拒绝空配置；**把传入配置设为当前活动配置**；从活动配置生成 `BuildPlayerOptions`，再走下面的入口 | [`BuildPipeline.bindings.cs:132-142`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L132-L142) |
| `BuildPlayer(BuildPlayerOptions)` | 完整校验后调用原生 `BuildPlayerInternal` | [`BuildPipeline.bindings.cs:199-297`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L199-L297) |
| 旧签名 `BuildPlayer(scenes, path, target, options)` | 组装 `BuildPlayerOptions` 后转发；注释要求不要再增加重载 | [`BuildPipeline.bindings.cs:144-175`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L144-L175) |
| `BuildPlayerFromUI` | UI 使用的入口，直接调用 `BuildPlayerInternal`；注释承认它与脚本入口的错误处理并不一致 | [`BuildPipeline.bindings.cs:299-312`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L299-L312) |

`BuildPlayer(BuildPlayerOptions)` 依次执行以下检查 [`BuildPipeline.bindings.cs:199-297`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L199-L297)：

1. 已经有构建在进行时，抛出 `InvalidOperationException`。
2. 目标组未知时，从目标平台推导。
3. 校验输出路径和场景路径，不合法时抛出 `ArgumentException`。
4. 使用 `AcceptExternalModificationsToPlayer` 时，检查目标平台是否支持追加构建。
5. 场景路径中的 `\` 统一为 `/`，并压缩重复斜杠。
6. 非 Development 构建不能使用 `AllowDebugging`、`EnableCodeCoverage`、`EnableDeepProfilingSupport` 或 `ConnectWithProfiler`；Development 构建使用覆盖率时，目标平台必须支持。
7. 校验构建路径；Standalone 平台还会解析并回写子目标。
8. 调用原生 `BuildPlayerInternal(...)` [`BuildPipeline.bindings.cs:381-382`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L381-L382)。

错误处理并不统一：前 6 步的 `ArgumentException` 和 `InvalidOperationException` 直接抛给调用方；第 7、8 步被 `try` 包住，其中 `ArgumentException` 只记录日志并返回 `null`，其它异常调用 `LogBuildExceptionAndExit`，注释称这是为了“防止数据丢失而退出” [`BuildPipeline.bindings.cs:261-296`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L261-L296)。

原生构建开始后，会回调 `PreparePlayerBuild`：它从原生指针恢复 `BuildPlayerOptions`，创建 `BuildPlayerContext`，并调用所有 `BuildPlayerProcessor.PrepareForBuild` [`BuildPipeline.bindings.cs:107-114`](../../Editor/Mono/BuildPipeline/BuildPipeline.bindings.cs#L107-L114)、[`BuildPipelineInterfaces.cs:1134-1138`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1134-L1138)。

## 2. 构建回调的发现

所有回调接口都继承 `IOrderedCallback`，通过 `callbackOrder` 排序，值越小越早执行 [`BuildPipelineInterfaces.cs:29-37`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L29-L37)。接口包括 `IPreprocessBuild`、`IPreprocessBuildWithReport`、`IPreprocessBuildWithContext`、`IFilterBuildAssemblies`、`IPostprocessBuild*`、`IProcessScene*`、`IPreprocessShaders`、`IPreprocessComputeShaders` 等 [`BuildPipelineInterfaces.cs:73-614`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L73-L614)。

`InitializeBuildCallbacks(findFlags)` 由原生调用 [`BuildPipelineInterfaces.cs:878-1005`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L878-L1005)：

- **缓存**：如果请求的标志和当前活动构建目标都与上次相同，就直接返回。目标比较是为修复 UUM-109242 加入的，因为某些回调会在构造函数中缓存与目标相关的数据。
- **发现**：遍历 `TypeCache.GetTypesDerivedFrom<IOrderedCallback>()`，跳过抽象类型和接口；一个类型即使实现了多个接口，也只创建**一个实例**，并加入多个列表 [`BuildPipelineInterfaces.cs:833-841`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L833-L841)。
- **旧式特性**：`[PostProcessBuild]` 和 `[PostProcessScene]` 方法会被包装成 `AttributeCallbackWrapper`，加入带 Report 的列表 [`BuildPipelineInterfaces.cs:965-977`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L965-L977)。
- **排序**：每个列表按 `callbackOrder` 排序 [`BuildPipelineInterfaces.cs:818-821`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L818-L821)、[`BuildPipelineInterfaces.cs:979-1005`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L979-L1005)。

构建后会发生域重载，所以启动后回调 `IPostprocessLaunch` 在使用时重新发现，而不依赖 `InitializeBuildCallbacks` 的缓存 [`BuildPipelineInterfaces.cs:1326-1345`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1326-L1345)。

## 3. 回调的调用

以预处理为例，原生调用 `OnBuildPreProcess(report)`，后者把旧接口 `IPreprocessBuild` 和 `IPreprocessBuildWithReport` 两个列表交给 `InvokeCallbackInterfacesPair` [`BuildPipelineInterfaces.cs:1140-1161`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1140-L1161)：

- **合并排序**：两个列表同时遍历，每次调用 `callbackOrder` 更小的回调，从而保持全局顺序。`callbackOrder` 相等时，**第二个列表**（这里是 WithReport）先执行，因为比较条件是严格小于 [`BuildPipelineInterfaces.cs:1085-1111`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1085-L1111)。
- **异常**：反射调用产生的 `TargetInvocationException` 会被拆开，记录内部异常；其它异常直接记录。`exitOnFailure` 为真时停止调用剩余回调 [`BuildPipelineInterfaces.cs:1113-1128`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1113-L1128)。预处理阶段只有构建选项或 AssetBundle 选项包含 `StrictMode` 时才会这样做 [`BuildPipelineInterfaces.cs:1155`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1155)。
- `OnBuildPreProcess` 没有使用 `InvokeCallbackInterfacesPair` 的返回值；严格模式下构建是否失败由原生判断，本仓库无法确认。

着色器预处理器会把回调类型集合计算为版本哈希 [`BuildPipelineInterfaces.cs:1240-1250`](../../Editor/Mono/BuildPipeline/BuildPipelineInterfaces.cs#L1240-L1250)。推断：这与 [资源导入 §1.3](asset-pipeline.md#13-版本与重新导入) 的后处理器哈希类似，用于在回调集合变化时让缓存失效。

## 4. 局限

- 原生 `BuildPlayer` 的阶段划分（何时调用 `PrepareForBuild`、`OnBuildPreProcess`、场景处理和后处理）没有 C# 证据，本页只列出各回调入口。
- `BuildTargetDiscovery.bindings.cs`（约 108 KB）、IL2CPP 工具（`Il2Cpp/IL2CPPUtils.cs`）、程序集裁剪（`AssemblyStripper.cs`）、构建配置模块（`BuildProfileEditor`，57 个文件）和构建报告模块（`BuildReportingEditor`）没有分析。
