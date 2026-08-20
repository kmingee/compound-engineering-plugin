# `ce-brainstorm`

> 一次只问一个问题，想清楚目标最终应该变成什么，然后写出大小合适、只包含需求的统一 plan。

`ce-brainstorm` 是一个**定义问题与目标**的 skill。适合在你已经有方向、但开放问题仍是“它到底应该是什么？”时使用。它每个 turn 只问一个问题；用具名 gap lenses 对前提做 pressure-test；在推荐某个方案之前先列出 2–3 个具体 approaches；对于软件工作，还会写出只包含需求的统一 plan，避免 planning 阶段凭空发明产品行为。

它既适用于软件功能，也适用于非软件主题（活动、商业决策、旅行、命名 brief），以及两者之间的工作。软件路径会写 requirements-only unified plan。非软件路径保持 facilitation mode：先在聊天中做 synthesis，然后可选 handoff 给 `ce-plan`，由它生成适合该领域的 plan。

这是 compound-engineering ideation 链中的中间步骤。如果你已经有明确需求，或者连方向都还没有，就跳过它：

```text
/ce-ideate         /ce-brainstorm      /ce-plan             /ce-work
"What's worth      "What does this     "What's needed       "Build it."
 exploring?"        need to be?"        to accomplish
                                        this?"
```

它也经常作为独立入口使用：问题不是“我该怎么做？”，而是“我到底在做什么，这个形状对吗？”

它不会给出是否采用某方案的 verdict。如果请求是在一个已命名的外部候选上做是否采用的决策（某项技术、library、pattern、platform 或 architecture，并针对当前项目判断），skill 会先提议使用 `/ce-pov`，而不是直接为一项你还没有决定要做的工作划 scope。这只是 offer，不会静默切换。没有单一候选的开放式设计仍留在这里。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | 通过协作式对话澄清 scope、pressure-test 前提、探索 approaches，并写出 requirements-only unified plan |
| 什么时候用？ | 模糊的功能想法、存在多个合理方向、scope 不清楚、陌生领域中的工作、非软件决策 |
| 会产出什么？ | 软件：在 `docs/plans/` 中写一份带有 `artifact_readiness: requirements-only` 和 R/A/F/AE IDs 的 requirements-only unified plan。非软件：聊天 synthesis，可选保存、可选发布到 Proof、可选 handoff 给 `ce-plan`。轻量对齐可以不写文档。 |
| 下一步是什么？ | 软件：创建 implementation plan（推荐 `ce-plan`）、用 `lfg` 自主交付、pressure-test requirements 或 prototype 尚未解决的 feel-question、在 browser 中打开 HTML artifact，或继续提问。非软件：创建 plan、保存 summary、发布到 Proof，或停止。 |

---

## 调用示例

空调用会先问你想探索什么。传入已有 requirements-only plan 的路径会提供 resume。`output:html` 会改变 artifact 格式。指定 model 只会提升 approach generation 那一步。

```text
# Ask what to explore, then start the dialogue
/ce-brainstorm

# Shape an ambitious feature or project before committing to a plan
/ce-brainstorm design a self-serve migration platform for enterprise customers

# Turn a rough feature idea into a requirements artifact
/ce-brainstorm add a way for users to pause notifications

# Explore a problem without prescribing the solution up front
/ce-brainstorm support agents get paged overnight for non-urgent events

# Resume or continue an existing requirements-only plan instead of starting a duplicate
/ce-brainstorm docs/plans/2026-08-10-feat-notification-mute-plan.md

# Name an ideate survivor already in this conversation. Its tagged basis,
# rationale, and tradeoffs travel with it (same as picking "Brainstorm one idea")
/ce-brainstorm the per-channel mute idea

# Brainstorm non-software work with the same one-question discipline
/ce-brainstorm plan a two-day customer advisory workshop

# Unfamiliar territory: offer a blindspot map before questioning that area
/ce-brainstorm I know nothing about color grading but need a review workflow for it

# Verdict-shaped: offer ce-pov rather than scoping an adoption you have not made
/ce-brainstorm should we adopt Biome in this repo?

# Ask for a self-contained HTML artifact in plain language
/ce-brainstorm add account-level notification settings and make the artifact a self-contained HTML page

# Equivalent shorthand when a repeatable automation needs it
/ce-brainstorm add account-level notification settings output:html

# Keep the session on your usual model; generate the approaches on a named one
/ce-brainstorm add account-level notification settings, use fable
```

