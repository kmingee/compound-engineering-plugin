# Skill 文档

面向最终用户的 compound-engineering plugin skills 文档。每个页面都会介绍该 skill 的高层用途、新颖机制、使用场景，以及它与其他 skills 的链路位置。

对于 runtime 行为和贡献者参考，以 `skills/` 下各 skill 源目录中的 `SKILL.md` 为权威定义。

多个 skills 共用、仅对当前 checkout 生效的默认配置，记录在 [Compound Engineering 配置](./configuration.md)中。

这些页面中出现的 artifact 路径（`docs/plans/`、`docs/solutions/`、`docs/ideation/` 等）都是**默认值**。项目可以通过 `docs_root` 把所有 CE artifact 目录迁移到同一个仓库相对根目录下；设置后，应把文中路径理解为 `<your-docs_root>/plans/`、`<your-docs_root>/solutions/` 等。详见 [Artifact 根目录](./configuration.md#artifact-root)。

---

## compound-engineering 核心循环

```text
   [/ce-ideate]       (optional) "What's worth exploring?"
        │
        ▼
┌─→ /ce-brainstorm    "What does this need to be?"
│       │
│       ▼
│   /ce-plan          "What's needed to accomplish this?"
│       │
│       ▼
│   /ce-work          "Build it."
│       │
│       ▼
└── /ce-compound      "Capture what we learned."
```

`/ce-compound` 是让整个循环真正产生*复利*的收尾步骤：它把经验写入 `docs/solutions/`，下一轮的 `/ce-brainstorm` 和 `/ce-plan` 会读取这些内容作为 grounding。那条返回箭头就是核心。`/ce-ideate` 是可选前奏，适合你还不知道该做什么的时候。本目录里的其他 skills，要么是环绕核心循环的锚点，要么是在出现特定需求时按需调用的工具，并不是每次都要走一遍的固定步骤。

---

## 核心循环

每次工程迭代的主要步骤。只有在需要先寻找方向时才运行 `/ce-ideate`；其余四个步骤针对每一项工作按顺序运行。

| Skill | 说明 |
|-------|-------------|
| [`/ce-ideate`](./ce-ideate.md) | *可选第一步*：发现值得探索、且有 grounding 的方向（六种 frame、带标签的依据、adversarial cut） |
| [`/ce-brainstorm`](./ce-brainstorm.md) | 定义目标应该变成什么：一次一个问题，只问决策，使用具名 gap lenses，并写出仅含需求的统一 plan |
| [`/ce-plan`](./ce-plan.md) | 用 guardrails 限定执行范围（U-IDs、test scenarios、自动 confidence check）。决定 WHAT，而不是代码层面的 HOW |
| [`/ce-work`](./ce-work.md) | 执行可实施的 plan：面对代码决定 HOW，然后通过质量闸门完成交付 |
| [`/ce-compound`](./ce-compound.md) | 把学到的内容写入 `docs/solutions/`，闭合循环，让下一次迭代可以直接读取 |

---

## 环绕循环的 Skills

这些 skills 用来锚定、输入或维护核心循环，但本身不是循环中的固定步骤。

| Skill | 说明 |
|-------|-------------|
| [`/ce-strategy`](./ce-strategy.md) | 创建或维护 `STRATEGY.md`；它是 `ce-ideate`、`ce-brainstorm` 和 `ce-plan` 会读取的上游 grounding 锚点 |
| [`/ce-product-pulse`](./ce-product-pulse.md) | 外层观察循环：针对一个时间窗口汇总 usage、performance、errors 和 follow-ups，保存到 `docs/pulse-reports/` |
| [`/ce-sweep`](./ce-sweep.md) | 周期性反馈扫描：摄取 Slack/GitHub items（email 为实验性支持）、在来源处确认，并维护一份可直接交给 `/lfg` 的滚动 plan |
| [`/ce-compound-refresh`](./ce-compound-refresh.md) | 随时间维护 `docs/solutions/`（Keep / Update / Consolidate / Replace / Delete），支持 Interactive 或 Autofix |

---

## 按需使用

当特定需求出现时调用，不属于任何固定链路。

| Skill | 说明 |
|-------|-------------|
| [`/ce-pov`](./ce-pov.md) | 形成有项目 grounding 的明确判断：adopt/hold/reject、对一份文档的整体看法，或对给定 approaches 的立场。可选指定 peers/`oracle` panel。 |
| [`/ce-explain`](./ce-explain.md) | 为概念、diff、想法或最近一段工作生成可长期保存的教学文档。可选 opt-in check-in。 |
| [`/ce-prototype`](./ce-prototype.md) | 构建一次性 prototype，让人实际体验产品应该如何工作、呈现或表达；然后把这些决策写回现有 plan，或继续进入 brainstorm/plan |
| [`/ce-debug`](./ce-debug.md) | 找出错误行为的根因：建立 causal chain、做 predictions，然后可选地修复并进入 PR handoff |
| [`/ce-code-review`](./ce-code-review.md) | 对 diff 或 PR 做结构化 review：使用 skill 本地 personas，并对 findings 做 confidence gate |
| [`/ce-doc-review`](./ce-doc-review.md) | 对需求或 plan 文档做结构化 review：输出 findings，而不是整体 verdict |
| [`/ce-simplify-code`](./ce-simplify-code.md) | 在保持行为不变的前提下，整理最近修改的代码，提高复用、质量与效率 |
| [`/ce-optimize`](./ce-optimize.md) | 通过并行 experiments 和持久 experiment log 运行 metric-driven 优化循环 |
| [`/ce-retune`](./ce-retune.md) | 为新模型重新调校 skill corpus：先做 baseline 和 noise floor，再进行可测量的 cut passes |

---

## Research 与上下文

| Skill | 说明 |
|-------|-------------|
| [`/ce-riffrec-feedback-analysis`](./ce-riffrec-feedback-analysis.md) | 把 [Riffrec](https://github.com/kieranklaassen/riffrec) 录制转成结构化反馈：可以是在聊天中快速输出一个 bug，也可以做深入分析后 handoff 给 `ce-brainstorm` |

---

## Git 工作流

| Skill | 说明 |
|-------|-------------|
| [`/ce-commit`](./ce-commit.md) | 只创建本地 git commit(s)：理解约定、按指定文件 staging、可按文件拆分（最多三个）。不 push。 |
| [`/ce-commit-push-pr`](./ce-commit-push-pr.md) | 把工作中的改动变成 open PR。三种模式：完整交付、重写现有 description、或仅根据 URL 生成 description。 |
| [`/ce-babysit-pr`](./ce-babysit-pr.md) | 持续监看 open PR：review 到来时调用 `/ce-resolve-pr-feedback`，CI 出问题时调用 `/ce-debug`。在 `target` 或 `stack-ready` posture 下不 merge；`stack-land` 可 merge 已确认的 managed stack。 |
| [`/ce-worktree`](./ce-worktree.md) | 在 git worktree 中隔离工作：检测已有隔离，优先使用宿主原生工具，否则使用普通 git |

---

## 自主 Pipeline

| Skill | 说明 |
|-------|-------------|
| [`/lfg`](./lfg.md) | 免值守运行到 open PR 的完整 pipeline（plan、implement、review、ship、有边界地监看 CI）。存在 remote 时无需再次提示即可 push；否则只保留本地 commits。不 merge。 |

---

## 前端设计

| Skill | 说明 |
|-------|-------------|
| [`/ce-polish`](./ce-polish.md) | 对已经可用的功能进行对话式 UX polish：启动 dev server、打开 browser、持续迭代。只能手动调用。 |

---

## 协作

| Skill | 说明 |
|-------|-------------|
| [`/ce-proof`](./ce-proof.md) | 通过 [Proof](https://www.proofeditor.ai) 发布、查看、评论或拉取 Markdown。单向 publish；不是 review skill。 |

---

## 工作流工具

| Skill | 说明 |
|-------|-------------|
| [`/ce-promote`](./ce-promote.md) | 为已交付功能起草公告文案（X、changelog、LinkedIn、email、blog、demo）。只生成草稿；从不自动发布。 |
| [`/ce-resolve-pr-feedback`](./ce-resolve-pr-feedback.md) | 用一轮流程评估、修复并回复 PR review comments，包括 nitpicks。Babysit 会在监看中调用它。 |
| [`/ce-dogfood`](./ce-dogfood.md) | 对当前分支做免值守 browser QA：梳理 flows、修复小问题、写报告。只能手动调用。 |
| [`/ce-test-browser`](./ce-test-browser.md) | 用宿主原生 browser 对当前 diff 做端到端测试，并以 `agent-browser` 作为 fallback。不 checkout PR 或 branch。 |
| [`/ce-test-xcode`](./ce-test-xcode.md) | 在 simulator 中构建并测试 iOS app（screenshots、logs、人工验证）。不是 XCUITest。 |
| [`/ce-setup`](./ce-setup.md) | 诊断可选工具能力，并创建或修复仓库 `config.yaml` |
| [`/ce-handoff`](./ce-handoff.md) | 写入 session handoff，或查找并从选定来源完成定向。不自动继续后续工作。 |

---

## 另请参阅

顶层安装与使用指南见 [`README.md`](../../README.md)。每个 skill 的权威 runtime spec 位于 `skills/<skill>/SKILL.md`。
