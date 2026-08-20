# `ce-code-review`

> 结构化 code review：按风险选择 personas、以 confidence gate 筛选 findings，并输出合并/去重后的报告。

`ce-code-review` 是一个针对 diff、按需调用的 **findings** skill。它可以分析 PR、指定 branch 或当前 checkout，根据实际改动选择 reviewer personas 并 dispatch，随后把 findings 合并、去重成一份报告。每条 finding 都带 severity（P0-P3）、用于表示后续处理形态的 autofix class（`gated_auto`、`manual`、`advisory`），以及 owner。

Review 默认只报告，不修改。只有传入 `apply:local`，或用户明确要求应用本次 review 的 findings，才允许本地修复。`mode:agent` 始终只报告，把 mutation 留给 caller。

它不是对文档做整体 verdict 的 `ce-pov`，不是对 planning doc 输出 findings 的 `ce-doc-review`，也不是调查 broken behavior 的 `ce-debug`。

`ce-work` 在 shipping 前会调用它作为可移植 review 路径。`ce-optimize` 和 `ce-debug` 也会对各自产生的 diff 调用它。任何时候需要结构化 review，都可以直接调用。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | 根据 diff 选择 reviewer personas，dispatch 它们，并通过 confidence gating 把 findings 合并成一份报告 |
| 什么时候用？ | 打开 PR 前、大型或敏感改动后，或 harness 没有内置 `/review` 时 |
| 会产出什么？ | 一份结构化 findings 报告。获得显式 local-apply 权限时，还可以应用经过验证的 fixes，并添加 Applied section。永远不 push |
| Modes | Markdown 报告（默认）和 `mode:agent` JSON handoff。除非另行授权 local apply，两者都只报告 |

---

## 调用示例

可以 review 当前 branch、不 checkout 的 PR、指定 branch、指定 base ref、关联 plan、给 caller 的 JSON，或显式 local apply。“Quick”会让位于 harness-native reviewer。

```text
# Review the current branch. Base comes from origin/HEAD or PR metadata.
# Relevant plan and session context are discovered automatically.
/ce-code-review

# Review a specific PR without checking it out
/ce-code-review https://github.com/acme/widgets/pull/1234
/ce-code-review 1234

# Review a named branch without checking it out
/ce-code-review feat/notification-mute

# Review the current checkout against an explicit base (skips scope detection)
/ce-code-review base:origin/main

# Load a plan for requirements verification
/ce-code-review plan:docs/plans/2026-03-25-001-feat-foo-plan.md

# JSON handoff for a caller. Always report-only. The caller applies.
/ce-code-review mode:agent

# Review this checkout and apply verified findings locally. Never pushes.
/ce-code-review apply:local
/ce-code-review review this branch and fix eligible findings locally

# Force the full reviewer roster (skip the small-diff lite path)
/ce-code-review depth:full

# Flat report, no thematic groups
/ce-code-review grouping:off

# Ask for a lighter pass. Defers to the harness-native /review when one exists.
/ce-code-review give this branch a quick review
```

不要把 `base:` 与 PR/branch target 一起使用。不要把 `apply:local` 与 `mode:agent` 一起使用。冲突的 mode 或 grouping flags 会直接报错并停止。

---

## 问题

通用型 code review prompt 往往以这些方式退化：

- 只给表面 finding（“consider adding tests”），却不说具体要测什么
- Finding 与 diff 不匹配：docs-only change 却做 security review，typo fix 却谈 performance
- 没有 severity calibration，所有 finding 都被说成 critical
- 没有 confidence calibration，推测性的“could be a bug”与已验证 defect 看起来一样
- 只经过一个 model 的一次 reasoning
- Findings 只留在聊天中，没有记录，也没有 fix queue
- 另一个 agent 正在跑 tests 时 mutation shared checkout，结果不可定义

## 解决方案

`ce-code-review` 把 review 运行成带显式 gates 的 pipeline：

