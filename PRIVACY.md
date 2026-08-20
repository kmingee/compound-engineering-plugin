# 隐私与数据处理

本仓库包含：
- 一个由 Markdown/配置内容组成的根级 plugin package
- 一个 CLI（`@every-env/compound-plugin`），用于为不同的 AI 编程工具转换并安装 plugin 内容

## 摘要

- Plugin package 不包含遥测或分析代码。
- Plugin package 不会运行后台服务来自动上传 repository/workspace 内容。
- 只有当你的宿主/工具，或你明确调用的集成发起网络请求时，数据才会离开你的设备。

## 哪些情况可能发送数据

1. AI 宿主/模型提供商

如果你在 Claude Code、Cursor、Codex、Gemini CLI、Copilot、Windsurf 等工具中运行本 plugin，这些工具可能会把 prompt、上下文或代码发送给其配置的模型提供商。此行为由这些工具和提供商控制，而不是由本 plugin 仓库控制。

2. 可选集成和工具

Plugin 包含一些可选能力，在你明确使用时可能调用外部服务，例如：
- Context7 MCP（`https://mcp.context7.com/mcp`），用于查找文档
- Proof（`https://www.proofeditor.ai`），用于 share/edit 流程
- 其他 opt-in skills（例如图像生成或云上传工作流），它们会调用各自的外部 API/服务

如果你不调用这些集成，它们就不会传输你的项目数据。

3. Package/安装器基础设施

安装依赖或 package（例如使用 `npm`、`bunx`）时，会按照你的 package manager 配置与 package registry/CDN 通信。

## 数据所有权与保留

本仓库不运营用于收集或存储你的项目/workspace 数据的后端服务。模型 prompt 或可选集成的数据保留与处理方式，由你使用的外部服务决定。

## 安全问题报告

如果你发现本仓库存在安全问题，请按照 [SECURITY.md](SECURITY.md) 中的披露流程处理。