当你还没有方向时用 `ce-ideate`。候选已经明确、需要 verdict 时用 `ce-pov`。产品形状已经确定时用 `ce-plan`。

---

## 问题

从一个模糊想法直接跳到实现，通常会带来：

- 做出来的东西解决了错误的问题，因为没人 pressure-test 最初前提
- Scope creep，因为边界从未被写清
- 每次有人碰到 plan，都要重新争论一遍产品决策
- Requirements 要么变成没人维护的过度仪式化 PRD，要么只有一句话，逼得 planning 靠猜来补全

典型的 AI “一起 brainstorm”也有结构问题。它一次消息问五个问题；你回答两个，剩下的就丢了。它会立刻选一个方案，而不是先展示 alternatives。它把 implementation 混进产品讨论。最终产物只是 conversation，而不是可以 handoff 的 artifact。

## 解决方案

`ce-brainstorm` 会运行一段结构化对话，并可以把它收束成持久 artifact：

- 每个 turn 只问一个问题，默认使用平台的 blocking question tool
- 能由环境回答的事实会被 lookup，而不是问用户；进行中的 lookup 不会阻塞与它无关的问题
- 如果用户术语或对系统行为的陈述，与现有 `CONCEPTS.md` 或已验证代码冲突，而且会改变产品决策，就先挑战该冲突
- 仪式程度与工作规模匹配：Lightweight、Standard、Deep 或 Deep-product
- 生成 approaches 前先用具名 gap lenses 检查前提
- 当你对领域不熟、无法衡量选项时，可 opt-in blindspot pass
- 在你回答开场问题时，后台 grounding scout 同时收集 repo 中的逐字证据
- 给出 2–3 个具体 approaches 及 tradeoffs，再明确推荐一个
- 对于“看粗略 sketch 比读 prose 更容易判断”的决策，可 opt-in visual probes
- 当某个 approach 一旦选定后代价很高，而对话和便宜 sketch 又无法解决时，可选提议 `ce-prototype`
- 文档落地前，用 Synthesis Summary 提供最后一个低成本修正 scope 的机会
- 文档落地前，用 fresh-context verifier 检查其中关于 repo 的 claims
- 每个 artifact 只对应一个连贯 work unit
- Handoff 前运行 Ready for Planning Check，修复 completeness、consistency、focus 和 planning-readiness
- 在统一 plan 中生成大小合适的 Product Contract，并使用稳定 R/A/F/AE identifiers 流入 planning

---

## 它的新颖之处

### 1. 一次只问一个问题

一条消息塞进多个问题，会稀释回答。`ce-brainstorm` 每个 turn 只问一个问题；存在自然选项时，默认使用平台的 blocking question tool 和 single-select。始终允许 free-text。它也只问真正需要你做的决策：如果 repo、grounding dossier 或其他可访问来源能够决定答案，就直接查，而不是把问题丢给你。进行中的 lookup 不会阻塞那些不依赖它的问题。如果你的措辞与现有 `CONCEPTS.md` 或已验证代码冲突，并且这种冲突会改变产品决策，它会在把措辞视为 settled 前先明确指出。它不会创建 `CONCEPTS.md`；glossary 的写入仍在 plan 之后进行。

### 2. 仪式程度随工作规模变化

Lightweight 用于小而边界清晰的想法。Standard 处理普通功能和一些必要决策。Deep 会为 cross-cutting 工作加入更多 probes。Deep-product 还必须建立 product shape（actors、core outcome、positioning、durability），而不是默认继承它。

### 3. 先用具名 gap lenses，再生成 approaches

生成 approaches 前，skill 会扫描开场信息中的 rigor gaps，只探测实际存在的那些：

- Evidence：声称“users want X”，却没有任何可观察行为支持
- Specificity：受益者过于抽象，设计阶段会被迫凭空发明“他们是谁”
- Counterfactual：不知道人们今天怎么做，也不知道什么都不交付会怎样
- Attachment：某个具体 solution shape 已经被默认当作“要做的东西”
- Durability（仅 Deep-product）：价值建立在可能发生变化的现实状态上

