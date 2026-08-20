# `ce-compound`

> 把刚解决的问题记录下来，让下次遇到同类问题只花几分钟，而不是几小时。知识会产生复利。

`ce-compound` 是**知识捕获** skill。解决一个非 trivial 问题后，它会在 `docs/solutions/` 写一份结构化文档，记录 symptoms、root cause、没有奏效的方法、最终 solution 和 prevention。未来的 `ce-plan` 和 `ce-ideate` 会把这个目录当作 institutional memory，因此同一调查无需做第二遍。

Compound-engineering ideation 链是 `/ce-ideate` -> `/ce-brainstorm` -> `/ce-plan` -> `/ce-work`。`ce-compound` 负责**闭环**。在 debug 或 build session 结束时捕获下来的文档，会作为 grounding 反馈回上游。第一次解决“brief generation 中的 N+1 query”可能要研究 30 分钟；第二次找到已有文档，2 分钟就能修好。

它是可选的。Typo、一行修复和纯机械工作可以跳过。

---

## TL;DR

| 问题 | 回答 |
|----------|--------|
| 它做什么？ | 把已解决问题记录到 `docs/solutions/[category]/[filename].md`，包含结构化 frontmatter、bug-track 或 knowledge-track sections，以及 cross-references |
| 什么时候用？ | 解决非 trivial 问题后；当你说“that worked”“it's fixed”“problem solved”时 |
| 会产出什么？ | `docs/solutions/` 中的一份文档，加上可选的 `CONCEPTS.md` 词汇捕获。Interactive Full 在获得同意后，还可能编辑 `AGENTS.md`/`CLAUDE.md` 以提高 discoverability。 |
| 下一步是什么？ | 没有 menu。如果新 learning 暗示旧文档可能已 stale，可选运行 `/ce-compound-refresh`。 |

---

## 调用示例

空调用会捕获当前 conversation 中最近一个已验证 fix。如果 session 里有多个已解决问题，可以给 context hint 来聚焦。`mode:non-interactive` 面向 automation 和 standing instructions：不问 blocking questions，默认 Full。`depth:` 只在 non-interactive intent 下有效。

```text
# Capture the verified solution from the current conversation
/ce-compound

# Focus capture when the session contains several solved problems
/ce-compound the email digest race condition we fixed

# Unattended Full capture (default depth when mode:non-interactive is set)
/ce-compound mode:non-interactive the verified caching fix

# Unattended single-pass capture: no subagents, no overlap research
/ce-compound mode:non-interactive depth:lightweight the verified caching fix

# Unattended Full capture, including the automatic session-history probe
/ce-compound mode:non-interactive depth:full the verified caching fix
```

每次 run 只记录一个 learning。如果 session 产生了多个不同 learning，请按顺序分别调用。单独要求“bootstrap CONCEPTS.md”会被转给 `ce-compound-refresh`。只有在 caller 应负责所有 follow-up decisions 时才使用 non-interactive mode。普通 interactive capture 在修改 project guidance 前仍可以询问。

---

## 问题

很多团队会把同一个问题解决两次，甚至由同一个人重复解决，因为第一次 solution 只存在于 conversation、chat history 或某个 teammate 的脑子里。常见失败形态：

- Solution 只存在聊天里：Slack thread、Linear comment、agent transcript；一周后就找不到
- 虽然写了文档，但不可发现：放在没人搜索的 wiki；或者 `docs/solutions/` 已经存在，但 agents 根本不知道要查
- 再次遇到时又重写：同一个问题产生一份略有不同的新文档，之后两份一起 drift
- 没记录 anti-patterns：调查中“什么没用”往往最昂贵，却最先消失
- 捕获发生在 session-end clutter，而不是 session-end clarity：等 context 已经淡掉才写

## 解决方案

`ce-compound` 会在上下文最新鲜的时候，以结构化 capture flow 运行：

