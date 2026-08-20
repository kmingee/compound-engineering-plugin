# `ce-plan`

> 建立实现所需的 guardrails（decisions、units、files、tests、scope、risks），但不预先规定实际代码或逐步 choreography。Plan 记录 WHAT；真正实现的 agent 决定 HOW。

`ce-plan` 生成的是**带执行 guardrails 的决策文档**，不是 implementation choreography。Plan 会记录：哪些 decision 已经做出、scope 哪些在内哪些在外、有哪些 atomic units of work、每个 unit 涉及哪些 files、必须通过哪些 test scenarios，以及哪些 risks 需要 mitigation。它不会预写代码、精确 API signatures 或逐步 shell sequences。这些都应该留给真正实现的人——`ce-work`、其他 AI agent 或 human——在面对实际代码时决定。

预先把实现写进 plan，到了真正 implementation 时往往已经错了：signature 编译不过、choreography 过时、micro-steps 把真正的 decision 淹没。记录 guardrails 的 plan 则能在数周甚至数月后仍保持可移植性，把判断留给 implementer。

任何多步骤、且结构能带来帮助的任务都可以使用它：软件功能、refactor、bug fix、study plan、research workflow、event planning，甚至每年的 hot-water-tank maintenance。使用同一套 engine、同一套稳定 U-ID、同一套 right-sized template。

这是 compound-engineering ideation 链中的第三步：

```text
/ce-ideate         /ce-brainstorm      /ce-plan             /ce-work
"What's worth      "What does this     "What's needed       "Build it."
 exploring?"        need to be?"        to accomplish
                                        this?"
```

已有 brainstorm 会提供有用上下文，但从不是硬性要求。很多团队会直接把 requirements-only unified plan、旧版 requirements doc、GitHub issue、PRD、粗略描述，或非软件多步骤任务交给 `ce-plan`。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | Research 上下文、记录 decisions 与 scope、把工作拆成带稳定 ID 的 atomic units、为每个 unit 枚举 test scenarios，然后通过 confidence check 自动加强薄弱部分 |
| 什么时候用？ | Requirements 已就绪，需要 execution guardrails；任务已经清楚时独立 planning；非软件多步骤任务；需要结构化答案的 investigative questions |
| 会产出什么？ | 软件：在 `docs/plans/YYYY-MM-DD-HHMM-<type>-<name>-plan.md` 写统一 plan（使用本地 wall-clock 写入时间，发生冲突时以数字 suffix 原子保留）。来自 brainstorm 的 plan 会在原文件中从 `artifact_readiness: requirements-only` 升级到 `implementation-ready`。非软件 plan-seeking 会写 domain plan（或发布到 Proof）。Answer-seeking 直接在聊天中给答案，不写 plan file。 |
| 下一步是什么？ | 软件：启动 `ce-work`（推荐）、宿主支持时作为 `/goal` 运行、处理剩余 review items 或 prototype 尚未解决的 feel-question、创建 tracked issue，或在 browser 中打开 HTML plan。非软件：保存、发布到 Proof，或两者都做。Answer-seeking：答案本身就是终点。 |

---

## 调用示例

空调用如果当前 conversation 已经有任务（包括刚完成的 brainstorm），就直接使用；否则会询问要 plan 什么。传入 requirements-only plan 路径会原地丰富该文件。`output:html` 改变 artifact 格式。`confirm:auto` 只跳过 pre-plan scope confirmation。