- Diff-aware persona selection。Correctness 始终运行；其他 reviewer 只有在实际改动 surface 涉及对应风险时才选
- 并行 persona dispatch，并受 harness active-subagent limit 约束
- Confidence-gated synthesis。Findings 会 merge、dedupe，在 cross-persona agreement 时提升 confidence，并按 autofix class 路由
- Severity（P0-P3）与 autofix class 相互独立：前者表示紧急程度，后者表示后续处理形态
- Presentation 与 authority 分离。默认 Markdown 与 `mode:agent` JSON 都只报告；`apply:local` 才授予本地 mutation 权限
- Quick-review short-circuit。包含“quick”“fast”“light”的请求会交给 harness-native `/review`

---

## 它的新颖之处

### Diff-aware persona selection

小型低风险改动会运行 correctness（如果存在适用 standards files，再加 project-standards）。带 migration 的 Rails auth feature 会加入相关 domain lenses。Skill 会根据 diff 判断哪些 personas 合适：

- **Always-on：**`correctness-reviewer`
- **Standards：**只有存在至少一个适用 standards file 时才运行 `project-standards-reviewer`
- **Generic conditional：**changed tests/harnesses，或 meaningful runtime behavior 变化但没有对应 test work 时选 testing；大型/结构性工作选 maintainability；agent-facing surface 选 agent-native；只有现有 `docs/solutions/` corpus 有合理匹配时才选 learnings
- **Cross-cutting conditional：**security、performance、API contract、data migrations、reliability、adversarial、previous-comments。只有 diff 真正触及对应 concern 才选择
- **Stack-specific：**Julik frontend races、Swift/iOS。只有匹配 runtime domain 被触及时选择
- **CE conditional：**风险 migration diff 使用 `deployment-verification-agent`。Schema drift 和 migration safety 归 `data-migration` persona

Persona selection 是 agent judgment，不是 keyword matching。Instruction-prose files（Markdown skills、JSON schemas）属于 product code，但会跳过 runtime-focused reviewers。例外是**可能静默放行的 verification mechanism**（CI/CD gate、build/deploy step、coverage/lint gate，或可能遮蔽 production 的 test harness/mock）：即使只是很小的 config diff，也会加 adversarial lens，因为风险在 fidelity（真实是红的却能变绿），而不在 blast radius。

传入 PR number 或 URL 时，trivial automated PR（lockfile bump、chore version increment）会跳过。Draft PR 正常 review。

`depth:auto`（默认）会把 1–39 行、低风险、code-only diff 收缩到 lite roster。`depth:full` 关闭这条 fast path，让完整 always-on roster 无论大小都运行。两种 token 都不会凭空加入无关 domain。

### Cross-model adversarial pass

当 adversarial 被选择，且 working tree 就是被 review 的 head（当前 branch，或本地 tree 已匹配 PR head）时，adversarial lens 会在独立 read-only process 中通过**与 host 不同的一个 model provider**运行。只要 peer 启动成功，它就**替代** in-process `adversarial` persona；两者绝不会收到同一份 brief。如果 peer 无法启动，或启动后只返回 session-quota/auth-context failure，则回退到 in-process persona；若还有符合条件、已 announce 的不同 family peer，会先尝试下一位，否则本地 persona 覆盖该 lens。对于顽固 transient rate limit，同一路由只 retry 一次，然后回本地；不会换 recipient。Remote PR/branch diff 保持 in-process persona，因为该 reviewer 可以直接检查 fetched refs。

Peer 与其他 in-process reviewer 达成 agreement，是 synthesis 中很强的 promotion signal。

CE config 中 `cross_model_review_mode: off` 会完全关闭这一步——不解析 peer，也不会有任何内容离开 host；in-process reviewers 覆盖该 lens，Coverage 会说明 pass 被 checkout config 禁用。对话中直接请求 peer 可以只针对本次运行覆盖它。Peer target 的选择既可自动也可覆盖，优先级为：conversation、CE config 的 `cross_model_peer:`、active project instructions，然后 `codex → claude → grok → composer`。`Cursor` 表示 `cursor-agent` 使用已配置 default/Auto model；`Composer` 表示通过 Cursor 使用 Composer model。除非能验证 serving family 与 host 不同，否则 Cursor Auto 不计入 independent agreement。CE config 的 `cross_model_model:` 与 `cross_model_effort:` 可固定 target model（例如 `fable`）和 reasoning effort；peer 无法满足时会明确说明并跳过，而不是自行替换。详见[配置参考](./configuration.md)。

