# `ce-work`

> 按 plan 的 guardrails 执行；面对真实代码决定 HOW；完整交付功能，并 handoff 到一个干净 PR。

`ce-work` 是**执行** skill。它接收一份 plan（scope 较小时也可以只给 bare prompt），按 plan 的 guardrails 实现；持续运行 tests；选择 implementation engine 与安全 scheduling strategy；运行 quality gates；最后 handoff 给 commit + PR 流程。Implementation 可以留在当前 host，也可以把有边界的 units 路由给另一个合格 model 或 harness。最终 verification、canonical commits 和 shipping 仍由 host 负责。

它把 plan 视为**决策 artifact**：scope、decisions、units 和 tests 都以 plan 为权威。真正 implementation 由它自己决定。**这正是 `ce-plan` 刻意不预写的 HOW 阶段。**

这是 compound-engineering ideation 链中的第四步：

```text
/ce-ideate         /ce-brainstorm      /ce-plan             /ce-work
"What's worth      "What does this     "What's needed       "Build it."
 exploring?"        need to be?"        to accomplish
                                        this?"
```

`ce-work` 主要面向软件。它会 commit、跑 tests、打开 PR，并与 code review skills 集成。同时它还有一个轻量的**非代码 carve-out**：如果 plan 标记了 `execution: knowledge-work`（由 `ce-plan` 的 approach-altitude flow 生成），就会进入 knowledge-work 路径：读取 sources、synthesize 并产出 deliverable，跳过代码 lifecycle。其他没有该 marker 的非软件工作仍止于 `ce-plan`，由人执行。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | 读取 implementation-ready plan（或为 bare prompt 划 scope），按 guardrails 执行，持续跑 tests，并交付一个经过 review 的 PR |
| 什么时候用？ | 实现带 `artifact_readiness: implementation-ready` 的 `ce-plan` plan；小/中等 bare-prompt 工作；恢复部分已交付工作 |
| 会产出什么？ | Commits 和 PR（no-PR 路径则只有 commits）。Knowledge-work plans 则产出保存后的 deliverable，不进入 commit/PR lifecycle。 |
| Caller-owned mode | 给外层 orchestrator（例如 `lfg`）使用：`mode:return-to-caller <plan path>` 负责实现与本地验证，然后返回结构化 envelope，并跳过独立运行时的 shipping tail（最终 simplify、review、PR、CI）。实现中间的 Simplify as You Go 仍然会运行。 |
| 下一步是什么？ | Review PR；运行 `/ce-compound` 捕获 learnings |
| 区别在哪里？ | Plan-aware idempotency、原生或 cross-model implementation engines、保守 parallel waves、host-owned verification 与 commits、PR 中的 operational validation |

---

## 调用示例

空调用会选择 `docs/plans/` 中最新、符合条件的 implementation-ready code plan。如果最新匹配项仍是 requirements-only、knowledge-work 或 approach-plan，它会停止，而不是猜。传入 requirements-only 路径也会拒绝，直到 `ce-plan` 把它 enrich。Path 参数就是要执行的 plan。指定 engine 只改变“谁来写代码”，不会改变“谁负责验证或 shipping”。

```text
# Execute a specific implementation-ready plan and own the shipping tail
/ce-work docs/plans/notification-mute.md

# Implement a clear small or medium task without writing a plan first
/ce-work extract a shared duration formatter from the notification views

# Pick up the newest eligible implementation-ready code plan in docs/plans
/ce-work

# If this file is still requirements-only, the run stops and offers ce-plan
/ce-work docs/plans/2026-08-10-feat-notification-mute-plan.md

# Execute a knowledge-work plan: read sources, synthesize, skip the code lifecycle
/ce-work docs/plans/2026-08-12-research-memo-approach.md

# Prefer another harness or model for implementation; the host still verifies and ships
/ce-work use Codex for implementation on docs/plans/2026-07-15-example.md
/ce-work implement docs/plans/2026-07-15-example.md with Cursor
/ce-work use Codex to add retry limits to the existing webhook sender

# Require that route (interactive standalone asks before weakening it)
/ce-work only use Composer for implementation on docs/plans/2026-07-15-example.md

# Outer orchestrator: implement and locally verify, then return a structured envelope
/ce-work mode:return-to-caller docs/plans/notification-mute.md

# Resume, inspect, or clean up an existing external implementation run
/ce-work resume run 20260812-1430-ab12
```

