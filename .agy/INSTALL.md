# 为 Antigravity CLI (`agy`) 安装 Compound Engineering

Antigravity 以原生插件包的形式安装 CE。仓库根目录就是插件包：`plugin.json` 加上 `skills/` 目录。已提交的 `.agy/` 子目录是一个兼容入口，供偏好显式插件包路径的本地 checkout 使用。

## 一条命令安装（推荐）

安装 [Antigravity CLI](https://antigravity.google) 后：

```bash
agy plugin install https://github.com/EveryInc/compound-engineering-plugin
```

验证安装：

```bash
agy plugin list
agy plugin validate https://github.com/EveryInc/compound-engineering-plugin
```

无需先 clone。`agy` 会将插件放到 `~/.gemini/antigravity-cli/plugins/compound-engineering/` 下。

## 从本地 checkout 安装

当你需要特定分支、tag 或尚未发布的改动时，先 clone 仓库：

```bash
git clone https://github.com/EveryInc/compound-engineering-plugin
agy plugin install ./compound-engineering-plugin
```

也可以安装随仓库提供的 `.agy/` 入口（通过符号链接指向等价的 manifest）：

```bash
agy plugin install ./compound-engineering-plugin/.agy
```

## 固定到某个版本

从 release tag 安装：

```bash
git clone --branch compound-engineering-vX.Y.Z --depth 1 \
  https://github.com/EveryInc/compound-engineering-plugin.git
agy plugin install ./compound-engineering-plugin
```

将 `X.Y.Z` 替换为 [releases 页面](https://github.com/EveryInc/compound-engineering-plugin/releases)中的 tag。

## 本地开发

在你的工作副本中：

```bash
agy plugin install "$PWD"
agy plugin validate "$PWD"
```

编辑 `skills/` 下的 skill，然后重启 `agy` 或新建一个 session，以加载文本改动。

## 上下文文件

`agy` 会读取 `GEMINI.md` 和 `AGENTS.md` 作为 workspace 上下文。请从包含这些文件的项目中运行 `agy`；如果是在开发 CE 本身，也可以直接从这个 checkout 中运行。

## 卸载

```bash
agy plugin uninstall compound-engineering
```

## 导入旧版 Gemini CLI 安装

如果你之前把 CE 安装在 Gemini CLI 下：

```bash
agy plugin import gemini
```

对于新的安装环境，优先使用上面的原生安装命令。