- 两种模式：Full（并行 subagents 做 cross-referencing 和 duplicate detection）与 Lightweight（single-pass，更快）
- Bug track 和 knowledge track 根据文档类型采用不同 section structure
- Overlap check 决定应该更新 existing doc，而不是再造 duplicate
- Discoverability check 确保项目 instruction file 会提示 `docs/solutions/`，让未来 agents 找得到（interactive Full 编辑前询问 consent；non-interactive/lightweight 只报告或提示）
- Specialized post-review 可以加强文档：performance、security、data-integrity，以及 read-only simplification checks 会 review drafted learning，但不 mutation product code

---

## 它的新颖之处

### 两种模式，由 agent 自行选择

**Full mode** 会并行运行三个 research subagents（Context Analyzer、Solution Extractor、Related Docs Finder），并自动扫描 Claude Code、Codex、Cursor、Pi 和 oh-my-pi (omp) 的 session history。它会 cross-reference existing docs、检测 duplicates，并运行 specialized reviews。

**Lightweight mode** 以 single pass 写同一种 solution-doc artifact，不使用 subagents，也不 cross-reference。它会跳过 overlap detection、session-history research 和 semantic grounding validation。

**由 skill 自己选择 mode，不会问你。** Full 是默认值。只有真正存在 context pressure（session 接近上限，或 fix 很 trivial、cross-referencing 没有价值）时才选 Lightweight。这些条件 agent 能观察到，而用户看不到。Skill 会在输出第一行说明运行了哪个 mode 以及原因。如果选错，重新运行成本很低。

Automation 可以无需 prompt 明确选择同样的 tradeoff：`mode:non-interactive depth:lightweight` 运行 single-pass workflow；`mode:non-interactive depth:full` 运行完整流程，包括自动 session-history probe。裸 `mode:non-interactive`（以及 deprecated alias `mode:headless`）默认仍是 Full。Depth 只在 non-interactive 下可用。没有 non-interactive intent 却带 depth flag、未知 value，或冲突 depth flags，都会显式失败。

### Bug track 与 knowledge track

Skill 根据 `problem_type` 分类：

- Bug track：Symptoms、What Didn't Work、Solution、Why This Works、Prevention。适用于 build errors、test failures、runtime errors、performance issues、integration issues。
- Knowledge track：Context、Guidance、Why This Matters、When to Apply、Examples。适用于 architecture patterns、design patterns、tooling decisions、conventions、workflow practices。

Track 决定 section order 和 frontmatter fields。

`problem_type`、`severity` 和 `resolution_type` 是 closed enums。`component` 与 `root_cause` 是 open vocabulary，category directories 只是默认 layout：当 `docs/solutions/` 已经有 learnings 时，classifier 会抽样现有 frontmatter 与 directory names，优先复用 corpus 在该领域已经采用的词汇；只有空 corpus 或未覆盖领域才回退到 schema suggested values 与 default mapping。这样已有自己文档 vocabulary 的 repo 能保持原样，现有 retrieval 也仍能找到新文档。

### Overlap、discoverability 与 grounding

Related Docs Finder 会从五个维度给 existing `docs/solutions/` 内容做 overlap scoring：problem statement、root cause、solution approach、referenced files、prevention rules。

- High overlap（4–5 个维度匹配）：更新 existing doc，path 不变，并添加 `last_updated`
- Moderate overlap（2–3 个维度）：新建 doc，同时标记需要 consolidation review（可能触发 `ce-compound-refresh`）
- Low/none：正常新建 doc

每次 run 也会检查项目 instruction file（`AGENTS.md` 或 `CLAUDE.md`）是否能引导未来 agent 发现 `docs/solutions/`。如果不能，interactive Full 会提出最小增量，询问 consent 后应用。Non-interactive 会报告 `Instruction-file edit: gap noted, not applied`，但不编辑。Lightweight 只给 tip。

Learning 真正进入复利知识库之前，会先对照 tree 检查 claims。Deterministic script 会检查引用的 repo paths、commit SHAs、relative links 和 dangling scaffold（flags 需要 adjudicate，不会自动判 fail）。之后 read-only validator subagent（Full mode，包括 non-interactive Full）会通过引用定义行为的 source line 验证 code-behavior claims，并把 merge-state claims 与 remote truth 对照。Lightweight 只保留 deterministic check，跳过 validator subagent。