工作很大、或 product shape 仍开放时先用 `ce-plan`。Bare-prompt mode 适合你已经能自己划 scope 的工作。

---

## 问题

只对 agent 说“implement this plan”，常见失败包括：

- 接手部分完成 branch 时，重复实现已经 shipped 的工作
- 把 plan 当脚本：机械修改字面列出的 files，即使更好的结构完全不同
- Tests 全部 mock：只能证明 isolated logic，无法证明 layers 是否真的交互
- 半成品功能：可见部分做好了，但 callbacks 没接、edge cases 没处理
- Parallel work 静默丢数据：多个 agents 改同一个 file，只有最后一个写入 survives
- 没有 quality gate：diff 直接进 PR，没有 simplification pass、没有 review、没有 operational monitoring

## 解决方案

`ce-work` 把 execution 运行成带明确 gates 的结构化过程：

- Plan 对 WHAT 有权威性；agent 面对代码决定 HOW
- 每个 task 前做 idempotency check：verification 已满足就跳过
- 根据 scope 选择 implementation（默认 native inline/subagents，也可使用受批准的 cross-model route）和 scheduling（serial 或 bounded independent waves）
- 行为变更前先发现 tests、选择证据；任何 task 标记 done 前补 integration coverage
- Portable、self-sizing code review，外加 residual-work gate：accept、file、fix 或 stop，但绝不静默 ship
- 每个 PR 都携带 operational validation plan：监控什么、什么条件触发 rollback

---

## 它的新颖之处

### Plan-aware execution，以及幂等 re-entry

`ce-work` 把 plan 当 decision artifact，而不是 script。对于 unified plans，它先检查 metadata；遇到 `artifact_readiness: requirements-only` 会拒绝，直到 `ce-plan` enrich。Scope、decisions、U-IDs、files、test scenarios 和 verification criteria 都具有权威性。执行期间 plan body 保持只读；progress 存在 git commits 和 task tracker 中。

每个 task 开始前，它都会检查该 unit 的工作是否已经存在、并且是否符合 plan intent。如果 verification 已满足，就把 task 标记 complete 后继续，不做 silent reimplementation。Context compaction 后 resume、接手别人的 branch，或数周后回到 partly-shipped plan 时尤其重要。

### Engine、workspace、scheduling 是三个独立决策

普通 synchronous native work 留在 active checkout。每个 implementation unit 使用当前 harness 提供的隔离能力，在新的、single-use native worker context 中执行。Detached external worker 一律拥有私有 linked worktree。真正 apply、verify、commit external 结果的只有 host，而且是在 canonical checkout 中完成。

Scheduler 只有在检查 dependencies、实际/预期 paths、shared interfaces、generated/config surfaces、migrations 和 shared runtime resources 后，才会允许 bounded wave 并行 author。之后结果按 advancing canonical tree 一次一个 fold 进来。Clean patch 不等于 semantic compatibility。只要有 overlap 或 uncertainty，就把相关工作退回 host resolution、re-dispatch 或 serial execution。

如果 plan 定义了 U-IDs，它们会成为 task prefixes，并进入 commit messages 和最终 summary。因为 U-IDs 稳定，所以跨 plan edits 也有效。来自 brainstorm 的 IDs（R/A/F/AE）如果存在也会保留。

### Test evidence、review 与 operational validation

代码能编译不代表 task 完成。改变 behavior 前，`ce-work` 会先发现已有 test files，并选择正确证据：使用已有 failing test、更新/加强真正拥有该 contract 的既有 test、添加 focused failing test、记录 characterization coverage，或写明 deliberate exception 以及替代 verification。承载 feature 的 task 标记 complete 前，还会检查 test scenarios 是否覆盖适用类别（happy path、edges、error paths、integration），并向外 trace 两层 callbacks、middleware 和 observers。

