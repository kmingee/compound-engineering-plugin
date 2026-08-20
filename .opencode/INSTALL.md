# 为 OpenCode 安装 Compound Engineering

将 Compound Engineering 添加到全局或项目 `opencode.json` 的 `plugin` 数组中：

```json
{
  "plugin": ["compound-engineering@git+https://github.com/EveryInc/compound-engineering-plugin.git"]
}
```

修改配置后重启 OpenCode。OpenCode plugin 会直接注册 Compound Engineering 的 skills 目录；无需 Bun 安装器，也无需生成 skill 副本。

如果要固定到某个版本，请添加 tag。将 `X.Y.Z` 替换为你需要的 release；可在 [releases 页面](https://github.com/EveryInc/compound-engineering-plugin/releases)查看可用 tag：

```json
{
  "plugin": ["compound-engineering@git+https://github.com/EveryInc/compound-engineering-plugin.git#compound-engineering-vX.Y.Z"]
}
```

## 本地开发

在这个 checkout 中，让 OpenCode 指向 package 路径：

```json
{
  "plugin": ["/path/to/compound-engineering-plugin"]
}
```

修改 package 来源后重启 OpenCode。