这些 probe 用 prose 提问，而不是菜单。四选一菜单会暗示“哪些证据才算数”；prose 会迫使你提供真实观察。

Phase 2 随后会给出 2–3 个具体 approaches，其中至少一个来自非显而易见的角度（inversion、constraint removal 或 cross-domain analogy）。Approaches 保持在 mechanism 或 product-shape 粒度，不上升到 architecture。在 research 很薄时决定 architecture 属于 `ce-plan`。推荐一定放在 approaches 之后，让你先看到 alternatives。

### 4. 先 visual probe；粗 sketch 不够时再 prototype

当决策涉及空间、行为或视觉，skill 可以提议制作一个粗略的本地 visual probe。这些 probe 是用于产品反馈的一次性 sketch，只负责展示；你在聊天中回应。如果某个问题粗 sketch 无法解决（例如 finish 或 motion），或者原本就是为这个问题做的 sketch 最终仍没解决，它会转向 `ce-prototype`。

### 5. Synthesis、identifiers，以及 handoff 前的最后检查

写文档前，skill 会给出 scoping synthesis：要做什么、对话产生了哪些 trade-offs、哪些内容被 defer，以及是否还存在真正的 forks。没有问过 blocking question 的 Lightweight run 会把它压成一句面向未来的话。Standard、Deep，以及任何问过 blocking question 的 run，都会得到完整 synthesis 和显式 confirmation gate；即使开场信息非常充分、不需要对话，也同样如此。

Product Contract 携带 R-IDs（Requirements）、A-IDs（Actors）、F-IDs（Key Flows）和 AE-IDs（Acceptance Examples）。`ce-plan` 会把每个 implementation unit 和 test scenario trace 回这些 IDs。原始 scope boundary——包括“Outside this product's identity”——会原样流入后续阶段。

Requirements 从用户视角描述预期行为。除非 brainstorm 本身就是技术决策，否则不会描述 libraries、schemas、endpoints、file layouts 或 code structure。

在对话中真正审视并由你选择的 decision，会落成带 label 的 Key Decision（`session-settled: user-directed` 或 `user-approved`），之后不会重复询问。`ce-plan` 会继承该 label。

在 Standard 和 Deep 软件路径中，低成本 scout 会在你回答第一个问题时并行收集 grounding dossier（逐字 quotes 加 `file:line` pointers）。写 plan 前，一个从未看过对话的 verifier 会检查 Product Contract 中关于 repo 的 claims。被反驳的 claim 会被修正；无法验证的会变成显式 assumption。Dossier 路径会交给 `ce-plan`。

### 6. Blindspot pass 与非软件 facilitation

当你明确表示自己不熟悉某领域，或连续回答显示你无法衡量选项时，skill 会先提议 blindspot pass，再继续追问该领域：列出 3–7 个 decisions 和 hazards，每项说明为什么重要、现实可选项，以及推荐默认值。你选择哪些值得逐项讨论；其余使用默认值，并作为显式 assumptions 记录。软件和非软件路径都支持这一机制。

非软件工作使用 domain-agnostic facilitator，同时保持“一次一个问题”的纪律。它不会写软件 unified-plan artifact。

---

## 快速示例

你从“I want to add a way for users to pause notifications.”开始。Skill 把工作判断为 Standard，并在你回答第一个问题时派一个低成本后台 scout 去收集 repo evidence。

Pressure test 发现 specificity gap（这些“users”到底是谁？）和 attachment gap（“pause”本身已经是一种 solution shape）。它用 prose 一次一个地追问。你说清真正痛点（support 会在凌晨 3 点因为非紧急事件收到通知），以及能够解决问题的最小版本。

随后出现三个 approaches：按 notification type 设置带 TTL 的 mute、全局 do-not-disturb schedule、把 mute 放在 rule 而不是 channel 上。接着给出 tradeoffs 和 recommendation。Synthesis Summary 会复述最终 shape（“per-channel mute on notification rules，针对凌晨 3 点 support 通知提供 24h preset”）、trade-offs（per-channel 而非 per-user；mute 存在 rule 上）、deferred 内容（presence-based mute、quiet-hours schedules），以及 rule-delete loss path 的 call-out。你确认，并补充 24h preset。

