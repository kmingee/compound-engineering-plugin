# Compound Engineering

[![构建状态](https://github.com/EveryInc/compound-engineering-plugin/actions/workflows/ci.yml/badge.svg)](https://github.com/EveryInc/compound-engineering-plugin/actions/workflows/ci.yml)

让每一个工程工作单元，都比上一个更容易完成的 AI skills。

<a id="install"></a>
## 安装

### Claude Code

```text
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering
```

> **已经安装了 Compound Engineering？** Compound Engineering 已迁移到根目录原生布局。更新之前必须先刷新 marketplace——参见[已有安装](#existing-installs)。只运行 `/plugin update` 会让你继续停留在旧版本。

### Cursor

在 Cursor Agent 聊天中，从 plugin marketplace 安装：

```text
/add-plugin compound-engineering
```

也可以在 plugin marketplace 中搜索“compound engineering”。

### Grok Bot

Grok Bot 是独立应用，但它使用你的 Cursor 账号和 plugin library，没有单独的 Grok Bot 登录。只需在该账号上安装一次 Compound Engineering，Grok Bot agents 就可以加载它。

在 Cursor Agent 聊天中：

```text
/add-plugin compound-engineering
```

或者在 Cursor plugin marketplace 中搜索“compound engineering”。不要在 Grok Bot 聊天里运行 `/add-plugin`，也不要把这个仓库 clone 到 Grok Bot 所在的电脑上。

<a id="codex-app"></a>
### Codex App

Compound Engineering 目前还没有出现在 Codex 内置的 plugin marketplace 中。请把它添加为自定义 marketplace：

1. 在 Codex app 中，从侧边栏打开 **Plugins**。
2. 点击 **Create** 旁边的箭头，然后选择 **Add marketplace**。
3. 填写：

   | 字段 | 值 |
   | --- | --- |
   | Source | `EveryInc/compound-engineering-plugin` |
   | Git ref | `main` |
   | Sparse paths | 留空 |

4. 点击 **Add marketplace**。
5. 搜索 **Compound Engineering**，安装 **compound-engineering-plugin**，然后重启 Codex。

Codex app 的安装已经包含 Compound Engineering 所需的全部内容。专门的 reviewer 和 research 行为都作为本地 prompt 资源存在于 skills 内，无需额外安装 custom agent。

<a id="codex-cli"></a>
### Codex CLI

先注册 marketplace，再安装 plugin。

1. **在 Codex 中注册 marketplace：**

   ```bash
   codex plugin marketplace add EveryInc/compound-engineering-plugin
   ```

2. **安装 plugin：**

   ```bash
   codex plugin add compound-engineering@compound-engineering-plugin
   ```

   你也可以启动 `codex`，运行 `/plugins`，找到 **Compound Engineering** marketplace，选择 **compound-engineering** plugin，然后选择 **Install**。安装完成后重启 Codex。

原生 Codex plugin 安装已经包含 Compound Engineering 所需的全部内容。专门的 reviewer 和 research 行为都作为本地 prompt 资源存在于 skills 内，无需额外安装 custom agent。

如果使用非默认 Codex profile，所有 Codex 相关步骤都必须针对同一个 `CODEX_HOME` 执行。下面的示例把 CE 安装到名为 `work` 的 profile：

```bash
CODEX_HOME="$HOME/.codex/profiles/work" codex plugin marketplace add EveryInc/compound-engineering-plugin
CODEX_HOME="$HOME/.codex/profiles/work" codex plugin add compound-engineering@compound-engineering-plugin
```

Marketplace 步骤只是让 plugin 变得可用；真正激活该 profile 中原生 CE skills 的是 plugin 安装步骤。

#### 移除旧版 Codex tool map（原生安装之前的版本）

如果你过去通过 Bun `convert` / `install --to codex` CLI 安装 Compound Engineering（即原生 Codex plugin 支持出现之前），该流程可能在你的**全局** Codex instructions 文件中插入过一个受管理的区块：

`<!-- BEGIN COMPOUND CODEX TOOL MAP -->` … `<!-- END COMPOUND CODEX TOOL MAP -->`

它位于 `$CODEX_HOME/AGENTS.md`（默认 `~/.codex/AGENTS.md`）中。这个用于兼容 Claude 的 tool map 已经过时——CE skills 现在会直接在正文中写明 Codex tools——而且其中一行还曾错误地指示 Codex 把 subagent dispatch 折叠到主线程。原生 plugin 安装**不会**添加这个区块。

把下面这段内容粘贴给 Codex（或任何能够访问你 home directory 的 agent）即可移除它：

```text
Remove the obsolete Compound Engineering Codex tool-map block from my Codex home AGENTS.md.

1. Check `$CODEX_HOME/AGENTS.md` if CODEX_HOME is set, otherwise `~/.codex/AGENTS.md`. If I use Codex profiles, also check `~/.codex/profiles/*/AGENTS.md`.
2. Look for the exact sentinels `<!-- BEGIN COMPOUND CODEX TOOL MAP -->` and `<!-- END COMPOUND CODEX TOOL MAP -->`.
3. If both are present, delete only the span from the BEGIN line through the END line (inclusive), leaving any other user content untouched. Do not edit project/repo AGENTS.md unless those exact sentinels are present there.
4. If the file is empty after the removal, delete the file.
5. Show a short before/after summary of what you changed (or say the block was already absent). Do not add a replacement tool map.
```

重新运行当前版本用于 Codex 的 Bun convert/install CLI，也会移除仍然存在的这个区块；它不会再把该区块插回去。

**使用其他编辑器或 CLI？** Kimi Code CLI、Cline、Grok Build CLI、Devin CLI、GitHub Copilot、Factory Droid、Qwen Code、OpenCode、Pi、oh-my-pi (omp) 和 Antigravity CLI 都受支持——参见[更多安装方式](#more-install-options)。

---

## 理念

**每一个工程工作单元，都应该让后续工作更容易，而不是更困难。**

调用语法：本 README 对支持 slash skill 的宿主使用 `/skill-name` 示例。在 Codex 中，请用 `$skill-name` 调用已安装的 skill（例如 `$ce-plan` 和 `$lfg`）。在 oh-my-pi (omp) 中，这些 prompt 可以通过模型路由到可见的 skills；对于仅手动或隐藏的 skills，请使用原生且确定性的 `/skill:<name>` 形式（例如 `/skill:ce-polish`）。`/goal` 仍然是 Codex 的内置命令。

传统开发会不断累积技术债。每个功能都会增加复杂度，每次 bug 修复都会留下少量局部知识，之后还得有人重新发现。代码库越来越大，上下文越来越难掌握，下一次改动也就越来越慢。

Compound engineering 把这个过程反过来：80% 的精力放在规划和审查，20% 放在执行：

- 写代码前，用 `/ce-brainstorm` 和 `/ce-plan` 围绕同一个、基于 readiness 的 plan artifact 做充分规划
- 用 `/ce-code-review` 和 `/ce-doc-review` 审查问题，并校准判断
- 用 `/ce-compound` 把知识固化为可复用内容
- 持续维持高质量，让未来的改动更容易

重点不是增加仪式，而是获得杠杆。好的 brainstorm 会让 plan 更清晰；好的 plan 会让执行范围更小；好的 review 会抓住模式，而不只是某个 bug；好的 compound note 会让下一个 agent 不必从头重学同一个教训。

**进一步了解**

- [Skill 文档目录](docs/skills/README.md)
- [Compound engineering：Every 如何与 agents 一起写代码](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents)
- [Compounding engineering 背后的故事](https://every.to/source-code/my-ai-had-already-fixed-the-code-before-i-saw-it)

## 工作流

核心循环有六步：**brainstorm** 需求、**plan** 实现、按 plan **work**、**simplify** 刚写出的代码、**review** 结果，然后把经验 **compound** 下来——再带着更好的上下文重复下一轮。

| Skill | 用途 |
|-------|---------|
| [`/ce-brainstorm`](docs/skills/ce-brainstorm.md) | 通过交互式问答梳理功能或问题，并在正式规划前写出只包含需求的统一 plan |
| [`/ce-plan`](docs/skills/ce-plan.md) | 把功能想法或仅含需求的 plan 丰富为可直接实施的 plan |
| [`/ce-work`](docs/skills/ce-work.md) | 原生执行可实施 plan，或交给符合条件的跨模型 author 执行，同时保留宿主侧验证、commit 与交付流程 |
| [`/ce-simplify-code`](docs/skills/ce-simplify-code.md) | 在 review 前整理刚写出的代码，提高可读性与复用性 |
| [`/ce-code-review`](docs/skills/ce-code-review.md) | 合并前针对 plan 做仅报告的 multi-agent review；是否在本地应用修改由用户明确决定 |
| [`/ce-compound`](docs/skills/ce-compound.md) | 把经验写入 `docs/solutions/`，让下一轮从更聪明的起点开始 |

每一轮都会产生复利：`/ce-compound` 写出的经验会成为下一次 `/ce-brainstorm` 和 `/ce-plan` 的 grounding——brainstorm 让 plan 更精准，plan 会反过来帮助未来的 plan，review 能发现更多问题，模式被记录下来。那条返回箭头就是整个方法的核心。

> `docs/solutions/`、`docs/plans/` 这类 artifact 目录只是**默认值**。如果项目本身把 `docs/` 用作受版本控制的正式内容，可以通过 `docs_root` 设置，把所有 CE artifact 目录统一迁移到一个仓库相对路径下——参见[配置](docs/skills/configuration.md#artifact-root)。

### 其他 skills

这些 skills 位于核心循环周围，或在需要时按需使用——不是每一轮都要跑。

| Skill | 什么时候使用 |
|-------|---------|
| [`/ce-ideate`](docs/skills/ce-ideate.md) | *进入循环之前*，当你还不知道该做什么——生成有 grounding 的想法并进行严格排序，然后把最强候选送入 `/ce-brainstorm` |
| [`/ce-strategy`](docs/skills/ce-strategy.md) | *上游锚点*——创建并维护 `STRATEGY.md`；ideate、brainstorm 和 plan 会读取它作为 grounding，让战略选择进入每个功能 |
| [`/ce-product-pulse`](docs/skills/ce-product-pulse.md) | *外层循环*——针对某个时间窗口汇总用户真实经历（usage、performance、errors），保存到 `docs/pulse-reports/`；后续行动再流回 ideation 和 brainstorming |
| [`/ce-debug`](docs/skills/ce-debug.md) | 当输入是 bug 而不是功能时，*替代 brainstorm -> plan -> work*——复现、追踪根因、修复，然后在需要时 polish/review，再进入 PR handoff |
| [`/ce-pov`](docs/skills/ce-pov.md) | *按需，在做出承诺之前*——针对采用某方案、整份文档或给定 approaches 形成明确且有项目 grounding 的判断；可选地通过 blind initial round 和有边界的 reconciliation，让指定 peers 或 `oracle` 交叉检查 |
| [`/ce-explain`](docs/skills/ce-explain.md) | *按需，用来说明或理解工作*——把概念、diff、想法或“我这周做了什么？”变成信息密集、可长期保存的独立视觉文档；当内容值得记住时，还可加入可选 check-in（diff 使用 predict-then-reveal，练习会给出纠正） |

完整目录以及 skills 如何串联，请参阅 [docs/skills](docs/skills/README.md)。完整清单见[下方](#full-skill-inventory)。

## 快速示例

**寻找方向**——当你还没有具体想法时，先 ideate，再把最强的候选带入循环：

```text
/ce-ideate new drawing tools
/ce-ideate surprise me
/ce-ideate open issues     # ground ideas in your tracker's open issues (GitHub, Linear, Jira)
```

`/ce-ideate` 会先做功课（代码库、过去的经验、Web 上的 prior art，以及可选的 issue tracker），然后给出一组有 grounding 且已排序的候选，供你带入 `/ce-brainstorm`。

**标准功能循环**——把粗略想法变成已交付、已审查的代码：

```text
/ce-brainstorm make background job retries safer
/ce-plan
/ce-work
/ce-simplify-code
/ce-code-review
/ce-compound
```

**简化代码**——在刚完成实现后使用，或者直接指向某段持续拖慢改动的代码：

```text
/ce-simplify-code
/ce-simplify-code simplify the code in my most-churned file
```

第一次调用会在 review 前收紧当前分支最近的改动。定向调用适用于某个文件持续吸收无关修复、follow-up 或 merge conflict 的情况。

**调试 bug**——当起点是错误行为而不是新功能时：

```text
/ce-debug the checkout webhook sometimes creates duplicate invoices
/ce-code-review
/ce-compound
```

**自主执行**——把功能交出去，让 agent 跑完整条 pipeline：

```text
/ce-brainstorm describe the feature
/lfg
```

`/lfg` 会免值守地跑完整个循环：规划、按 plan 工作、简化、执行 code review 并应用修复、运行 browser tests，然后 commit。如果存在 git remote，它会 push、打开 PR，并通过一个有边界的修复循环监看 CI（不会 merge；如果耗尽修复预算，也可能带着 leftovers 结束）。没有 remote 时，它会停在本地 commits。请在 `/ce-brainstorm` 之后启动它，这样它会基于真实需求做 plan，而不是围绕一句话 prompt 规划。如果你想离开一段时间，回来时直接看到一个 open PR，它就是标准循环的 autopilot 版本。当一个符合条件的多区域 plan 仍有尚未规划的工作时，`lfg` 还会推荐并说明下一个应该单独规划的区域；只有你接受后，它才会为新的 session 和独立 plan 创建 `/ce-handoff`。

## 开始使用

安装后，在任意项目中运行 `/ce-setup`。它会报告可选工具能力；如果仓库缺少 `.compound-engineering/config.yaml` 则创建它；刷新已提交的 example；并把已有的本地 override 加入 gitignore。

`compound-engineering` plugin 目前包含 33 个 skills。其核心工作流会根据需要生成专门的 subagents，用于 research、review、planning 和 implementation。每个 skill 都会用自己的 prompt 去初始化通用 subagent，而不是依赖独立 plugin agents，因此即使不同 harness 对正式 agent 定义的处理方式不同，这些工作流仍能保持可移植性。

<a id="full-skill-inventory"></a>
### 完整 Skill 清单

| Skill | 用途 |
|-------|---------|
| [`/ce-strategy`](docs/skills/ce-strategy.md) | 创建或维护 `STRATEGY.md` |
| [`/ce-ideate`](docs/skills/ce-ideate.md) | 生成并严格评估有 grounding 的想法 |
| [`/ce-pov`](docs/skills/ce-pov.md) | 针对采用某方案、文档或一组 approaches 形成明确且有项目 grounding 的 POV |
| [`/ce-explain`](docs/skills/ce-explain.md) | 把概念、diff、想法或你自己的某段工作窗口记录为可长期保留的视觉 artifact |
| [`/ce-brainstorm`](docs/skills/ce-brainstorm.md) | 探索需求并写出大小合适的需求文档 |
| [`/ce-plan`](docs/skills/ce-plan.md) | 创建结构化实施计划 |
| [`/ce-work`](docs/skills/ce-work.md) | 通过原生或跨模型 implementation 执行 plan，保留持久进度，并由宿主以事务方式完成 integration |
| [`/ce-code-review`](docs/skills/ce-code-review.md) | 使用 skill 本地 reviewer personas 审查代码 |
| [`/ce-doc-review`](docs/skills/ce-doc-review.md) | 审查需求和 plan 文档 |
| [`/ce-debug`](docs/skills/ce-debug.md) | 复现故障、追踪根因、修复 bug，并为非小改动准备 PR |
| [`/ce-compound`](docs/skills/ce-compound.md) | 记录已解决的问题，让团队知识持续复利 |
| [`/ce-compound-refresh`](docs/skills/ce-compound-refresh.md) | 刷新过时或发生漂移的经验 |
| [`/ce-optimize`](docs/skills/ce-optimize.md) | 运行迭代式优化循环 |
| [`/ce-retune`](docs/skills/ce-retune.md) | 以 measurement-first 的方式，为新模型重新调校 skill corpus |
| [`/ce-product-pulse`](docs/skills/ce-product-pulse.md) | 生成限定时间窗口的 product pulse 报告 |
| [`/ce-riffrec-feedback-analysis`](docs/skills/ce-riffrec-feedback-analysis.md) | 把 Riffrec 录制或 notes 转换为结构化反馈 |
| [`/ce-sweep`](docs/skills/ce-sweep.md) | 扫描反馈来源、跟踪 item 生命周期，并输出可直接交给 `/lfg` 的 plan |
| [`/ce-resolve-pr-feedback`](docs/skills/ce-resolve-pr-feedback.md) | 处理 PR review 反馈 |
| [`/ce-commit`](docs/skills/ce-commit.md) | 创建带有清晰 message 的 git commit |
| [`/ce-commit-push-pr`](docs/skills/ce-commit-push-pr.md) | Commit、push 并打开 PR（或构建/提交用户主动选择的 managed stack），同时讲清楚本次改动新引入的任何概念 |
| [`/ce-babysit-pr`](docs/skills/ce-babysit-pr.md) | 监看 open PR（或在既定 posture 下已确认的 managed stack），根据 review comments 与 CI 状态持续推动其走向 merge |
| [`/ce-worktree`](docs/skills/ce-worktree.md) | 确保工作发生在隔离的 git worktree 中 |
| [`/ce-promote`](docs/skills/ce-promote.md) | 起草面向用户的公告文案 |
| [`/ce-test-browser`](docs/skills/ce-test-browser.md) | 针对 PR 影响到的页面运行 browser tests |
| [`/ce-test-xcode`](docs/skills/ce-test-xcode.md) | 在 simulator 中构建并测试 iOS app |
| [`/ce-setup`](docs/skills/ce-setup.md) | 诊断可选工具能力并创建仓库 `config.yaml` |
| [`/ce-handoff`](docs/skills/ce-handoff.md) | 在默认临时存储或指定目的地创建 session handoff，然后从选定来源恢复 |
| [`/ce-simplify-code`](docs/skills/ce-simplify-code.md) | 简化最近的代码改动 |
| [`/ce-prototype`](docs/skills/ce-prototype.md) | 构建一次性 prototype，让人能亲自体验产品应该如何工作、呈现或表达 |
| [`/ce-polish`](docs/skills/ce-polish.md) | 启动 dev server 并迭代 UX polish |
| [`/ce-proof`](docs/skills/ce-proof.md) | 创建、编辑并分享 Proof 文档 |
| [`/ce-dogfood`](docs/skills/ce-dogfood.md) | 针对当前分支 diff 范围做免值守 browser QA，并自动修复 |
| [`/lfg`](docs/skills/lfg.md) | 完整自主工程工作流 |

---

<a id="more-install-options"></a>
## 更多安装方式

[Claude Code、Cursor 和 Codex](#install) 的安装方法在最上方。本节列出的其他方式同样受支持。

### Kimi Code CLI

Kimi Code CLI 可以直接从本仓库安装 Compound Engineering，因为仓库自带原生 `.kimi-plugin/plugin.json` manifest：

```text
/plugins install https://github.com/EveryInc/compound-engineering-plugin
```

你也可以通过 Kimi 的自定义 marketplace 流程浏览它：

```text
/plugins marketplace https://raw.githubusercontent.com/EveryInc/compound-engineering-plugin/main/.kimi-plugin/marketplace.json
```

安装或更新后，运行 `/reload` 或开启新的 Kimi session，让 plugin skills 被加载。

### Cline

Cline 从按需 `SKILL.md` 目录加载 CE skills。先在 Cline 扩展中启用 **Settings -> Features -> Enable Skills**，然后把本仓库的 skills 全局链接或按项目链接：

```bash
git clone https://github.com/EveryInc/compound-engineering-plugin
./compound-engineering-plugin/.cline/scripts/install-skills.sh --global
```

从 checkout 进行按项目安装：

```bash
./compound-engineering-plugin/.cline/scripts/install-skills.sh --project
```

安装或更新 skills 后，新建一个 Cline task。有关固定版本、本地开发和卸载步骤，请参阅 [`.cline/INSTALL.md`](.cline/INSTALL.md)。

### Grok Build CLI (`grok`)

xAI 的 [Grok Build CLI](https://x.ai/cli)（`grok`）可以直接从本仓库安装 Compound Engineering——仓库根目录本身就是有效的 Grok plugin（`grok` 会读取已有的 Claude-compatible manifests，同时仓库还提供原生 `.grok-plugin/plugin.json`）：

```bash
grok plugin install EveryInc/compound-engineering-plugin
```

这种方式会跟踪仓库；运行 `grok plugin update` 即可拉取最新版本。如果你更想把它作为 marketplace source 浏览，仓库还提供原生 `.grok-plugin/marketplace.json`：

```bash
grok plugin marketplace add EveryInc/compound-engineering-plugin
grok plugin install compound-engineering
```

两种方式都会直接跟踪仓库（不固定 commit），因此无需 Bun 安装步骤。加入 `--trust` 可以跳过安装确认。`grok` 会把配置保存在 `~/.grok`；安装后请开启新的 session，让 skills 被加载。

Compound Engineering 也正在提交到官方 [xAI plugin marketplace](https://github.com/xai-org/plugin-marketplace)；维护者操作手册见 [`docs/grok-marketplace-submission.md`](docs/grok-marketplace-submission.md)。

### Devin CLI

Devin CLI 可以直接从 GitHub 安装 Compound Engineering，因为仓库自带原生 `.devin-plugin/plugin.json` manifest：

```bash
devin plugins install EveryInc/compound-engineering-plugin
```

验证安装并查看 skills：

```bash
devin plugins list
devin plugins info compound-engineering
```

运行 `devin plugins update compound-engineering` 即可更新到最新版本。Plugins 在 session 启动时加载，因此安装或更新后请开启新的 Devin session，skills 才会出现（以 `/compound-engineering:<skill>` slash commands 的形式）。

少数 skills 声明了 Devin 不映射的 Claude-style `allowed-tools` 名称（例如 `Bash`）；这些 skills 仍可使用，但部分动作会请求权限，而不是自动获批。详见 [`docs/specs/devin.md`](docs/specs/devin.md)。

### GitHub Copilot

对于 **VS Code Copilot Agent Plugins**：

1. 从 VS Code command palette 运行 `Chat: Install Plugin from Source`
2. 仓库填写 `EveryInc/compound-engineering-plugin`
3. 当 VS Code 显示本仓库中的 plugins 时，选择 `compound-engineering`

对于 **Copilot CLI**：

在 Copilot CLI 内：

```text
/plugin marketplace add EveryInc/compound-engineering-plugin
/plugin install compound-engineering@compound-engineering-plugin
```

在带有 `copilot` binary 的 shell 中：

```bash
copilot plugin marketplace add EveryInc/compound-engineering-plugin
copilot plugin install compound-engineering@compound-engineering-plugin
```

Copilot CLI 会读取现有的 Claude-compatible plugin manifests，因此无需额外的 Bun 安装步骤。

### Factory Droid

在带有 `droid` binary 的 shell 中：

```bash
droid plugin marketplace add https://github.com/EveryInc/compound-engineering-plugin
droid plugin install compound-engineering@compound-engineering-plugin
```

Droid 使用 `plugin@marketplace` 形式的 plugin ID；这里 `compound-engineering` 是 plugin，`compound-engineering-plugin` 是 marketplace 名。Droid 会安装现有的 Claude Code-compatible plugin，并自动转换格式，因此无需 Bun 安装步骤。

### Qwen Code

```bash
qwen extensions install EveryInc/compound-engineering-plugin:compound-engineering
```

Qwen Code 会直接从 GitHub 安装 Claude Code-compatible plugins，并在安装过程中转换 plugin 格式，因此无需 Bun 安装步骤。

### OpenCode

将 Compound Engineering 添加到全局或项目 `opencode.json` 的 `plugin` 数组：

```json
{
  "plugin": ["compound-engineering@git+https://github.com/EveryInc/compound-engineering-plugin.git"]
}
```

修改配置后重启 OpenCode。OpenCode plugin 会直接注册 Compound Engineering skills 目录；无需 Bun 安装器或生成 skill 副本。固定版本示例见 [`.opencode/INSTALL.md`](.opencode/INSTALL.md)。

### Pi

从本仓库把 Compound Engineering 安装为 Pi package：

```bash
pi install git:github.com/EveryInc/compound-engineering-plugin
```

对于会 dispatch reviewer、research 或 implementation subagent 的 CE 工作流，以下 companion 是必需的：

```bash
pi install npm:pi-subagents
```

如需更丰富的 blocking questions，推荐安装：

```bash
pi install npm:pi-ask-user
```

### oh-my-pi (omp)

oh-my-pi (omp) 通过自己的 marketplace 流程安装 Compound Engineering。仓库提供原生 `.omp-plugin/marketplace.json` catalog，其中的 plugin entry 带有由 release 管理的 `version`，因此 omp 的更新检查器能够看到每一次新的 CE release：

```text
omp plugin marketplace add EveryInc/compound-engineering-plugin
omp plugin install compound-engineering@compound-engineering-plugin
```

如果希望自动保持最新，请启用 auto-update：

```bash
omp config set marketplace.autoUpdate auto
```

默认的 `notify` 模式只会把可用更新写入 debug log——不会弹出提示——因此如果不用 `auto`，你不会主动得知新 release。若想手动升级，请运行 `omp plugin upgrade compound-engineering@compound-engineering-plugin`。

<details>
<summary>其他安装路径（固定 snapshot 与贡献者开发）</summary>

`omp install https://github.com/EveryInc/compound-engineering-plugin` 会把仓库安装为 npm-style plugin。这个路径**没有更新机制**——请把它视为固定某个 snapshot，而不是推荐的安装方式。

本地开发时，请从 checkout 使用实时符号链接：

```bash
omp plugin link "$PWD"
```

</details>

安装后运行 `/reload-plugins`，或开启新的 omp session，让 skills 被加载。omp 的原生确定性命令是 `/skill:<name>`（例如 `/skill:ce-plan`）；普通 `/skill-name` prompt 也可以由模型路由到可见 skills，但仅手动或隐藏 skills 必须使用原生形式。详见 [`docs/specs/omp.md`](docs/specs/omp.md)。

### Antigravity CLI (`agy`)

Google 已用 [Antigravity CLI](https://antigravity.google)（`agy`）取代面向消费者的 Gemini CLI，不过它仍运行 Gemini models。可以直接从 GitHub 安装 Compound Engineering——无需先 clone：

```bash
agy plugin install https://github.com/EveryInc/compound-engineering-plugin
```

用 `agy plugin list` 验证。仓库根目录就是 plugin package（`plugin.json` 加 `skills/`）。

本地 checkout 或固定 release 的安装方式：

```bash
git clone https://github.com/EveryInc/compound-engineering-plugin
agy plugin install ./compound-engineering-plugin
```

随仓库提供的 `.agy/` 目录仍是兼容入口（`agy plugin install ./compound-engineering-plugin/.agy`）。`agy` 也会从 checkout 加载 `GEMINI.md` workspace 上下文。

固定版本、本地开发、卸载和旧版 Gemini 导入方式见 [`.agy/INSTALL.md`](.agy/INSTALL.md)。

<a id="existing-installs"></a>
### 已有安装

Compound Engineering 已迁移到根目录原生、仅 skills 的布局。已有 marketplace 安装会保留一个**缓存的** marketplace snapshot，而旧 snapshot 仍指向原来的 `plugins/compound-engineering` 路径。因此如果只更新 plugin，它会继续读取过期 snapshot，让你停留在上一版本。必须**先**刷新缓存 marketplace，再更新 plugin——顺序很重要。

**Claude Code**

```text
/plugin marketplace update compound-engineering-plugin
/plugin update compound-engineering
```

**Codex CLI**

```bash
codex plugin marketplace upgrade compound-engineering-plugin
codex plugin add compound-engineering@compound-engineering-plugin
```

没有 `codex plugin update`；重新运行 `add` 会从已刷新的 snapshot 重新安装。如果使用非默认 profile，请让两条命令都针对同一个 `CODEX_HOME`。

**Codex App**

在 **Plugins** 面板刷新 marketplace（如果没有 refresh 控件，就移除后重新添加 `EveryInc/compound-engineering-plugin` marketplace），然后重新安装 **compound-engineering** 并重启 Codex。

**Grok Bot**

在对应的 Cursor 账号上重新安装或刷新 Compound Engineering（在 Cursor Agent 聊天中运行 `/add-plugin compound-engineering`，或通过 marketplace 搜索）。之后 Grok Bot 会从共享 plugin library 加载新 snapshot。正常更新时，不要把这个仓库 clone 到 Grok Bot 所在的电脑上。

如果你给某个宿主配置过直接路径或 sparse path，且路径位于 `plugins/compound-engineering` 下，请编辑或重新安装该来源，让它指向仓库根目录，并去掉 sparse path。

如果之前由 Bun 安装的副本仍在遮蔽原生 plugin skills，请从本仓库的一个 checkout 运行当前 cleanup 命令：

```bash
git clone https://github.com/EveryInc/compound-engineering-plugin.git /tmp/compound-engineering-plugin-cleanup
cd /tmp/compound-engineering-plugin-cleanup
bun install
bun run cleanup --target all
```

---

## 本地开发

```bash
bun install
bun test
bun run release:validate
```

### 从本地 checkout 加载

主动开发时，请让你要测试的 harness 直接加载当前 checkout。

**Claude Code**

```bash
claude --plugin-dir "$PWD"
```

**Cursor Agent CLI**

```bash
cursor-agent --plugin-dir "$PWD"
```

**Codex**

如果你只是想要接近生产环境的普通 plugin 安装，请使用上面的 [Codex App](#codex-app) 或 [Codex CLI](#codex-cli) 说明。下面的工作流只面向贡献者：他们需要让 Codex 加载某个精确 checkout 或 linked worktree 中尚未发布的文件。

<details>
<summary><strong>高级：在 Codex 中测试当前精确 checkout</strong></summary>

把当前 worktree 设为正在使用的 Codex 开发来源：

```bash
bun run codex:dev -- local
```

这会在 `$CODEX_HOME/skills/compound-engineering-local`（默认 `~/.codex/skills/compound-engineering-local`）创建一个 collection symlink，指向当前 worktree 的 `skills/` 目录。它还会通过 Codex CLI 移除已安装的 Compound Engineering plugin variants，避免缓存的 marketplace plugin 遮蔽或重复本地 skills。它不会复制 skills、切换 checkout、执行 Git pull，也不会触碰 `$CODEX_HOME/skills` 下不相关的条目。

这个链接会暴露选中 worktree 中的精确内容，包括 modified 和 untracked skills。因此普通编辑无需重新安装，而且当前版本的 Codex 会自动检测直接的 skill 改动。在 local 和 remote 安装模式之间切换后，请新开 session；如果普通 skill 编辑没有出现，再重启 Codex。

使用以下命令检查和切换模式：

```bash
bun run codex:dev -- status
bun run codex:dev -- refresh
bun run codex:dev -- remote
bun run codex:dev -- remove
```

- `status` 会报告 local、remote、mixed、drifted 或 absent 状态，以及 linked checkout、worktree kind、branch、commit SHA 和 dirty counts。
- `refresh` 是 `local` 的幂等 alias；可用于修复意外的 plugin 安装。实时链接本身已经会反映文件变化。
- `remote` 会刷新官方 Git marketplace，安装并验证 `compound-engineering@compound-engineering-plugin`，然后移除本地链接。用它模拟 release 后的用户体验。
- `remove` 会移除 Compound Engineering plugin variants 和受管理链接，但保留 checkout 以及不相关的用户 skills。

脚本会自行推导仓库路径，因此无论 checkout 位于哪里都能工作，包括含空格的路径。它继承当前 `CODEX_HOME`；测试隔离 profile 时，请在命令上设置 `CODEX_HOME`。所有模式都必须针对你启动 Codex 时使用的同一个 `CODEX_HOME` 运行。

不要在实时本地开发中使用 `codex plugin marketplace add "$PWD"`。它会安装当前 checkout 的缓存副本，因此之后的编辑不会体现，除非重新安装 plugin；即使 manifest version 相同，也不能证明缓存与 worktree 一致。`codex:dev` 工作流则会让 Codex 始终链接当前 skill 文件。

</details>

**Kimi Code CLI**

在 Kimi Code CLI 中：

```text
/plugins install /path/to/compound-engineering-plugin
```

如果要测试本地 marketplace catalog，请传入 catalog 路径：

```text
/plugins marketplace /path/to/compound-engineering-plugin/.kimi-plugin/marketplace.json
```

**Cline**

```bash
/path/to/compound-engineering-plugin/.cline/scripts/install-skills.sh --global
```

在 Cline 扩展中启用 **Settings -> Features -> Enable Skills**，然后开启新的 task。

**Devin CLI**

```bash
devin plugins install /path/to/compound-engineering-plugin
```

本地安装会链接到 checkout，而不是复制内容，因此 skill 编辑会在下一个 Devin session 生效，无需重新安装。

**OpenCode**

```json
{
  "plugin": ["/path/to/compound-engineering-plugin"]
}
```

修改 `opencode.json` 后重启 OpenCode。

**Pi**

```bash
pi -e "$PWD"
```

**oh-my-pi (omp)**

```bash
omp plugin link "$PWD"
```

**Antigravity CLI (`agy`)**

```bash
agy plugin install "$PWD"
agy plugin validate "$PWD"
```

也可以安装随仓库提供的 `.agy/` 入口：

```bash
agy plugin install "$PWD/.agy"
```

远程安装和固定版本示例见 [`.agy/INSTALL.md`](.agy/INSTALL.md)。

## 限制

OpenCode、Pi 和 oh-my-pi (omp) 使用来自本仓库的原生 package/plugin 加载方式。Bun CLI 仍保留给仓库开发与 converter 维护使用，不再是正常安装路径。

Release version 由 release automation 管理。常规 feature PR 不应手动提升 plugin 或 marketplace manifest version。

## 常见问题

### 安装 Compound Engineering 需要 Bun 吗？

不需要。Bun 只用于仓库开发任务和 converter 维护。

### 在哪里查看所有可用 skills？

本 README 中有 skill 清单，更详细的目录在 [`docs/skills/README.md`](docs/skills/README.md)。每个 skill 的权威 runtime spec 位于 `skills/<skill>/SKILL.md`。

### 在哪里查看 release 历史？

GitHub Releases 是 release notes 的权威来源。根目录 [`CHANGELOG.md`](CHANGELOG.md) 会指向该历史记录。

## 贡献

欢迎贡献。Issues、bug reports 和 pull requests 都能帮助这个项目变得更好，我们由衷感谢——尤其感谢 bug reports。

关于预期，有一点需要提前说明：Compound Engineering 从设计上就是有明确立场的。它由 [@kieranklaassen](https://github.com/kieranklaassen) 和 [@tmchow](https://github.com/tmchow) 维护，发展方向体现了他们对于 AI-assisted engineering 应该如何工作的具体观点。因此我们欢迎帮助，但无法承诺接受每一个改动——有些 proposal 即使本身是好想法，也可能不符合这个方向。

欢迎开 issue 或提交 PR；只要能把 plugin 推向正确方向，我们就会尽量吸收。我们只是希望提前说清楚：并不是所有改动都会被合入。

## 许可证

[MIT](LICENSE)