```text
# Use this conversation if it already has a task; otherwise ask what to plan
/ce-plan

# Enrich a requirements-only brainstorm artifact into an implementation-ready plan
/ce-plan docs/plans/notification-mute.md

# Plan directly from an issue or PRD
/ce-plan https://github.com/acme/widgets/issues/1234
/ce-plan docs/product/account-notifications-prd.md

# Bootstrap planning from a clear rough idea
/ce-plan add a background email digest at 8am UTC

# Revisit and deepen an existing implementation-ready plan (interactive accept/reject)
/ce-plan deepen docs/plans/auth-rewrite.md

# Plan a non-software multi-step project (save and/or publish to Proof)
/ce-plan organize a two-day customer advisory workshop

# Answer-seeking: state a plan-of-attack in chat, then deliver the answer (no plan file)
/ce-plan how often does this customer star our repos, and is that a real signal?

# Hold at an approach-plan before committing to the deliverable
/ce-plan plan for a plan: synthesize the three research PDFs into a decision memo

# Write the plan as a self-contained HTML page
/ce-plan turn the notification mute requirements into an implementation-ready plan and make it a self-contained HTML page

# Equivalent shorthand when a repeatable automation needs it
/ce-plan turn the notification mute requirements into an implementation-ready plan output:html

# Skip the pre-plan scoping-confirmation pause for this run only
/ce-plan add a background email digest at 8am UTC confirm:auto

# Keep the session on your usual model; author the plan on a named one
/ce-plan turn the notification mute requirements into an implementation-ready plan, use fable
```

当 product shape 仍未 settled 时，先从 `ce-brainstorm` 开始。当 intended outcome 已经清楚时，直接 planning 最合适。

---

## 问题

人写的 plan（或者没有结构约束的 AI plan）会以一些可预测方式失败：

- Renumbering chaos：重排 unit list 后，issue、PR 和 conversation 中所有引用都错了
- 模糊的 test “scenarios”：“test the new behavior” 对 implementer 没有任何帮助
- 忘记 origin context：brainstorm 已经决定功能是给某个特定 actor，但 plan 完全没提
- 半解决问题：“TBD: figure out caching strategy” 几个月后仍躺在 plan 里
- Implementation choreography：预先写死 exact method signatures、micro-steps 或 shell sequences，真正 implementation 时已经不对
- 没有 depth check：author 不知道 plan 是否足够 grounded，可以执行

## 解决方案

`ce-plan` 把**需要被遵守的 decisions**与**如何在代码中实现它们**分开：

- Plan 记录 decisions、scope boundaries、atomic units、files、test scenarios 和 risks
- 不预写代码、精确 API signatures 或逐步 shell choreography
- 稳定 U-IDs 能在重排、拆分、删除后继续存在，因此 blocker references 和 PR mentions 不会失效
- Plan decisions 可以 trace 回 origin（来自 brainstorm 的 R-IDs；test scenarios 引用 AE-IDs）
- Structuring 前并行 research（repo、learnings、framework docs、best practices、spec flow）
- 写完 plan 后自动运行 confidence check，dispatch 有针对性的 sub-agents 去加强薄弱 section
- 区分 planning-time 与 implementation-time questions，不制造虚假的确定性

---

## 它的新颖之处

### Guardrails 优先于 choreography

Plan 记录 decisions 与 constraints，而不是代码：已做出的 decisions（含 rationale）、scope boundaries、atomic units of work、涉及 files、必须通过的 test scenarios，以及需要 mitigation 的 risks。它排除 exact method signatures、framework-specific syntax、逐步 shell sequences，以及伪装成 implementation spec 的 pseudo-code。Implementing agent 读取这些 guardrails，然后在面对实际代码时决定 HOW。

这也是同一套 engine 能处理非软件任务的原因。Hot-water-tank-maintenance plan 同样有 decisions、units、files-equivalent（哪些 valves、哪些 manuals）、test scenarios（“verify no leaks after refill”）和 risks，只是没有代码。

### 永不重编号的 U-IDs

每个 unit heading 都是 `### U1. Name`、`### U2. Name`，依此类推。已有 ID 在 reordering、splitting 或 deleting 后永远不会重新编号。拆分时，原 concept 保留原 U-ID；新 unit 使用下一个未使用 number；删除后留下 gap。

