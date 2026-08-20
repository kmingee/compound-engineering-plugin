# `lfg`

> 从 planning 一路免值守运行到 open PR。它会直接 push 并打开 PR，不会停下来等批准。它不会 merge。

`lfg` 是**自主 pipeline**。它把 Compound Engineering 的主要工作流串成一次长运行：plan、implement、simplify、review、应用符合条件的 review fixes、运行 browser tests、commit、push、打开 PR，然后监看 CI，并在有边界的 loop 中修复失败。

当你希望 agent 把一个软件 task 从描述（或 requirements-only plan）一路带到 open PR，而且不需要逐阶段检查时使用。它不适合 in-the-loop 工作。如果你想自己 approve plan、diff 或 review findings，请逐个运行对应 skills。

最好在 `/ce-brainstorm` 后使用，因为这样 pipeline 能基于真实 requirements planning，而不是围绕一句话 prompt。软件 brainstorm wrap-up 在已经存在 unified plan artifact，且 `Resolve Before Planning` 没有 blocker 时，会提供“Ship it autonomously with `lfg`”。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | Plan、implement、simplify、review、应用符合条件的 fixes、运行 browser tests、commit、push、打开 PR，并监看 CI |
| 什么时候用？ | 想免值守交付的软件 task；最好已经由 `/ce-brainstorm` 定形，或至少足够清楚可以交给 `/ce-plan` |
| 会产出什么？ | Code changes、commits，通常还有 PR。未解决 review/CI leftovers 会变成持久 notes。没有 remote：只创建本地 commits。 |
| 下一步是什么？ | Review PR。运行 `/ce-babysit-pr` 继续监看到 review/merge 阶段。可选 `/ce-explain` 解释新概念，或 `/ce-compound` 捕获可复用 learning。 |
| 它不会做什么？ | Merge PR、跳过 planning、执行非软件工作；除非你接受 closeout handoff offer，否则不会继续下一个 area |

---

## 调用示例

常见路径是先 brainstorm，再空调用 `/lfg`。传 plan path 会先 enrich artifact，再交付。Stage assignment 只改变谁来 author planning/implementation；pipeline 其他部分仍由 `lfg` 负责。

```text
# Most common: settle requirements, then ship from that context
/ce-brainstorm design account-level notification controls for enterprise teams
/lfg

# Same handoff, but author the plan on a named model (implementation stays native)
/ce-brainstorm design account-level notification controls for enterprise teams
/lfg plan with fable

# After brainstorm already settled the requirements. No feature text needed.
/lfg plan with fable

# Clear, already-bounded software task. Weaker product context than a brainstorm.
/lfg add a CSV export button to the account reports page

# Enrich a requirements-only plan in place, then ship it
/lfg docs/plans/feedback-sweep-plan.md

# Preference: try Codex for implementation, fall back to native if that route is down
/lfg add account-level notification mute settings, use Codex for implementation

# Requirement: only Composer may implement. If that route is unavailable, lfg stops.
/lfg implement the settled plan, but only use Composer for implementation

# Both stages, each to its own model or harness
/lfg add account-level notification mute settings, plan with fable and use Codex for implementation
```

没有 scope 的“use fable”或“with Codex”只绑定到**implementation**，而且 `lfg` 会在开场第一行说明。把 harness 指派给 planning（如“Plan with Codex”）不受支持，会 block。需要自己 inspect/approve 每个 stage 时，请用独立 skills。

---

## 问题

标准 CE workflow 刻意分阶段：plan、work、simplify、review、ship。想逐步检查时很好；但当 task 边界明确、你想让 agent 整条带完时，handoff 太多。

没有显式 pipeline 时，自主 run 很容易跳过 planning、把 review 当可选、忘记持久化 leftover findings，或在“PR opened”时就停下，哪怕 CI 还是红的。

## 解决方案

`lfg` 把 sequence 明确写成带 gates 的流程：

1. 从 conversation 中整理一份简短 settled-decisions brief（每个 decision、class、被拒绝 alternative 和理由），只限定当前 feature；把它交给 `/ce-plan`，避免这些选择被重新询问。没有 settled 内容时跳过 brief。
2. `/ce-plan` 必须先产出 implementation-ready 的**代码** plan 才能进入 work。Requirements-only plan、knowledge-work plan 或非软件结果都会停止 pipeline。
3. `/ce-work` 以 return-to-caller mode 运行，让 `lfg` 保留 shipping tail。Behavior-changing work 必须返回 verification evidence。Evidence 缺失只 retry 一次，然后停止，绝不 blind ship。
4. Review 前在 branch diff 上运行 `/ce-simplify-code`；docs-only 或大约少于 10 行的改动可跳过。
5. `/ce-code-review`（`mode:agent`）只报告 findings。`lfg` 应用符合条件的 mechanical fixes 并 commit。Review 本身不编辑 tree。
6. 剩余 actionable findings 加上 flagged settlement conflicts，会持久化为 tracker tickets，并在 PR 上留下一个 run-report comment；不会写进 PR body。
7. `/ce-test-browser` 以 pipeline mode 运行。
8. `/ce-commit-push-pr mode:pipeline branding:on` 提交剩余变化、push，并在存在 remote 时打开 PR，同时记录 CE provenance。如果 project instructions 指定了自己的 shipping process（例如 `/create-pr` skill），则运行项目流程，因此 CE branding 可能不出现。
9. `/ce-babysit-pr mode:pipeline` 监看 open PR：CI failure 通过 `/ce-debug` 修复，incoming review comments 通过 `/ce-resolve-pr-feedback` 处理，默认最多三轮 fix。Pipeline babysit 停在“CI decided”，不是“merged”。
10. 打印 `DONE`。如果 plan 明确描述了更大的、需要单独 planning 的工作体，并且还有未规划 area，`lfg` 可以 offer 一个 opt-in `/ce-handoff`，交给 fresh session。它自己不会继续那个 area。

