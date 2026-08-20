# 为 Cline 安装 Compound Engineering

Cline 通过原生 **skills** 发现机制加载 CE——也就是本仓库 `skills/` 目录中随仓库发布的同一批 `SKILL.md` 目录。无需 Bun 转换器，也无需生成副本。

## 扩展（VS Code、Cursor、JetBrains）

1. 在编辑器中安装 [Cline 扩展](https://docs.cline.bot/getting-started/installing-cline)。
2. 启用 **Settings -> Features -> Enable Skills**。
3. 将 CE skills 全局链接，或链接到你的项目中（见下文）。
4. 新建一个 Cline task。当 `ce-brainstorm`、`ce-plan` 等 skill 的描述与你的请求匹配时，它们就会出现。

## 安装 skills

从本仓库的 clone 中运行：

```bash
# Global (~/.cline/skills/) — available in every project
./compound-engineering-plugin/.cline/scripts/install-skills.sh --global

# Project (.cline/skills/ in the current directory)
./compound-engineering-plugin/.cline/scripts/install-skills.sh --project
```

脚本会创建符号链接，让 Cline 直接读取这个 checkout 中实时的 skill 目录。执行 `git pull` 后，如果 skill 目录名发生变化，请重新运行脚本以刷新链接。脚本只会创建或替换 CE 自己拥有的符号链接（即目标解析到当前 checkout 的 `skills/` 树下的链接）；如果现有的 `~/.cline/skills/<name>` 指向你自己的 skill、某个 fork 或其他 checkout，它会保持不动。默认安装也只会移除由 CE 拥有的“仅手动”符号链接。

标记为 `disable-model-invocation: true` 的 skills（例如 `ce-dogfood`、`ce-polish`、`ce-setup`）默认**不会**被链接——Cline 会根据描述匹配自动激活 skill，但没有“仅手动”闸门，因此链接它们可能导致意外触发。在你主动选择启用之前，这些 slash command 不可用：

```bash
./compound-engineering-plugin/.cline/scripts/install-skills.sh --global --include-manual
```

`--include-manual` 会链接仅手动 skills，使 `/ce-polish` 等命令可用；同时会提示：当描述匹配时，Cline 仍有可能自动激活它们。如果你不需要在 Cline 中使用这些工作流，请省略该参数。

## 固定到某个版本

Clone 你需要的 tag，然后针对该 checkout 运行安装脚本：

```bash
git clone --branch compound-engineering-vX.Y.Z --depth 1 \
  https://github.com/EveryInc/compound-engineering-plugin.git
./compound-engineering-plugin/.cline/scripts/install-skills.sh --global
```

将 `X.Y.Z` 替换为 [releases 页面](https://github.com/EveryInc/compound-engineering-plugin/releases)中的 tag。

## 本地开发

在你的工作副本中：

```bash
/path/to/compound-engineering-plugin/.cline/scripts/install-skills.sh --global
```

编辑 `skills/` 下的 skill，然后新建一个 Cline task，以加载文本改动。

## 卸载

从 `~/.cline/skills/` 或 `.cline/skills/` 中移除 CE skill 的符号链接。Skill 目录名与 `skills/` 下的目录一致（例如 `ce-brainstorm`、`ce-plan`）。

## Cline CLI

Cline CLI 支持独立安装 `AgentPlugin`，用于自定义工具和 hooks。CE 的 skills 无需 CLI plugin 即可工作。从终端运行 Cline 时，请使用上面的 skills 安装脚本。