Standalone shipping 直到有 `ce-code-review` receipt，或 shipping summary 包含精确 skip phrase（`Code review: skipped (mechanical diff)` 或 `Code review: skipped (ce-code-review unavailable)`）才算完成。Mechanical 仅指 formatting、dep bumps、lint-only 或 generated artifacts。Review 本身只读。`ce-work` 随后应用符合条件的 fixes，再把剩余 actionable findings 送入四选一 residual gate（apply / file tickets / accept with durable sink / stop）。“Accept”必须有真实 durable record。Return-to-caller mode 则把 review 留给 caller（例如 `lfg`）。

每个 PR description 都包含 `Post-Deploy Monitoring & Validation` section。如果真的没有 production impact，也仍保留该 section，并把“无影响”作为明确 decision 记录。

### 对 bare prompt 做智能 triage

不是每次 invocation 都有 plan。`ce-work` 接受 bare prompt，并按复杂度 triage：trivial work（少量 files、无 behavior change）直接 implementation；small/medium work 建 task list；large 或 sensitive work 推荐先 `/ce-brainstorm` 或 `/ce-plan`。这套 triage 使 small work 直接调用变得合理。

Invocation 来源不会改变规则。Agent harnesses 不能可靠告诉 skill 是用户显式点名，还是模型自己选的。如果 conversation 中有一份明确、仍 active 的 plan（例如 agent 刚刚 author，用户说“proceed”），会优先使用该 plan，再考虑 bare-prompt triage。否则，具体 implementation request 就是 bare prompt。

如果为明确 bare-prompt work 选中了合格 external implementation route，`ce-work` 不会把整段 conversation 交给 worker。它会把请求提炼成私有、有边界的 implementation brief：goal、scope、发现的 files/tests、acceptance/verification、constraints、以及保守 units。如果无法在不猜的情况下填出 goal、bounded scope 和 authoritative verification，它会先澄清或路由到 `ce-plan`，不会发生任何 external egress。

### Session-settled decisions 不是留给你“优化”的

带 `session-settled:` label 的 KTD，记录的是用户真正审视并有理由选择过的 decision。`ce-work` 应按原样实现，而不是擅自“improve”。这种克制只针对被 label 的 KTD。Plan 留白部分仍正常使用 judgment，settled approach 内部真正 defects 也照常以 full strength 暴露。如果发现 settled decision 真正无法工作，这是 blocker return，绝不能静默变成 accepted residual。

---

## 快速示例

一份包含四个 implementation units 的 plan 到达。`ce-work` 读取它，注意到某个 unit 的 `Execution note` 要求先拿到 failing request-level proof，同时看到一个 deferred-implementation question。它以 U-ID 为 prefix 建 task list，并无需询问就离开 default branch，切到根据 plan 命名的 feature branch。

两个 units 共用 contract，所以串行。另两个独立，可以并行 author。Native execution 时使用 host 可用的 worker isolation。若选择 external route，每个 unit 得到 detached sibling worktree。Host 检查每个实际 change set，一次一个 fold 到 active checkout，verify，并分别创建 canonical commits。Idempotency check 发现其中一个 unit 的 verification 已被前一 session 满足，于是直接标 complete，不重复实现。

`ce-code-review` 针对小而低风险的 diff 自行选择 lite roster。两个建议 finding 随后都被处理。Final validation 通过，operational validation plan 完成，`ce-work` 调用 `ce-commit-push-pr` 并带 `branding:on`（如果项目 instructions 指定自己的 shipping process，就走项目流程）。Plan 本身保持 untouched。是否 shipped 根据 git 推导，不写回 doc。

---

## 什么时候该用它

适合使用 `ce-work` 的情况：