**前置条件：peer agent CLI。** 该 pass 驱动 read-only *agent* CLI（`codex`、`claude`、`grok` 或 `cursor-agent`），让 peer 自己检查 tree；仅有 `OPENAI_API_KEY`、Anthropic key 或 Gemini key 并不能启用它。Peer 会从 `PATH` 查找，也会检查 Codex desktop app 内附的 CLI（2026 年 7 月 app merger 后位于 `ChatGPT.app/Contents/Resources/codex`，旧安装为 `Codex.app/…`；app 不会自动把它链接进 `PATH`）。Gemini 没有独立 peer target——只有 `cursor-agent` 能证明 serving family 是 Gemini 时，才通过 Cursor 参与。没有安装任何 peer CLI 时，skill 会运行 in-process adversarial reviewer，并报告 `cross-model pass: not run`；skip reason 会说明要安装什么。

它与 `ce-doc-review` 共用 provider/route kernel，但 product scope 更窄：只做 adversarial、使用 diff/work-tree delivery；不像 doc-review 那样运行 judgment trio 或 whole-doc sweep。

### Severity 与 autofix class 相互独立

Severity 回答**紧急程度**（P0 = critical breakage，P3 = 由用户自行决定）。Autofix class 是关于后续处理形态的**信号**，不是 apply 权限：

- `gated_auto` → 存在具体 `suggested_fix`，是清晰可应用候选
- `manual` → 需要 design input 或 handoff 的 actionable work
- `advisory` → 只报告（learnings、rollout notes、residual risk）

最终 route 由 synthesis 决定。Persona 提供的 routing metadata 只是输入。发生分歧时默认选择更保守 route。某条 finding 是否真的应用，要在获得 apply authority 后再判断。

### Presentation 与 apply authority 分离

| Mode | 何时 | 行为 |
|------|------|----------|
| **默认 Markdown** | 用户直接调用 | 只报告的 Markdown，带稳定 findings 和 Actionable Findings summary |
| **`mode:agent`** | `mode:agent`（alias `mode:headless`） | 输出一个 JSON object。只报告，由 caller 应用 findings。`mode:non-interactive` 不是该 alias（传入会 fail closed） |
| **显式 local apply** | `apply:local`，或明确要求 apply/fix 本次 review findings | 继续使用 Markdown presentation。Pre-review tree 干净时可以应用 verified fixes 并 commit。永远不 push |

Skill 从不切 branch。PR 或 branch argument 只选择 review *scope*（无需 checkout 计算 diff），不授予 mutation 权限。显式 local apply 会就地编辑当前 checkout。若要把当前 checkout 与另一个 ref 比较，使用 `base:<ref>`。

### Quick-review short-circuit

当你要求“quick”“fast”或“light”review 时，skill 会交给 harness-native code review（例如 Claude Code 的 `/review`），而不是 dispatch multi-agent pipeline。`mode:agent` 会绕过 short-circuit，始终运行完整 pipeline。

### Synthesis、grouping 与 plan checks

Reviewers 返回后，synthesis 会验证每条 finding、锚定到实际 diff、跨 personas 去重、在 agreement 时提升 confidence、解决 contradiction，并按 autofix class 路由。输出是一份统一报告，包含校准后的 severity、evidence quotes 和明确 ownership。

当 findings 跨越明显不同 concerns 时，相关 finding 会归到短主题下（默认 `grouping:auto`）。Groups 只是 triage lens，不会重构 findings：稳定 `#` 保持不变，group 通过 `#2, #3` 引用。`grouping:off` 输出平铺报告；`grouping:always` 即使小 review 也强制分组。

如果 diff 有关联 plan（`docs/plans/*.md` 或 `.html`），skill 会通过 `plan:` argument、PR body link，或 branch name auto-discovery 找到它；对于 implementation-ready artifact，会根据 Product Contract Requirements 和 Implementation Units 验证 diff。

`plans/`、`solutions/` 和旧版 `brainstorms/` 下的 pipeline artifacts 受保护。任何要求删除或 gitignore 它们的 finding 都会被丢弃。

如果发现的 plan 带 `session-settled:` decisions，仅仅偏好不同 approach 的 finding 会作为 report-only 路由，并打 `settled_conflict` stamp。Settled approach 内的真实 defect 仍保留完整 severity。Reviewers 看不到这些 annotations；由 orchestrator 事后 triage。