Planning 或 review 中出现足以 invalidate 的 settlement conflict，会在 shipping 前停止 pipeline。不导致 halt 的 flagged conflicts 会变成 residual，并进入 PR 的 settled-decisions line。

没有 git remote：只在本地 commit，跳过 push、PR creation 和 CI watch。这是正常 terminal local-only path，不是需要 retry 的错误。

`lfg` 自己永远不会启动 `/goal`。如果 goal-mode 是合适 engine，由 `ce-work` 选择，而且最终仍必须 return control。

---

## 它的新颖之处

### Hard gates，然后只有一个 shipping tail

Planning 必须产出 implementation-ready code plan。Behavior change 的 implementation 必须返回 evidence。Review 刻意设计为只报告。`lfg` 应用符合条件的 fixes，把不应用的内容持久化，然后独占唯一 push/PR/CI tail。任何 stage 都不能跳级直接 coding。

### 可以路由两个 stage，而不是整条 run

Planning 可以通过 `ce-plan` model elevation 交给 named model（`plan with fable`）author。Implementation 可以发送到 harness（`use Codex for implementation`、`only use Composer for implementation`）。没有 scope 的 assignment 只绑定 implementation。Standing defaults 写在 CE config（`plan_model`、`work_engine_mode`、`work_engine_preferences`）。详见[实现路由](./configuration.md#implementation-routing)。

Preference 失败时回退 native，并明确说明。Requirement 无法执行时 block；`lfg` 不会问是否弱化要求。

在 string-only host 中，implementation seam 是 `mode:return-to-caller implementation_engine:<compact-json> <plan-path>`。`plan_model:<alias>` carrier 与它并列传递，不会塞进 `ce-plan` request 内部。这两种 carrier 都不会变成 plan content、settled product decision 或 review input。

### Residuals 和 CI leftovers 会活过 session

未应用 review findings 会被 filed 和 committed。无法修好的 CI 会记录在 PR。`needs-human` leftovers（产品或设计决策）会 defer，而不是猜。只要 durable records 已经存在，run 可以带着这些 leftover 到达 `DONE`。

### Next work 是 offer，不是第二条 pipeline

如果已完成 plan 明确写了后续需要单独 planning 的 areas，`lfg` 会根据当前 evidence 选一个并 offer handoff。接受后会创建 `ce-handoff`，让 fresh session 对该 area 重新 brainstorm，并生成**独立** requirements-only plan。它不会编辑刚刚已经 shipped 的 plan。

---

## 快速示例

你刚完成 account-level notification mute 的 `/ce-brainstorm`。Wrap-up 提供 `lfg`。你运行 `/lfg`（或 `/lfg plan with fable`）。

`lfg` 从 brainstorm 整理 settled-decisions brief，在 requirements-only artifact 上调用 `/ce-plan`，并等待文件达到 `implementation-ready` 且 `execution: code`。之后 `/ce-work` 以 return-to-caller mode 实现。运行 Simplify。Review 报告 findings。`lfg` 应用符合条件的 mechanical findings 并 commit；其他内容转成 tracker tickets，加上 PR 的一个 run-report comment。Browser tests 运行。`ce-commit-push-pr` 打开 PR。`ce-babysit-pr` 最多做三轮 CI repair。

Run 最后打印 `DONE`，并提示如果想继续监看到 review/merge，可以运行 `/ce-babysit-pr <pr-url>`。它不会 merge。如果 plan 提到后续 area，可能额外收到 handoff offer；拒绝后 session 结束。

---

## 什么时候该用它

适合使用 `lfg`：

- 有一个软件 task，可以无需你介入地完成 plan、implementation、review 和 PR
- Task 已由 `/ce-brainstorm` 定形，或至少足够清楚能交给 `/ce-plan`
- 希望 CI failures 在有边界 loop 中自动处理
- 可以接受 branch 被 push、PR 被打开

以下情况跳过 `lfg`：

- 工作是非软件或 answer-seeking
- 仍需要 interactive product shaping → `/ce-brainstorm`
- 想 inspect/approve 每个 stage → `/ce-plan`、`/ce-work`、`/ce-code-review`、`/ce-commit-push-pr`
- 已有工作只需要 commit + PR → `/ce-commit-push-pr`
- 只想修一个已知 bug → `/ce-debug`
- Repo 有特殊 shipping rules，需要人工控制 git/release 工作

---

## 作为工作流的一部分

```text
/ce-brainstorm describe the feature
/lfg
```

从 `/ce-brainstorm` 开始，planner 会拿到 Product Contract。`lfg` 自己调用 `/ce-plan`；如果结果不是 implementation-ready code plan，就停止。

Sweep-reconciled plan 使用同一 seam：

```text
/ce-sweep
/lfg docs/plans/feedback-sweep-plan.md
```

`DONE` 之后：

```text
/ce-babysit-pr <pr-url>          # watch through review toward merge
/ce-explain <new-concept>        # only if lfg printed a New concepts: trailer
/ce-compound                     # optional, if there is reusable learning
```

## 独立使用

```text
/lfg add account-level notification mute settings
```

清楚的软件 task 可以直接调用。只是 planner 获得的 product context 会比 brainstorm 后少。

## 路由 planning 与 implementation

可以让 `lfg` 指定某个 model/harness author 一个 stage，同时让 `lfg` 保留 run 的其他部分。

- **限定 planning：**`plan with fable`、`plan with opus`。这是 `ce-plan` 内部的 model elevation。Planning 没有 cross-harness engine。把 harness 指派给 planning（`plan with Codex`、`plan on Cursor`）会 block。
- **限定 implementation：**`use Codex for implementation`（preference）、`only use Composer for implementation`（requirement）。`cursor` 表示 Cursor harness 的 default model；`composer` 表示通过 Cursor 使用 Composer-family model。
- **未限定 scope：**`use fable`、`with Codex`。只绑定 implementation。Interactive run 如果真正 ambiguous，`lfg` 只问一个问题，然后继续 hands-off。Headless run（scheduler、loop、nested orchestrator）则应用 implementation default，并 disclosure。
- **没有 stage instruction：**`ce-plan` 使用自己的 `plan_model` config（或无）；`ce-work` 先使用已在 context 中的 session/project instructions，再使用 checkout-local `work_engine_mode` 和 `work_engine_preferences`。

Feature text、quote、comparison 或 filename 中单纯提到 model 不会触发 routing。Fallback、timeouts 和 detached-worktree 行为见 [`ce-work`](./ce-work.md#choose-the-implementation-author)。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | 从当前 context planning（包括刚完成的 brainstorm）；如果 plan 是 implementation-ready code plan，就继续 pipeline |
| `<feature description>` | 传给 `/ce-plan`，然后继续 pipeline |
| `<requirements-only plan path>` | `/ce-plan` 原地 enrich 该文件，再继续 pipeline |
| `<description or path> + stage assignment` | Routing words 会从 product request 中移除。Scoped planning directive 交给 `ce-plan`；scoped implementation directive 交给 `ce-work`；unscoped assignment 只绑定 implementation。 |

输出：code changes、commits，通常还有 PR。没有配置 git remote：只有本地 commits。如果 bounded repair loop 后 CI 仍红，未解决 failures 会在 run 结束前被记录。

---

## FAQ

**`lfg` 会 merge PR 吗？**
不会。Pipeline babysit 会在 CI 已决定（或耗尽 fix budget）时停止。Merge 留给你。Closeout line 会指向 `/ce-babysit-pr`，用于 interactive watch toward merge。

**它会停下来让我 approve plan 或 diff 吗？**
不会。这正是这个 skill 的意义，也正因为如此，它不适合 in-the-loop work。

**如果 planning 无法产出 implementation-ready code plan？**
Pipeline 停止。非软件 task、requirements-only leftovers、knowledge-work plans、invalidating settlement conflicts 都会在 implementation 前 halt。

**剩余 review findings 去哪里？**
不会写进 PR description。能的话会 filed 到 project tracker，并在 PR 上留一个 run-report comment。

**如果没有 `origin` 会怎样？**
只创建本地 commits。不 push、不创建 PR、不监看 CI。

**可以把 planning 交给 Codex 吗？**
不可以。Planning 接受 model alias（`fable`、`opus`），而不是 harness。Implementation 才能换 harness。

---

## 另请参阅

- [`ce-brainstorm`](./ce-brainstorm.md)：最强的上游 requirements 来源；wrap-up 可以调用 `lfg`
- [`ce-plan`](./ce-plan.md)：pipeline 第一个必需步骤
- [`ce-work`](./ce-work.md)：implementation，以 return-to-caller mode 调用
- [`ce-simplify-code`](./ce-simplify-code.md)：review 前 simplification
- [`ce-code-review`](./ce-code-review.md)：只报告的 review gate
- [`ce-test-browser`](./ce-test-browser.md)：browser validation
- [`ce-commit-push-pr`](./ce-commit-push-pr.md)：有 remote 时的 shipping handoff
- [`ce-babysit-pr`](./ce-babysit-pr.md)：PR 打开后的 CI/review watch
- [`ce-handoff`](./ce-handoff.md)：closeout 时 opt-in 的 next-area snapshot
- [`ce-sweep`](./ce-sweep.md)：可由 `/lfg <plan path>` 交付的 rolling plan