### Session history、refresh 与 auto-invoke

Full mode 总会跑一个低成本 two-stage session-history probe。Discovery+metadata pass 与 research subagents 并行。Claude sessions 从 `~/.claude/projects/`（或设置了 `CLAUDE_CONFIG_DIR` 时的对应 `projects`）列出，并在记录的 `cwd` 是 repo root、其 parent 或其内部路径时保留——包括从 parent directory 启动的 sessions。Codex sessions 来自 `~/.codex/sessions/`（或设置 `CODEX_HOME` 时的对应 `sessions`）以及 `~/.agents/sessions/`。只有 candidate session 达到 relevance bar（当前 branch 匹配，或至少两个 topic-keyword hits）时，才升级到 extraction 和 synthesis。命中后，findings 会进入 bug track 的“What Didn't Work”，或 knowledge track 的“Context”。Lightweight 完全跳过此流程。

捕获新 learning 后，`ce-compound` 会检查是否应该针对一个 narrow scope hint 调用 `/ce-compound-refresh`。默认不会运行 refresh。只有新 learning 明确暗示某个特定旧 doc 可能 stale 时才会建议。

“That worked”“it's fixed”“working now”“problem solved”这类 phrase 会 auto-invoke skill，使 capture 发生在 context 仍新鲜时。也可以显式使用 `/ce-compound [context]`。

---

## 快速示例

你刚花了 45 分钟 debug brief-generation flow 中的 N+1 query。确认 fix 有效后，你说“that worked, ship it.”

`ce-compound` 会 auto-invoke（或由你显式调用）。由于剩余 context 充足，它选择 Full mode，并在输出顶部说明 `Ran Full mode.`，无需 prompt。

三个 subagents 并行 dispatch。Context Analyzer 把工作分类为 `performance_issue`（bug track），并建议 filename/category。Solution Extractor 用 before/after code 组织 fix。Related Docs Finder 发现与另一份旧 N+1 doc 有 moderate overlap。同时 session-history probe 扫描最近 sessions；没有 candidate 达到 relevance bar，因此记录“no relevant prior sessions.”

Orchestrator 组装文档、验证 frontmatter，并写入 `docs/solutions/performance-issues/n-plus-one-brief-generation.md`。随后运行 grounding validation：mechanical script 确认所有 cited path/SHA 都能解析；validator subagent 引用实际 source line，验证文档关于 ORM 默认 batching behavior 的 claim。Discoverability check 发现 `AGENTS.md` 没提 `docs/solutions/`，于是提出在 existing directory listing 里增加一行，并在你确认后应用。

Phase 3 dispatch 本地 `performance-oracle` prompt；由于 doc 有 code examples，还会对 drafted examples 做 read-only simplification check。Phase 2.5 给出 refresh recommendation：旧 N+1 doc 可能适合 consolidation review。Skill 建议 `/ce-compound-refresh n-plus-one` 作为 narrow scope hint，然后结束。没有“What's next?” menu。

---

## 什么时候该用它

适合使用 `ce-compound`：

- 刚解决一个非 trivial 问题，context 还很新鲜
- 你说“that worked”“it's fixed”“working now”“problem solved”
- 到了自然 pause，想在 context 淡掉前捕获 learning
- 这个问题花了有意义的调查成本（不是 typo 或 one-line fix）

以下情况跳过 `ce-compound`：

- 问题还在进行中，或 solution 尚未验证
- Fix 只是 trivial typo/obvious error，没有 generalizable insight
- 工作完全 mechanical（formatting、dependency bumps）
- 想要 repo-wide concept map，而不是一个 solved problem 的 learning → `/ce-compound-refresh`

---

## 作为工作流的一部分

`ce-compound` 是多个 workflow 的 closing loop。任何 verified、non-trivial fix 后都可以调用。常见时机：

- Successful debug 和 PR 后，bug 具有可泛化价值
- Shipping 后，工作产生了可复用 pattern、convention 或 tooling decision
- 独立 problem-solving session 结束后

输出会反馈给上游 skills：

