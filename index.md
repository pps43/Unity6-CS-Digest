# UnityCsReference 代码库 Wiki

**UnityCsReference** 是 Unity 6000.7.0b7 引擎与编辑器 **C# 部分**的参考源码，仅供阅读，不能修改或再分发 [`README.md:1-17`](../README.md#L1-L17)。本 Wiki 说明这些 C# 代码如何组织、如何与仓库外的原生引擎交互，以及运行时和编辑器核心子系统的实现机制。

## 快照与范围

| 项 | 内容 |
| --- | --- |
| 源码版本 | 提交 `6974f873c0dd24dd88840492397e3a7ce3b817ab`（Unity 6000.7.0b7 C# reference source code） |
| 分析日期 | 2026-10-31 |
| 目标读者 | 想理解 Unity C# API 背后机制、托管/原生边界和编辑器子系统协作方式的引擎或工具开发者 |
| 分析范围 | 工程结构；运行时核心（绑定、对象模型、生命周期、帧循环与异步、序列化、Jobs）；编辑器核心（主循环回调、窗口、IMGUI、Inspector、脚本编译、资源导入、播放器构建、包管理器）；UI Toolkit（更新阶段、事件、样式、布局、渲染）；全部 166 个模块的目录级盘点 |
| 输出语言 | 中文；代码标识符保留原文 |

## 阅读前必须知道的三件事

1. **原生代码不在仓库中。** 大量 API 是 `extern` 声明，实际实现位于 C++；本 Wiki 对原生行为的描述都标为推断。见 [仓库概览 §3.1](overview.md#31-原生代码缺席)。
2. **源码可能已按编辑器配置预处理。** 全仓库没有任何 `#if` 指令，播放器专用分支可能已被剥离。见 [仓库概览 §3.2](overview.md#32-源码已按编辑器配置预处理推断)。
3. **模块在真实构建中是独立程序集。** 参考工程把它们合并成两个程序集，`internal` 的可见范围因此与真实构建不同。见 [物理组织 §3](physical-layout.md#3-模块化与单体程序集)。

## 页面树

- [仓库概览](overview.md)：仓库的用途、形态、高层架构和能力地图
  - [阅读与导航指南](getting-started.md)：如何浏览、检索和追踪跨边界调用；构建状态
  - [物理组织：工程、程序集与模块](physical-layout.md)：解决方案、`.csproj`、`artifacts/`、`External/`、数据契约工程
    - [模块目录](module-catalog.md)：166 个模块的程序集、规模、命名空间和覆盖状态
  - [运行时架构](runtime/index.md)
    - [托管/原生绑定层](runtime/bindings.md)
    - [对象模型：`UnityEngine.Object`](runtime/object-model.md)
    - [代码加载生命周期作用域](runtime/lifecycle-scopes.md)
    - [PlayerLoop、协程与异步](runtime/player-loop-and-async.md)
    - [序列化](runtime/serialization.md)
    - [Jobs 与原生容器](runtime/jobs-and-collections.md)
  - [编辑器架构](editor/index.md)
    - [编辑器应用与回调](editor/editor-application.md)
    - [窗口系统](editor/window-system.md)
    - [IMGUI](editor/imgui.md)
    - [Inspector](editor/inspector.md)
    - [脚本编译管线](editor/script-compilation.md)
    - [资源导入管线](editor/asset-pipeline.md)
    - [播放器构建管线](editor/build-pipeline.md)
    - [包管理器](editor/package-manager.md)
  - [UI Toolkit 架构](ui-toolkit/index.md)
    - [事件分发](ui-toolkit/events.md)
    - [样式、布局与渲染](ui-toolkit/styles-layout-rendering.md)
- [覆盖与一致性审计](audit.md)

## 推荐阅读路线

| 目标 | 路线 |
| --- | --- |
| 第一次接触这个仓库 | 概览 → 阅读与导航指南 → 物理组织 → 绑定层 |
| 理解脚本的生命周期与重载 | 对象模型 → 生命周期作用域 → 编辑器应用与回调 → 脚本编译 |
| 理解编辑器窗口如何显示 | 窗口系统 → IMGUI → UI Toolkit 架构 → 事件分发 |
| 编写编辑器扩展 | 编辑器架构 §3（扩展点模式）→ Inspector → 资源导入 → 播放器构建 |

## 覆盖限制摘要

详细结论见 [audit.md](audit.md)。主要限制：

- **原生侧行为不可验证**：帧循环顺序、对象销毁时序、资源数据库、构建主流程、绑定生成器和 IL 注入器都只能根据 C# 声明和注释推断。
- **没有测试证据**：仓库不包含测试，所有契约只能从实现和注释得出。
- **有意排除的区域**：166 个模块中有 147 个只做了目录级盘点，例如 Physics、Animation、Audio、ProfilerEditor、QuickSearch、GraphToolkitEditor、MultiplayerEditor；另有 10 个只在页面中提及，例如 UIBuilder、Burst。`Runtime/Export` 下的大部分领域 API（Graphics、Camera、RenderPipeline 等）、场景视图、Overlays、动画编辑器和版本控制也未分析。
- **未执行的命令**：没有构建、运行或测试代码。行号和链接经过脚本检查，命令记录见 [audit.md](audit.md)。