`/ce-work` 等 callers 会读取 Actionable Findings summary（或 JSON `actionable_findings` 字段），并负责 residual handling：apply now、file tickets、accept with durable sink 或 stop。本 skill 不运行这道 gate。

---

## 快速示例

你在包含 database migration 的 Rails auth feature branch 上运行 `/ce-code-review`。

Skill 发现当前是 feature branch（还没有 PR），从 `origin/HEAD` 解析 base 并计算 diff。它根据 commit messages 写 2–3 行 intent summary，根据 branch name 自动发现 `docs/plans/` 中的 plan；若 artifact 已 implementation-ready，则读取 Product Contract Requirements 与 implementation U-IDs。

它选择 correctness；存在适用 standards files 时加 project-standards；migration 改变 test/harness code，或有 meaningful runtime behavior 变化却没有对应 test work 时加 testing；auth touched 加 security；token cleanup background job 加 reliability；存在 migration file 加 data-migration；migration 风险高时加 deployment-verification。Auth 和 persistence writes 会触发 adversarial。因为 review 的是当前 checkout，该 lens 会由 cross-model peer 运行，而不是 in-process persona。

Synthesis 把原始 findings 合成更小、互不重复的一组。若干是 caller 可处理的 `gated_auto` candidates，两条是 `manual` deployment decisions，其余为 advisory。每条 finding 都有 anchored evidence 和稳定 number。由于这是 bare invocation，review 只报告，不修改 checkout。

之后你可以自己应用 selected findings、把 JSON report 交给 `/ce-work`，或重新运行并加 `apply:local`。

---

## 什么时候该用它

适合使用 `ce-code-review`：

- 即将为敏感/大型工作开 PR（auth、payments、migrations、public APIs）
- Harness 没有内置 `/review`，但仍想做真正 review
- 想要带 calibrated severity 的结构化 findings，而不是 chat dump
- 明确想要更深入、multi-persona pass（“review this thoroughly”或 `depth:full`）
- 被其他 skill 调用（`/ce-work` shipping 前、`/ce-optimize` 对 cumulative optimization diff、`/ce-debug` 在非 trivial fix 后）

以下情况跳过 `ce-code-review`：

- 只想快速 light review。直接说“quick review”，short-circuit 会交给 harness-native `/review`
- 改动是 typo、formatting 或小 dependency bump。Lite roster 已足够
- 想 review planning document → `/ce-doc-review`
- 想对 plan 给整体 take，而不是 diff findings → `/ce-pov`
- 想调查 broken behavior → `/ce-debug`

---

## 作为工作流的一部分

`ce-code-review` 是其他 skills 调用的 portable review path：

- **`/ce-work`** shipping 前调用 `mode:agent`。它会 self-right-size（小而低风险的 code-only diff 使用 lite roster，否则完整 roster）。如果 plan、task 或用户要求 thorough review，就传 `depth:full`。随后 `ce-work` 应用 findings，并运行自己的 Residual Work Gate
- **`/ce-optimize`** 在 merge 前对 cumulative optimization-branch diff 运行它
- **`/ce-debug`** 在非 trivial fix 后运行，并限制 scope，避免跑到不相关 branch work

---

## 独立使用

- **当前 branch（只报告）：**`/ce-code-review`
- **当前 branch 并应用 verified findings：**`/ce-code-review apply:local`
- **指定 PR：**`/ce-code-review 1234` 或 `/ce-code-review <PR URL>`
- **指定 branch：**`/ce-code-review feat/notification-mute`
- **指定 base ref：**`/ce-code-review base:abc1234` 或 `base:origin/main`
- **指定 plan：**`/ce-code-review plan:docs/plans/.../plan.md`