一份 requirements-only unified plan 会写到 `docs/plans/`。Phase 4 menu 随后提供：用 `ce-plan` 创建 implementation plan（推荐）、用 `lfg` 自主交付、pressure-test requirements 或 prototype 尚未解决的 feel-question、如果文件是 HTML 就打开它，或继续提 clarifying questions。

---

## 什么时候该用它

适合使用 `ce-brainstorm` 的情况：

- 功能想法已经形成一部分，但你还无法勾勒实现
- 一个请求存在多个有效 solution，需要做选择
- Scope 不清楚（“add notifications”：哪一种、给谁、什么时候）
- 你需要一份结构化 artifact，交给其他人或 planning
- 你必须在自己不熟悉的领域里划 scope（blindspot pass 会先画出 decision surface）
- 主题不是软件（命名、活动、roadmap choices）

以下情况跳过 `ce-brainstorm`：

- 你还不知道该做什么 → `/ce-ideate`
- Requirements 已经明确（已有 PRD、issue 足够详细）→ `/ce-plan`
- 请求是“是否采用某个已命名外部候选” → `/ce-pov`
- Bug 已有已知 root cause → `/ce-debug`
- 改动 trivial 且 obvious → 直接做

---

## 作为链式工作流的一部分

```text
/ce-ideate          (optional: discover candidate directions)
   |  picks one survivor and carries its basis, rationale, and tradeoffs
   v
/ce-brainstorm
   |  produces a requirements-only unified plan
   |  software: R-IDs, A-IDs, F-IDs, AE-IDs and scope boundaries
   |  non-software: chat synthesis, optional handoff to ce-plan
   v
/ce-plan
   |  enriches the same plan to implementation-ready
   |  R-IDs flow into Requirements; A/F/AE-IDs trace into units and tests
   |  origin scope boundaries are preserved
   v
/ce-work
```

当 `ce-plan` 加载一份 requirements-only unified plan 时，它不会重新争论 product behavior。Product Contract 是权威来源。Plan 阶段的决策关注 execution guardrails，而不是重新决定要做什么。

在 repo 中，如果要继续处理一个 ideate survivor，总是先来这里，而不是直接去 `ce-plan`。`ce-plan` 需要一份经过 brainstorm grounding 的 Product Contract。

---

## 独立使用

很多团队会跳过 `ce-ideate`（因为已经知道要探索什么）。有些团队也会停在这里，把 brainstorm 当作 thinking artifact，之后再 plan。

- Feature briefs：把模糊想法变成 stakeholders 或新贡献者可以依赖的稳定 artifact
- Onboarding existing work：功能已经在开发，但 rationale 从未写下来
- Pre-PR alignment：写代码前，多个人需要先对 scope 达成一致
- Strategic decisions：Deep-product 会暴露 durability 和 adjacent-product risks
- Non-software：给产品命名、规划活动、决定 roadmap

软件路径的 Phase 4 menu 会提供 planning、用 `lfg` 自主交付（当 unified plan 已存在且没有 blockers）、document review 或 prototype、HTML browser 打开选项，或继续提问。这里不会提供跳过 planning 直接进入 `ce-work` 的选项。非软件 wrap-up 会提供 `ce-plan`、保存 summary、发布到 Proof，或停止。

如果已经存在相关 requirements-only plan，skill 会提议 resume，而不是再创建一份重复文档。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | 询问你想探索什么 |
| `<feature idea>` | 开放式 brainstorm |
| `<problem>` | 经过 product pressure test 路径 |
| 已有 requirements-only plan 路径、旧版 `*-requirements.md` 路径或匹配 topic | 提供 resume |
| 当前对话中已经出现的 ideate survivor | 连同该 idea 的 tagged basis、rationale 和 tradeoffs 一起加载 |
| Verdict-shaped prompt（`should we adopt X`） | 提议 `ce-pov`；拒绝后继续 brainstorm |
| `output:html` | 把 requirements-only unified plan 写成单个 self-contained HTML 文件，而不是 Markdown。二选一：artifact 要么是 `.md`，要么是 `.html`，不会同时生成。默认 Markdown。可以在 CE config 中设置 `brainstorm_output: html`（先 `config.local.yaml`，再 `config.yaml`）让 HTML 成为默认值。Pipeline mode（LFG、`disable-model-invocation`）始终强制 Markdown。详见[配置参考](./configuration.md)。 |
| `use fable` / `have opus generate these` | 只把 approach generation 提升到指定 model。也可以在 CE config 中设置 `brainstorm_model: <model>`。Prompt 请求优先于 config key。 |