- `ce-plan` plan 已 ready，准备交付
- 没有 plan 的小/中等工作（bare-prompt mode 可处理）
- 正在 resume partly-shipped work
- 想做保守 parallel execution，并用隔离 concurrent workers
- 想要完整 shipping flow：tests、simplify、review、residuals、operational validation、PR

以下情况跳过 `ce-work`：

- Product behavior 还没决定 → `/ce-brainstorm`
- 非 trivial work 的 implementation guardrails 还没建立 → `/ce-plan`
- Bug 已有已知 root cause 且修复 obvious → `/ce-debug`
- 任务是非软件，而且 plan 没有标记 `execution: knowledge-work`。普通非软件工作由人执行；标记后的 knowledge-work plan 才进入 carve-out。

---

## 作为链式工作流的一部分

```text
/ce-ideate          (optional)
   |
   v
/ce-brainstorm
   |  requirements-only unified plan
   v
/ce-plan
   |  implementation-ready guardrails: U-IDs, files, test scenarios, scope, risks
   v
/ce-work
   |  honors the guardrails; figures out the HOW with code in front of it
   |  derives progress from git, not the plan body
   |  ships through quality gates to PR
   v
/ce-code-review     (self-sizing review for non-mechanical changes)
   |
   v
/ce-compound        (capture the learning)
```

Shipping 后，`/ce-compound` 会把可复用 learning 捕获到 `docs/solutions/`，让未来 `ce-plan` 与 `ce-work` 直接使用。

---

## 独立使用

很多人会直接给 `ce-work` 一个 bare prompt。Scope 小而 agent 本身能划清时，`ce-plan` 反而是过度仪式。

- 已知 root cause 的 bug fix：trivial 就直接 implementation；small/medium 则建 task list
- 小 refactor：extract helper、rename concept、consolidate duplication
- Resume partly-shipped plan：idempotency 防止重复实现
- 给已设计好的 feature 接线，formal planning 只会增加 ceremony
- Multi-feature parallel work：scheduler 可以让真正独立的 units 并行 author，再顺序 integrate 和 verify

如果 bare-prompt scope 很大（cross-cutting、sensitive surfaces、many files），`ce-work` 会推荐先 `/ce-brainstorm` 或 `/ce-plan`，再按你的选择继续。

## 作为外层 Orchestrator 的下层执行器

如果另一个 workflow 自己负责 post-implementation shipping gates（最终 simplify、code review、PR creation、CI watching），调用：

```text
/ce-work mode:return-to-caller <plan path>
```

该模式只让 `ce-work` 负责 implementation 与本地 verification。Phase 2 中的 mid-implementation “Simplify as You Go”仍会运行。之后，`ce-work` 返回结构化 envelope，包含 changed files、completed units、verification evidence 和 blockers；设置 `standalone_shipping_skipped: true`；不运行 standalone shipping tail。所有 post-implementation gates 仍由 caller 负责。

Automatic caller 还可以在 plan path 前传 `implementation_engine:<compact-json>`（一个 `mode`、`target`、`model`、`source` binding）以及 `implementation_run:<safe-id>`（resume 已有 run）。

## 选择 Implementation Author

Native execution 是默认值。你可以在当前 prompt 中把 implementation 指派给某个 target，而不改变 verification、commits 或 shipping tail 的 owner：

```text
/ce-work use Codex for implementation on docs/plans/2026-07-15-example.md
/ce-work implement docs/plans/2026-07-15-example.md with Cursor
/ce-work use Cursor with Grok for implementation on docs/plans/2026-07-15-example.md
/ce-work only use Composer for implementation on docs/plans/2026-07-15-example.md
/ce-work use Codex to add retry limits to the existing webhook sender
```

前三个是 preference：`ce-work` 会尝试 route；如果不可用，就以醒目的 requested-versus-actual disclosure 说明后，回退到 native 继续。第四个是 requirement：interactive standalone run 会在弱化它前询问；headless 或 automatic caller 则不 prompt，直接返回 blocker。判断 intent，而不是某个特定 keyword。