- `/ce-plan` 在 Phase 1 research 中读取 `docs/solutions/`
- `/ce-ideate` 把它作为 grounding 的一部分读取

如果新 learning 暗示旧 doc 可能 stale，`ce-compound` 会带 narrow scope hint 推荐 `/ce-compound-refresh`。

---

## 独立使用

Skill 自己就是完整 cycle：

- 刚解决的问题：`/ce-compound`（也可能由“that worked”自动触发）
- 带 context hint：`/ce-compound "the email digest race condition we fixed"`
- 长 session 下的 Lightweight：context 紧张时，skill 自己选择 lightweight mode 并说明
- 低开销无人值守 capture：`/ce-compound mode:non-interactive depth:lightweight "the verified fix"`
- 完整无人值守 capture：`/ce-compound mode:non-interactive depth:full "the verified fix"`（裸 `mode:non-interactive` 等价）

Auto-invoke trigger 会在 conversation 中途发生。刚确认某件事有效时，不需要专门记得 slash command。

---

## 让 Capture 自动发生

Auto-invoke trigger phrase（“that worked”“it's fixed”）只会在你恰好说出这些话时生效。如果总是忘记 capture，可以在 agent instruction file 里加 standing instruction，让 agent 在 fix 验证后、最终 handoff 前主动提议 capture。

可以放到 repo 的 `AGENTS.md`/`CLAUDE.md`，或全局 instruction file（`~/.claude/CLAUDE.md`、`~/.codex/AGENTS.md`），让所有 repo 都生效。根据你希望有多强 checkpoint，选择对应版本：

**先 offer**（agent capture 前询问）：

> After a solved, verified problem produces a non-trivial, reusable learning, offer once, before the final handoff, to invoke the `ce-compound` skill. Only in repositories that accept `docs/solutions/` as a tracked knowledge store.

**自动运行**（不 prompt）：

> After a solved, verified problem produces a non-trivial, reusable learning, automatically invoke the `ce-compound` skill, passing `mode:non-interactive` as the skill argument. Only in repositories that accept `docs/solutions/` as a tracked knowledge store.

如果 standing workflow 接受更少 research/validation，换取 single-pass、no-subagent closure，可以使用 `mode:non-interactive depth:lightweight`。

Auto-run 会不询问地写入 `docs/solutions/`（也可能碰 `CONCEPTS.md`）。Non-interactive 从不编辑 `AGENTS.md`/`CLAUDE.md`。如果 discoverability 缺失，会报告 `gap noted, not applied`，让之后 interactive run 获得 consent 后再处理。把 `mode:non-interactive` 作为 argument 是最明确形式。Skill 也理解清楚的“run headless / without prompts”请求，但 token 可以消除歧义。没有 non-interactive signal 时，run 保持 interactive，并可能停在一次性的 discoverability-consent prompt。

Standing lines 里有几处 wording 是 load-bearing：

- 写“invoke the `ce-compound` skill”，不要写“run `/ce-compound`”：instruction files 会被你实际使用的 agent 读取，而 slash-command form 并非跨所有 agent 都能可靠调用
- 写“before the final handoff”，不要写“at the end of the session”：agent 无法可靠判断 session 何时真正结束，但知道自己何时准备把 verified result handoff 给你
- 写“non-trivial, reusable learning”：门槛是值得未来重读的 generalizable insight，不是一次昂贵但毫无复用价值的 one-off
- 写“repositories that accept `docs/solutions/`”：真正问题是 repo 是否欢迎 generated learning docs；fork 或你参与贡献的 open-source project 往往不欢迎

---

## 输出 Artifact

```text
docs/solutions/[category]/[filename].md
```

Categories 自动检测。Bug-track 示例：`build-errors/`、`test-failures/`、`runtime-errors/`、`performance-issues/`、`database-issues/`、`security-issues/`、`ui-bugs/`、`integration-issues/`、`logic-errors/`。Knowledge-track 示例：`architecture-patterns/`、`design-patterns/`、`tooling-decisions/`、`conventions/`、`workflow-issues/`、`developer-experience/`、`documentation-gaps/`、`best-practices/`。