`ce-work` 会跨 plan edits 通过 U-ID 引用 units。Deepening pass 中如果重编号，会静默破坏每一个 blocker reference、每个引用 unit 的 PR，以及所有 downstream conversation。

### Origin tracing 与 per-unit tests

当 plan 来源是 `ce-brainstorm` 的 requirements-only unified plan 时，identifiers 会在同一文件中继续流动。Requirements（R-IDs）留在 Product Contract。Actors（A-IDs）在影响行为或权限时继续携带。Key Flows（F-IDs）会被引用到真正实现它们的 units。Acceptance Examples（AE-IDs）会被引用到 test scenarios 中（`Covers AE3. <scenario>`）。Finalization 前，每个 Product Contract section 都会与 Planning Contract 对照检查。

每个承载 feature 的 unit 都会从适用类别中枚举 test scenarios：happy path、edge cases（boundaries、empty/nil、concurrency）、error/failure paths 和 integration。每个 scenario 都要写明 input、action 和 expected outcome。

### Confidence check，然后按 intent 匹配 research

Plan 写完后，`ce-plan` 会给 sections 打分，挑出最薄弱的部分，dispatch 有针对性的 sub-agents（units 用 correctness、migrations 用 data integrity、关键 technical decisions 用 architecture），再把 findings synthesis 回 plan。Auto mode（生成过程中默认）直接集成 findings。Interactive mode（当你要求 deepen 现有 plan 时）逐条展示 findings，由你 accept/reject。

Phase 1 始终并行运行本地 research（repo patterns 与 `docs/solutions/` learnings）；Standard/Deep plans 还会运行 spec-flow analysis，并可选 Slack research。External research 由 intent 决定，而不是一个简单 on/off switch。显式请求（“research competitors”“best practices from the web”“which library”）一定会运行。隐式 signal（本地 patterns 很薄，或 recommendations 依赖尚未 settled 的外部 option set）也可能触发。Implementation-guidance 会路由到 framework docs 和 best practices。Landscape 或 option-discovery 会走 web scan。Mixed request 先跑 landscape scan，再针对 shortlist 查 docs。

### Universal planning 与 approach altitude

非软件工作会跳过 software confidence check，但 U-IDs、dependency ordering、scope boundaries、verification scenarios 和 right-sized template 会继续使用。

有两种 disposition：

- Plan-seeking（旅行、study curriculum、活动）：saved plan 就是 deliverable。写完后，wrap-up 会提供保存到 disk、发布到 Proof，或两者都做。
- Answer-seeking（调查或分析，例如“how often does X happen, is it a big deal?”）：答案就是 deliverable。Skill 会先在聊天中陈述简短 plan-of-attack，然后执行（research 与 synthesis，绝不写代码），且不会写 plan file。只有真正单一事实 lookup 会跳过 planning，直接回答。

对于难题，你可以要求上一个层级：先生成有 grounding 的 **approach-plan**（也就是“如何产出 deliverable”的 plan），并停在 checkpoint。可以显式进入（`plan for a plan`、`don't write it yet, plan how you'd approach it`）。极少数情况下，当方法本身真正 unsettled、选错代价很高时，skill 也会主动提议。经过 light recon 后，它会在聊天中铺开 approach；是否写文件可选，也可以继续 deepen。你可以现在执行，也可以留到以后。代码仍交给 `ce-work`。非代码 deliverable 会标记为 `execution: knowledge-work`，走 `ce-work` 的 lightweight carve-out。`ce-plan` 自身从不执行。

### Session-settled decisions 会被携带，而不是重问

如果某个 decision 已在 invoking conversation 中经过审视并由用户选择，或者由 caller brief 提炼而来，`ce-plan` 会在对应 Key Technical Decision 上记录 `session-settled: user-directed` 或 `user-approved`，说明它是相对于什么替代项做出的选择，并且绝不重新询问。Research 只有基于证据才能反驳 settled decision：什么也没发现就静默继续；suboptimal-but-workable 时继续但加 conflict call-out；如果证据足以 invalidate（不可行、解决了错误问题、destructive），就停止 run。Pipeline mode 下会返回 `settled-decision-invalidated` blocked report。未经审视的 assertion 不算 settled，只会在 plan 阶段接受一次 challenge。