当前 task 的显式要求优先。仍 active 的 session preference 继续适用。Implementation-only caller binding 会保留其 recorded provenance。已经在上下文中的 active project/user instructions 可以提供 default。Per-checkout config 是 native execution 之前最后一层 preference。Feature prose、quoted text、examples 或 filenames 中偶然提到 model，不产生任何效果。

最后一个示例没有 plan。`ce-work` 会先基于 repository 和 tests 对请求划 scope，再只把有边界的 private brief 交给 Codex。Host 仍负责检查实际 change、authoritative verification、canonical commits 和 shipping tail。

可以在 CE config（先 `config.local.yaml`，再 `config.yaml`）写 host-relative 的有序 preference list：

```yaml
work_engine_mode: prefer       # off | prefer | require
work_engine_preferences:
  - harness: cursor
    model: composer
  - harness: codex
    model: "gpt-5.6"
  - harness: claude
```

[中央配置参考](./configuration.md#implementation-routing)解释了这个 checkout-local default 如何与 current-task、session 和 project instructions 共同生效。

每个 candidate 有一个 `harness`（`codex`、`claude`、`grok` 或 `cursor`）和可选 `model`。省略 `model` 表示使用该 harness 已配置的默认 model。Composer 是通过 Cursor 访问的 model family，因此写成 `harness: cursor` 加 `model: composer`。不要把 CLI flags 或 commands 放进 config。

`off`、被注释/缺失的 mode、以及 invalid mode 都保持 native default。`off` 只影响 standing config；不会取消当前 live intent 或 caller binding。`prefer` 按顺序尝试 candidates，之后可 fallback native，并 disclosure。`require` 只在 interactive standalone run 中询问；在 `lfg` 或其他 headless caller 下直接 block。

Candidate 只有在其 unattended、write-capable、isolated-workspace route 已通过 qualification，并且所需 CLI/auth 可用后才算 usable。

### External Run 会做什么

在任何 repository material 离开 host 前，`ce-work` 会 disclosure：instruction/config 来源、固定 recipient、暴露哪些 bounded unit material，以及哪些限制由 adapter 强制、哪些依赖 cooperative behavior。Adapter 使用 CLI 现有 authentication，接收 minimized environment，不能切换 recipient、扩大 scope、push、打开 PR 或自行选择 fallback。

每个 external unit 都从 clean recorded SHA 开始，在 `/tmp/compound-engineering-<effective-uid>/ce-work/<run-id>/` 下的 detached linked worktree 中运行（如果 `/tmp` 无法承载可写 private root，例如 sandbox 只 allowlist `$TMPDIR`，则使用 `$TMPDIR/compound-engineering-<effective-uid>/ce-work/<run-id>/`）。这用于 same-user concurrency 与 accidental-mutation containment，**不是 security sandbox**。Synchronous native units 仍使用 active checkout；`ce-work` 不会为每个 unit 都创建 temporary worktree。如果 selected plan 是唯一 dirty path，`ce-work` 会 disclosure，并先创建 plan-only checkpoint commit。任何不相关 dirt 都会让 external route unavailable。

每次 CE Work runner 启动都会独立固定两小时 hard cap，不受 shared runner 较短默认值影响。Worker 完成后让 working tree 保持 uncommitted。Host 会把完整 tree snapshot 成一个 synthetic transport commit，检查实际 change set，在不 commit 的情况下 apply，运行 authoritative tests，然后创建一个 host-owned canonical commit。Failed、timed-out、divergent 或未 integrated 的 runs 保留在 private run directory 中。用报告的 run id 再调用，可以精确 resume 一次。Live attempt 不能与 native fallback 竞态。也提供显式 reap 与 ownership-checked cleanup，用于 preserved attempts。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | 自动使用 `docs/plans/` 中最新的 `implementation-ready` code plan（或 legacy code plan）。如果最新匹配项是 requirements-only、knowledge-work、approach-plan 或 unclassified，则停止。 |
| `<plan path>` | 执行该 plan。Requirements-only unified plan 会被拒绝，直到 `ce-plan` enrich。 |
| `<bare prompt>` | 按复杂度 triage（Trivial / Small-Medium / Large） |
| `use Codex` / `with Cursor` / `only use Composer` | 请求或要求 external implementation author。Host 仍负责 verify、commit、ship。 |
| `mode:return-to-caller <plan path>` | 给 outer-orchestrator：实现并本地 verify，然后返回结构化 evidence，不运行 standalone shipping tail |
| `mode:return-to-caller implementation_engine:<compact-json> <plan path>` | Automatic-caller 形式，携带一个 implementation-only `mode`、`target`、`model`、`source` binding |
| `implementation_run:<safe-id>` 或 `resume run <id>` | Resume、inspect 或 clean up 已有 external run。不启动新工作。 |
| Knowledge-work plan（`execution: knowledge-work`） | 生成 planned deliverable；跳过 branch、test、review 和 PR machinery |

输出：commits，以及通常通过 `ce-commit-push-pr` 创建的 PR；如果 project instructions 指定了项目自己的 shipping process，则使用项目流程。优先级：user preference > project process > default。Plan 全程只读，`ce-work` 从不 mutate。是否 shipped 根据 git 推导，而不是记录在 doc 中。

---

## FAQ

**为什么 `ce-work` 不直接照 plan 的精确 signature 写全部代码？**
因为 plan 刻意不包含精确 signatures。它包含 decisions、units、files、scope 和 test scenarios。Plan 是 WHAT；`ce-work` 是 HOW。这种分离让 plan 能跨数周代码变化和不同 implementer 保持可移植。

**如果我没有 plan 呢？**
Bare-prompt mode 会按复杂度 triage。Trivial 直接 implementation。Small/medium 建 task list。Large 则建议先 planning。

**`ce-work` 会为每个 unit 创建 detached worktree 吗？**
不会。Synchronous native implementation 留在 active checkout，native subagents 使用 host harness 自己的 workspace behavior。只有独立运行的 external units 使用上文 controller-owned detached worktrees。

**这些 external worktrees 是 security sandbox 吗？**
不是。它们隔离 concurrent Git state、限制 accidental mutation，但 external CLI 仍以同一个 OS user 运行。`ce-work` 限制 packet 与 authority；更强 OS isolation 不在此功能范围。

**为什么每个 task 前都检查工作是否已经完成？**
Context compaction 后 resume、接手别人的 branch、回到 partly-shipped plan 都很常见。Idempotency 能避免 `ce-work` 静默重做已经存在的内容。

**什么是 Residual Work Gate？**
当 `ce-code-review` 产生 actionable findings，而 follow-up pass 没解决时，`ce-work` 不会静默 ship。它会询问：apply now / file tickets / accept（带 durable sink）/ stop。“Accept”必须真正记录到持久位置。

**`ce-work` 支持非软件 plan 吗？**
对于带 `execution: knowledge-work` 的 plan（由 `ce-plan` approach-altitude flow 生成），支持。轻量 carve-out 会读取 sources、synthesize 并生成 deliverable，跳过 commit/test/PR lifecycle。其他没有 marker 的非软件工作仍止于 `ce-plan`，由人执行。

**如果我传入 requirements-only brainstorm file 会怎样？**
Run 会停止，并告诉你 Product Contract 需要先由 `ce-plan` enrich。它会给出精确 `ce-plan <plan-path>` handoff。空调用如果最新匹配 artifact 仍是 requirements-only，也一样处理。

---

## 另请参阅

- [`ce-plan`](./ce-plan.md)：生成 `ce-work` 执行所依赖的 guardrails
- [`ce-brainstorm`](./ce-brainstorm.md)：定义 plan 应该实现什么
- [`ce-ideate`](./ce-ideate.md)：上游“what's worth exploring”发现流程
- [`ce-code-review`](./ce-code-review.md)：portable self-sizing review 路径
- [`ce-commit-push-pr`](./ce-commit-push-pr.md)：处理最终 commit + PR flow
- [`ce-compound`](./ce-compound.md)：shipping 后捕获可复用 learning