Bare 与 `mode:agent` review 都只报告，可以安全地与同一 checkout 上的 browser tests 并发。不要让一个获得显式 local-apply 权限的 review 去修改另一个 agent 正在主动使用的 checkout。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | Review 当前 branch（base 来自 `origin/HEAD` 或 PR metadata） |
| `<PR number or URL>` | 不 checkout，直接 review 该 PR |
| `<branch name>` | 不 checkout，直接 review 该 branch |
| `base:<sha-or-ref>` | 跳过 scope detection，把当前 checkout 与该 ref 对比 review |
| `plan:<path>` | 加载 plan 做 requirements verification |
| `mode:agent` | JSON machine handoff。只报告。`mode:headless` 是 deprecated alias。`mode:non-interactive` 在这里无效。`mode:report-only` 被忽略 |
| `apply:local` | 授权 verified local fixes。与 `mode:agent` 冲突 |
| `depth:full` / `depth:auto` | `full` 强制完整 roster（跳过 small-diff lite path）。`auto`（默认）self-right-size |
| `grouping:auto` / `grouping:off` / `grouping:always` | Thematic triage grouping（默认 `auto`）。只影响 presentation，从不改变 reviewer selection、merge 或 apply |

冲突 mode flags（或 grouping flags）会报错并停止。`base:` 与 PR/branch target 同时传也会报错。两者选其一。

---

## FAQ

**为什么不直接用 harness 内置 `/review`？**
适合时当然应该用。Quick-review short-circuit 会明确把快速请求交给它。`ce-code-review` 面向的是需要 diff-aware persona selection、calibrated severity 的 structured findings、autofix routing，以及 caller 可继续处理的 residual handoff 的情况。

**它怎么决定 dispatch 哪些 personas？**
根据实际 diff 做 agent judgment，不是 keyword matching。每次 multi-agent review 都运行 correctness。存在适用 standards files 时运行 project-standards。Generic、cross-cutting 和 stack-specific personas 只在对应 concern 存在时加入。仅仅存在 production file、或非 behavior change，不会选择 testing。可能 silent-pass 的 verification mechanism 不论大小都会选 adversarial（如果 tree 在本地，也会运行 cross-model pass）。

**默认、`mode:agent` 和 `apply:local` 有什么区别？**
默认是面向人的 Markdown report，只报告。`mode:agent` 是同一 pipeline 序列化成一个 JSON object 给 caller，始终只报告。`apply:local` 则是独立权限，允许 Markdown run 在本地应用 verified findings。`mode:headless` 是 `mode:agent` 的 deprecated alias。`mode:non-interactive` 在其他 CE skills 中表示“禁止 prompts”，在这里不是合法 mode。

**它和 `ce-doc-review` 的 cross-model pass 有什么区别？**
独立性系统相同（host attestation、multi-provider selection、read-only peer CLI），lens policy 不同：code-review 只跑 **adversarial**，且一旦 peer 启动就替代 in-process adversarial persona。Doc-review 会在 in-process reviewers 之外运行 judgment trio 加 whole-doc sweep。Code-review peers 就地 review work tree/diff；Doc-review 会把 document 嵌入更隔离的 scratch。

**为什么它永远不切 checkout？**
Skill 从不运行 `git checkout` 或 `git switch`。传 PR/branch 只选 review *scope*，不是 mutation permission。显式 local apply 可以编辑当前 checkout，但永远不切 branch。要把当前 checkout 与其他 ref 比较，传 `base:<ref>`。

**可以与 browser tests 并发吗？**
Bare 和 `mode:agent` review 都只报告，可以安全并发。显式授权 local-apply 的 run 可能 mutation working tree，因此不要对另一个 agent 正在使用的 checkout 这么做。

**支持非软件工作吗？**
不支持。这个 skill 与 git、code reviewers、PR contexts 紧密耦合。Docs（requirements、plans）用 `/ce-doc-review`；对文档整体判断用 `/ce-pov`。

---

## 另请参阅

- [`ce-work`](./ce-work.md)：主要上游 caller，shipping 前调用本 skill
- [`ce-doc-review`](./ce-doc-review.md)：面向 documents 而非 code 的 sibling skill
- [`ce-pov`](./ce-pov.md)：给 verdict/holistic take，而不是 diff findings
- [`ce-debug`](./ce-debug.md)：调查 broken behavior，包括 review 发现的 bugs
- [`ce-resolve-pr-feedback`](./ce-resolve-pr-feedback.md)：PR 打开后处理 incoming reviewer comments
- [`ce-simplify-code`](./ce-simplify-code.md)：`ce-work` 在 review 前调用；它是补充，不是替代