---

## 快速示例

你拿一份来自 `ce-brainstorm` 的 requirements-only unified plan 调用 `ce-plan`。Skill 检测到 `artifact_readiness: requirements-only`，把 Product Contract 作为 primary input，并验证没有 resolve-before-planning blockers。

它会并行 dispatch research（repo analyst、learnings researcher）。本地 patterns 很强，而且没有要求 external comparison，因此跳过 external research。如果你显式说“research competitors”或“best practices from the web”，则会覆盖默认判断。Spec-flow analyzer 会运行，用于暴露 edge cases。来自 brainstorm 的 scoping synthesis 会给出 tier-shaped summary，并附零个或多个 call-outs（也就是另一个合理 agent 可能做出不同选择的 plan-time forks）。你可以 confirm 或 redirect。只有没有值得提示 fork 的 Lightweight plan 才会 auto-proceed。Standard 和 Deep 始终保留显式 checkpoint。

Plan 写入后，confidence check 自动运行。它发现 `Risks & Dependencies` 对 mute-leak risk 描述过薄，而且某个 unit 的 tests 漏了 permission edge cases，于是 dispatch reviewers，并把 findings synthesis 回 plan。Plan 会盖上 `deepened:` 日期。

随后 document review 会以 non-interactive mode 运行在 Markdown 或 HTML plan 上。安全的 auto-fixes 会直接在 artifact 原生格式中静默应用。剩余 findings 会在 post-generation menu 上方以一行 summary 显示（`Doc review applied 2 fixes. 3 decisions, 1 FYI remain.`）。Menu 提供：启动 `ce-work`（推荐）、宿主支持时作为 `/goal` 运行、处理剩余 review items 或 prototype 一个尚未解决的 feel-question、创建 tracked issue，或在文件是 HTML 时打开它。软件 menu 没有 Proof，也没有 pause。文件已经保存。

---

## 什么时候该用它

适合使用 `ce-plan` 的情况：

- 已有来自 `ce-brainstorm` 的 requirements-only unified plan
- 已有 GitHub issue、PRD 或足够清楚的 feature description
- 工作是多步骤，并且 sequencing、dependency ordering 和 scope boundaries 能带来帮助
- 希望在 execution 前先枚举 test/verification scenarios
- 正在接手 stale plan，并想 deepen（`deepen the plan` 或 `deepening pass`）
- 任务是非软件但多步骤（study plan、event、trip、maintenance、research workflow）
- 问题是 investigative，希望获得结构化答案而不是 plan file

以下情况跳过 `ce-plan`：

- 任务真正只有一步（直接做，或使用 `ce-work` 直接执行）
- Product/outcome 还没有决定 → 先 `ce-brainstorm`
- Bug 已有已知 root cause，而且修复 obvious → `ce-debug` 或直接修

---

## 作为链式工作流的一部分

```text
/ce-ideate          (optional)
   |
   v
/ce-brainstorm      (define one direction)
   |  requirements-only unified plan: R/A/F/AE-IDs in software mode
   v
/ce-plan
   |  guardrails: U-IDs traced to R/A/F/AE-IDs
   |  test scenarios with AE-link convention (Covers AE<N>)
   |  scope boundaries preserved (including "Outside this product's identity")
   |  confidence-checked and auto-deepened
   v
/ce-work            (execute against the guardrails)
   |  reads U-IDs as the unit of execution
   |  figures out the actual HOW with code in front of it
   |  derives progress from git, not the plan body
   v
/ce-code-review     (optional)
   |
   v
/ce-compound        (capture the learning)
```

