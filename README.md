# Unity6-CS-Digest

本目录是对 Unity 6000.7.0b7 C# 参考源码（[UnityCsReference](https://github.com/Unity-Technologies/UnityCsReference)）的中文架构分析与导读，内容包括工程结构、运行时与编辑器核心子系统、UI Toolkit 以及模块目录。

> **说明**：本文档通过 claude-opus-5.5 xhigh 分析生成，仅供学习交流，不构成对 Unity 官方文档的替代，也不代表 Unity Technologies 的观点。

## 源码地址

- 原始源码仓库：<https://github.com/Unity-Technologies/UnityCsReference>
- 分析快照：提交 `6974f873c0dd24dd88840492397e3a7ce3b817ab`（Unity 6000.7.0b7 C# reference source code）

## 版权与使用限制

原始源码仅允许参考用途。根据仓库 README 所链接的 [Unity Reference Only License](https://unity3d.com/legal/licenses/Unity_Reference_Only_License)，不允许修改或再分发 C# 代码。本目录只包含分析文档，不包含引擎源码本身；如需修改源码，请以官方渠道的授权为准。

## 内容结构

| 文件 | 说明 |
| --- | --- |
| [index.md](index.md) | 入口页，包含页面树与推荐阅读路线 |
| [overview.md](overview.md) | 仓库用途、形态与高层架构 |
| [getting-started.md](getting-started.md) | 阅读与导航方法，以及跨托管/原生边界的追踪技巧 |
| [physical-layout.md](physical-layout.md) | 解决方案、工程、程序集与 `artifacts/` 组织 |
| [module-catalog.md](module-catalog.md) | 166 个模块的程序集归属与覆盖情况 |
| [runtime/](runtime/index.md) | 运行时架构：绑定层、对象模型、生命周期、PlayerLoop、序列化、Jobs |
| [editor/](editor/index.md) | 编辑器架构：应用回调、窗口、IMGUI、Inspector、脚本编译、资源导入、播放器构建、包管理器 |
| [ui-toolkit/](ui-toolkit/index.md) | UI Toolkit 架构：事件分发、样式、布局与渲染 |
| [audit.md](audit.md) | 覆盖范围与结论可信度审计 |

## 阅读前提

- **原生代码不在仓库中**：大量 API 通过 `extern` 指向 C++ 实现，文档中关于原生行为的描述均为根据 C# 侧代码推断。
- **没有测试与构建验证**：文档基于静态阅读撰写，未编译或运行源码。
- **覆盖有限**：大部分模块只做了目录级盘点，未逐一深入分析，具体范围见 [audit.md](audit.md)。

## 快速开始

第一次阅读建议从 [index.md](index.md) 开始，按“概览 → 阅读与导航指南 → 物理组织 → 绑定层”的顺序浏览。