---

## FAQ

**为什么一次只问一个问题？不会很慢吗？**
每个 turn 堆三个问题会稀释回答。人们会挑最容易的答，其他就丢了。一次一个问题能得到更清晰的答案，而且通常更快收敛。

**为什么要 pressure-test 我的前提？我只是想 brainstorm。**
具名 gap lenses 用来捕获 feature brief 在下游最常见的失败方式。只有实际存在 gap 时才会触发。一个具体且结构良好的 prompt 完全可能得到零 probes。

**可以跳过 requirements-only plan 吗？**
可以。Lightweight tier 和 announce-mode fast path 都支持。如果你只需要简短 alignment，就不会写文档。没有 artifact 时，`lfg` menu option 会隐藏，因为 `lfg` 无法为缺失文件向用户提问。

**如果我已经有 PRD 或详细 GitHub issue 呢？**
跳过 `ce-brainstorm`，直接去 `/ce-plan`。Plan skill 可以消费任何类型的输入。

**Synthesis 里的“Inferred”是什么意思？**
Agent 在展示 scoping synthesis 前，会先在内部生成三个 bucket（Stated / Inferred / Out of scope）。Inferred items 是用于填补对话空白的 bets。通过 keep test 的会作为 call-out 显示；其他内容在你确认后融入 Product Contract。

**它适用于非软件主题吗？**
适用。Domain-agnostic facilitator 会保持“一次一个问题”的纪律。Wrap-up 可以把 synthesis handoff 给 `ce-plan`、保存 summary，或发布到 Proof。它不会写软件 unified-plan artifact。

**可以从这里直接进入 `ce-work` 吗？**
Phase 4 menu 不提供该选项。软件路径下一步是 `ce-plan`，或 `lfg`（它会先 planning）。即使是 Lightweight scope，这里也不会提供 skip-to-build。

---

## Model elevation

如果你希望 heavy reasoning step 使用特定 model，`ce-brainstorm` 可以在该 model 上生成 approaches，而不是用 session model。只有 approach generation 会被 dispatch，并带 read access 以便验证 brief；skill 的其余部分仍运行在你的 session model 上。可以在 prompt 中指定 model（`use fable`、`have opus generate these`），或在 CE config 中设置 `brainstorm_model: <model>`（先 `config.local.yaml`，再 `config.yaml`）。Prompt 请求优先于 config key。

这在任意 harness 上都可用。宿主能原生提供所选 model 时直接使用；否则会调用 Claude CLI（必须已安装并完成认证）；再不行就用 session model 执行该步骤，并说明缺少了哪个 precondition。因此 `brainstorm_model` 会在你运行 `ce-brainstorm` 的所有 harness 上生效，而不只是在 Claude Code 中。

---

## 另请参阅

- [`ce-ideate`](./ce-ideate.md)：上游“what's worth exploring”发现流程；survivors 会连同 tagged basis 一起进入这里
- [`ce-pov`](./ce-pov.md)：针对已命名外部候选做明确 verdict，而不是创建新 scope
- [`ce-plan`](./ce-plan.md)：把 requirements-only unified plan 丰富为 implementation-ready plan
- [`ce-doc-review`](./ce-doc-review.md)：对 Markdown 或 HTML Product Contract 做 persona-based review
- [`ce-prototype`](./ce-prototype.md)：在 commit 某个 approach 前，决定它应该如何工作或呈现
- [`ce-strategy`](./ce-strategy.md)：让 brainstorm 以已记录 product strategy 为锚点
- [`lfg`](./lfg.md)：从 requirements-only artifact 开始，自主完成 plan-then-ship
- [`ce-proof`](./ce-proof.md)：发布非软件 summary（或任何你要求分享的 Markdown 文件）