`ce-plan` 到 `ce-work` 的 handoff 很具体：`ce-work` 读取 U-IDs、file paths、scope boundaries 和 test scenarios，然后决定真正 implementation。Plan 告诉 implementer：unit 完成时**必须满足什么**。Implementer 决定**如何让它成立**。

---

## 独立使用

很多人在已经知道该做什么时会直接使用 `ce-plan`，无论是软件还是非软件多步骤任务。

**软件：**

- 从 GitHub issue：`/ce-plan https://github.com/.../issues/1234`（或粘贴 issue body）
- 从 PRD：`/ce-plan` 加 PRD path
- 从 rough idea：`/ce-plan "add background email digest at 8am UTC"` 会运行 bootstrap；synthesis 让你在 research dispatch 前纠正 scope
- 重新 deepen 现有 plan：`/ce-plan deepen the auth-rewrite plan`（interactive accept/reject）
- Cross-repo planning：在另一个 repo 中执行 `/ce-plan "fix the busyblock bug in cli-printing-press"`。目标会先被 announce，plan 会写到目标 repo 的 `docs/plans/`

**非软件（universal-planning mode）：**

- Maintenance tasks，每个 unit 都带 verification
- Study plans，包含 prerequisites 和 per-unit knowledge checks
- Trip planning：bookings、packing、daily itinerary、contingency boundaries
- Research workflows：gathering、synthesis、drafting，并带显式 deliverables
- Event planning：venue、vendors、agenda、day-of run-of-show
- Personal projects：workshop build-outs、home renovations
- Answer-seeking questions：直接在聊天中交付答案，不写 plan file

在 universal-planning mode 中，U-IDs、dependency ordering、scope boundaries 和 right-sized template 都会保留。Software-specific confidence check 会跳过。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | 当前 conversation 已有 task 时直接使用，否则询问要 plan 什么 |
| `<feature description>` | Solo planning；运行 bootstrap |
| `<requirements-only plan path>` | 在原地丰富同一份 unified plan |
| `<legacy requirements doc path>` | 以 origin 为来源，在新 unified plan 中 planning |
| `<plan path>` | 提供 resume（intent 匹配时则 deepen） |
| `deepen the plan` / `deepening pass` | Re-deepen fast path（interactive mode） |
| `plan for a plan` / `don't write it yet` | Approach-altitude：生成 approach-plan，并停在 checkpoint |
| `<investigative question>` | Answer-seeking：先在聊天中给 plan-of-attack，再给答案；不写 plan file |
| `<bug description>` | 路由到 `ce-debug` suggestion menu（pipeline mode 中跳过） |
| `<task in another repo>` | Cross-repo announcement；plan 写到目标 repo |
| `output:html` | 把 plan 写成单个 self-contained HTML 文件，而不是 Markdown。二选一：plan 要么 `.md`，要么 `.html`，不会两者同时生成。默认 Markdown。可以在 CE config 中设置 `plan_output: html`（先 `config.local.yaml`，再 `config.yaml`）让 HTML 成为默认。Pipeline mode（LFG、`disable-model-invocation`）始终强制 Markdown。详见[配置参考](./configuration.md)。 |
| `confirm:auto` | 只针对本次运行跳过 pre-plan scoping-confirmation pause。Skill 会为自己写 scope summary，把 inferred scope 记录到 `Assumptions`，announce 正在继续，然后直接执行。真正 blocker 和 post-plan menu 仍会出现。用 `confirm:ask` 可只针对一次运行强制恢复 gate。设置 CE config 的 `plan_skip_scoping_confirm: true` 可以让跳过成为默认。 |
| `use fable` / `have opus plan this` | 只把 interpret-findings-then-author step 提升到该 model。也可在 CE config 设置 `plan_model: <model>`。Prompt 请求优先于 config key。 |

---

## FAQ