文档带 YAML frontmatter（`module`、`tags`、`problem_type` 等），便于搜索。Validation 会通过 `scripts/validate-frontmatter.py` 捕获 silent corruption；`scripts/validate-doc-claims.py` 会对照 tree 检查正文 cited paths、SHAs、links 和 drafting scaffold。

Interactive Full mode 中，如果 discoverability check 发现 knowledge store 没被暴露，而且你 consent，skill 还可能对 `AGENTS.md`/`CLAUDE.md` 做一处很小的 edit。Non-interactive 和 lightweight 永远不应用这种 edit。

---

## 参考

| 参数 | 效果 |
|----------|--------|
| _(empty)_ | 使用 conversation context 记录最近的 verified fix |
| `<brief context>` | 聚焦 capture（例如“the email digest race condition we fixed”） |
| `mode:non-interactive` | 无人值守：没有 blocking questions。默认 Full。Deprecated alias：`mode:headless`。 |
| `depth:lightweight` | 仅 non-interactive。Single-pass workflow：无 subagents、无 overlap research、无 session-history probe。 |
| `depth:full` | 仅 non-interactive。完整 workflow，包括自动 session-history probe。 |

Auto-invoke triggers：conversation 中任意位置出现“that worked”“it's fixed”“working now”“problem solved”等 phrase。

单独要求创建/bootstrap `CONCEPTS.md` 会重定向到 `ce-compound-refresh`。这里的 vocabulary capture 是记录真实 learning 的副作用，只限于该 learning 所在领域。

---

## FAQ

**为什么有两种 mode，而且不问我选哪个？**
Full mode 适合绝大多数情况：并行 subagents 能找 duplicates、相关 docs，并运行 specialized reviews。Lightweight mode 用于简单 fix 或 context 很紧的 session。Skill 自己选择，而不是 prompt，因为决定因素（还剩多少 context budget）agent 能看到、用户看不到。选择会写在输出中。如果判断错了，重新运行成本低。

**Bug track 和 knowledge track 有什么区别？**
Bug track 捕获 incident-level fixes：“X 坏了，原因是这个，我们这样修。” Knowledge track 捕获 durable guidance：“这里我们应该这样做 X，以及原因。” Bug track 有 Symptoms / What Didn't Work / Solution；knowledge track 有 Context / Guidance / When to Apply。

**为什么自动更新 doc，而不是总是新建？**
两份描述同一个问题的 doc 会 drift。新的 context 更新，因此 skill 会把它 fold 进 existing doc，最终保留一份随时间改善的 canonical doc。

**一次 run 能捕获多个 learnings 吗？**
不能。Grounding、overlap detection 和 cross-referencing 都假设单一 solved problem。每个 learning 顺序运行一次。

**非软件场景能用吗？**
Knowledge track 可以泛化到 conventions、decisions、workflow practices，但 skill 假设存在 code repo、`docs/solutions/` directory 和 YAML-frontmatter conventions。它主要是 software-team tool。

**如果我不想让它编辑 AGENTS.md 做 discoverability 呢？**
Interactive Full mode 在应用 edit 前会询问 consent。拒绝后 doc 仍照常写。Non-interactive 和 lightweight 永远不会编辑 instruction file，只会报告或提示 gap。如果你的 AGENTS.md 已提到 `docs/solutions/`，discoverability prompt 根本不会触发。

**有“What's next?” menu 吗？**
没有。Skill 在 summary 后结束。Cross-doc maintenance 通过 refresh recommendation line 延后给 `ce-compound-refresh`。

---

## 另请参阅

- [`ce-compound-refresh`](./ce-compound-refresh.md)：随着 codebase 演进维护 `docs/solutions/`；同时负责 repo-wide `CONCEPTS.md` bootstrap
- [`ce-debug`](./ce-debug.md)：常见 capture 时机，fix 验证后
- [`ce-work`](./ce-work.md)：常见 capture 时机，shipping 后
- [`ce-plan`](./ce-plan.md)：planning 中把 `docs/solutions/` 作为 institutional memory 读取
- [`ce-ideate`](./ce-ideate.md)：把 `docs/solutions/` 作为 grounding 的一部分读取