**Plan 不就是告诉你 HOW 吗？**
在 `ce-plan` 的定义里不是。Plan 告诉你必须遵守什么：decisions、scope、units、files、tests、risks。它不会预写 code、exact API signatures 或逐步 shell choreography。Implementing agent 会在代码就在眼前时决定 HOW。也正因为这样，同一套 engine 才能同时 plan 软件 refactor、tank-maintenance job 和 6-week study plan。

**为什么用 U-IDs，而不是普通编号？**
普通编号在 units reorder、split、delete 后会失效。U-IDs 保持不变。拆分时原 concept 保留原 ID；删除留下 gap。正因为如此，`ce-work` 的 blocker references 才能跨 plan edits 继续有效。

**为什么 confidence check 要自动运行？**
发现薄弱 section 最昂贵的时机是在 execution，而不是 planning。Auto-deepening 会在 research context 仍然“热”的时候执行。

**如果我想保留现有 plan，只做 review 呢？**
使用 deepen-intent fast path：`/ce-plan deepen <plan>`。它会以 interactive mode 运行，agents 逐条展示 findings，由你 accept/reject。

**Plan 里能不能放 implementation code？**
默认不允许。Pseudo-code 和 DSL grammars 可以出现在 High-Level Technical Design 中，用来表达 solution shape，但必须被 framing 成 directional guidance，而不是 implementation specification。Exact method signatures、imports、framework-specific syntax 和逐步 shell sequences 不属于 plan。

**它真的适合非软件 plan 吗？**
适合。Universal-planning 会保留 U-IDs、dependency ordering、right-sized template，以及 guardrails-not-choreography 框架。真实用例包括 tank maintenance、study plans、trip planning、research workflows 和 event planning。Investigative questions 使用同一套 engine，但直接在聊天中交付答案。

**可以从 post-plan menu 把软件 plan 发布到 Proof 吗？**
不可以。Proof 位于非软件 wrap-up menu（save、publish 或 both）。软件下一步是 `ce-work`、宿主支持时 `/goal`、review 或 prototype、创建 issue，或打开 HTML file。若之后需要可分享链接，可以手动用 `/ce-proof` 发布 Markdown plan。

---

## Model elevation

如果你希望 heavy reasoning step 使用特定 model，`ce-plan` 可以在该 model 上 author plan，而不是 session model。只有 interpret-findings-then-author step 会被 dispatch，并带 read access 以验证 brief。Dialogue 和 research 仍在 session model 上。可以在 prompt 中指定 model（`use fable`、`have opus plan this`），或在 CE config 中设置 `plan_model: <model>`（先 `config.local.yaml`，再 `config.yaml`）。Prompt 请求优先于 config key。

这在任何 harness 上都有效。宿主能原生提供目标 model 时直接使用；否则调用 Claude CLI（必须已安装并认证）；再不行就使用 session model 运行该步骤，并说明缺少的 precondition。因此 `plan_model` 会在所有运行 `ce-plan` 的 harness 上生效，不只是在 Claude Code。

## 另请参阅

- [`ce-brainstorm`](./ce-brainstorm.md)：生成 `ce-plan` 会继续丰富的 requirements-only unified plan
- [`ce-ideate`](./ce-ideate.md)：上游“到底值得做什么”的 ideation
- [`ce-work`](./ce-work.md)：按 U-ID 逐个执行 plan
- [`ce-doc-review`](./ce-doc-review.md)：对 Markdown 或 HTML plan 做 persona-based review
- [`ce-prototype`](./ce-prototype.md)：当 post-plan menu 中仍有昂贵、难以撤销的 feel-question 时提供
- [`ce-debug`](./ce-debug.md)：bug-shaped prompt 会路由到这里
- [`ce-strategy`](./ce-strategy.md)：让 plan 锚定在已记录的 product strategy 上
- [`ce-proof`](./ce-proof.md)：发布非软件 plan，或你要求分享的任何 Markdown plan