# 概念

本项目共享的领域词汇——包括实体、具名流程，以及在项目中有特定含义的状态概念。最初由核心领域词汇构成，之后会随着 `ce-compound` 和 `ce-compound-refresh` 处理经验而不断积累；也可以直接编辑。这里只是术语表，不是 spec，也不是包罗万象的说明文档。

## Plugin 及其组成部分

### Plugin
由一个 manifest 描述、可分发的一组 Skills、Agents、Commands 和 Hooks（可选包含 MCP servers），作为一个整体安装到 coding-agent 平台中——它也是 Converter 为非 Claude Targets 进行转换、Marketplace 负责分发的 artifact。

### Skill
定义在独立目录中、由用户调用的能力，也是用户最主要的入口。Skill 负责 orchestration：它可以按需逐步加载自己的 reference files，也可以 dispatch 使用 Specialist prompt assets 初始化的通用 subagents。它与 Agent 的区别在于：Skill 由用户调用并负责协调，而 Agent 或 subagent 被 dispatch 去完成范围明确的工作。

### Agent
在独立上下文中运行、专注单一用途并返回结果的 worker，而不是与用户对话。也称为 subagent。在当前 plugin 设计中，大多数 CE specialist 行为不会作为独立 Agent 定义暴露；Skills 会改为使用 Skill-local prompt material 初始化通用 subagents。

### Specialist prompt asset
由某个 Skill 拥有的内部 prompt 文件，用来为通用 subagent 定义 specialist persona，或 research/review 角色。它不是对外暴露的 plugin component：所属 Skill 决定何时加载、应用哪个 model 或 tool policy，以及如何合并输出。

## 转换

### Target
除 Claude Code 之外的目标 coding-agent 平台（OpenCode、Codex、Pi、Antigravity、Kimi Code 等）。仓库通过原生 plugin metadata，或 Converter/Writer 组合对它提供支持。当它走 conversion 路径时，也称为 target provider。

Plugin 会以两种 scope 之一安装到 Target：global（用户级）或 per-workspace。

### Native plugin surface
平台自身提供的安装 contract，可以直接消费本仓库已提交的 plugin manifest 或 marketplace metadata，无需生成转换后的 Bundle。当 Target 有原生 plugin surface 时，面向用户的支持通常应落在平台 metadata、release validation 和 docs，而不是新增 Converter 与 Writer。

### Converter
把解析后的 Plugin 转换成某个 Target 的内存表示的步骤；tools、permissions、hooks 和 model names 都要显式映射，而不是依赖约定。

### Writer
把某个 Target 的已转换 Bundle 写入磁盘、放到该 Target 期望的路径，并遵循其 merge semantics 的步骤。它与 Converter 成对存在，每个 Target 一套。

### Bundle
单个 Target 对应的 Plugin 内存转换形态——由 Converter 产生，再交给 Writer 消费。

### Install manifest
Writer 在安装时写入的 per-plugin ledger，精确记录该次安装在 Target 上创建了哪些 skill、agent、prompt 和 extension 路径——后续安装据此区分 tool-owned content 和 user-managed content。

最关键的不变量是：Writer 永远不能认领自己没有写入的路径。若用户替换了某个路径（例如指向个人 fork 的 symlink，或手写目录），该路径会从 manifest 中排除，重新安装时保留而不是覆盖；ledger 还是 self-healing 的——移除 override 后，下一次安装就会重新开始跟踪该路径。没有 manifest entry 的路径——包括在该机制出现之前由旧安装创建的路径——都会被视为无 owner，因此予以保留。

### Marketplace
用于分发的 catalog metadata，列出可安装 plugins 及其版本，并通过 release validation 与每个 Plugin 的 manifest 保持一致。

## Compound engineering

### Compound engineering
本项目所体现的方法论：组织工程工作，使每个工作单元都让下一个单元更容易；在过程中持续捕获可复用知识，让工具集越用越聪明。

### Pipeline
一条由 Skills 串联而成的进程，把一项工作从 strategy 和 ideation，推进到 brainstorm、plan、execution 和 review，最后以记录所学内容结束。每个阶段都会把持久 artifact 交给下一阶段；research 会在真正需要它的阶段收集，而不是在下游重复收集。

### Visual probe
Brainstorming 期间用于回答某个 shape、layout 或 relationship 问题的一次性、只用于展示的 decision sketch。用户看完后在聊天中回答。它不是 prototype，也不是 spec：如果粗略 sketch 无法解决某个决策——例如任何取决于真实 finish 或 motion 的问题——就应改用 experience prototype。

### Experience prototype
一次性的产品 prototype，目的是让人真正体验它——亲自操作，或看到接近真实完成度的表现——在选项进入 plan 和代码之前，决定某件事应该如何工作、呈现或表达。Modality、fidelity 和 medium 都遵循同一条规则：不要伪造正在被测试的维度。Throwaway 指“不维护、不交付”，而不是“必须删除”——scratch prototype 会尽可能保留下来，作为下一步实际构建内容的参考，并与做出的决策一起存在；但 in-app overlay run 会撤销，不留下任何痕迹。它不同于 visual probe（粗略、单一决策），也不同于 polish（针对已经可用的功能）。

### Learning
对过去问题的已记录解决方案——可能是 bug fix、convention 或 workflow pattern——作为复利知识的基本单元保存，让未来工作能够找到并复用。也称 solution doc。它带有结构化 metadata（category、tags、problem type）供检索；创建日期写在 entry 内，而不是文件名里。

### Pattern doc
从多个 Learnings 中抽象出的更通用指导。相比单个 incident-level Learning，杠杆更高；一旦过时，风险也更高，因为未来工作会把它当成普遍适用的规则。

### Knowledge track
Learning 所携带的两类 classification 之一，由 problem type 决定：knowledge track 保存 guidance——conventions、workflow patterns、practices、decisions——而 bug track 保存已经诊断出的 defects。Track 决定 Learning 必须带哪些 metadata，以及应该应用哪些 maintenance checks；与 procedure 形状有关的检查，例如把 Learning 与 Guidance layer 比较，会以 knowledge track 为判断依据。

### Guidance layer
Agent 在真正执行动作时加载的 agent-facing instructions——例如某个 skill 的 instructions、runbook、根级 instruction file。因为 agent 是在行动当下读取这些内容，所以与它冲突的 Learning 不只是“陈旧”，还可能在实际执行中直接被覆盖；因此这种 contradiction 的优先级高于普通 staleness。Maintenance skills 只会把 Learning 与它自己具名或链接的 guidance 比较，并根据当前代码实际遵循哪一方解决冲突；如果错误出在 guidance file，会报告它，而不是直接编辑。

### Explainer
为开发者本人编写的信息密集、视觉化 teaching artifact——解释某个概念、改动、想法，或自己最近的一段工作——使人在 agents 负责写代码时仍能持续学习。它与 Learning 互补：Learning 教未来的 repo 工作；explainer 教人。

### Session handoff
不可变的 continuity artifact，让一个新的 agent 无需先前 session transcript，也能恢复 objective、decisions、current state 和 unfinished work。CE 创建的 handoff 默认使用受管理的临时 Markdown，并指向权威 project artifacts，而不是取代它们。接收方 agent 也可以从任何由用户选择、且包含足够 continuity context 的来源恢复；选择只提供上下文，不代表获得自动继续工作的授权。

### Check-in
可以在同一 session 中跟在 explainer 后面的 active-recall 步骤：开发者先预测或作答，再由解释确认或纠正——对于 changes 使用 predict-then-reveal，对于 concepts 使用 checked exercises。如果内容不值得做 retention 练习，可以跳过。

### Concept-teaching section
生成 PR description 时可条件性加入的一个 section。当 agent 判断某次改动引入了 codebase 中的新概念时，会用它讲清概念是什么、为什么这里选择它，以及 PR 中的一个示例，让读者无需阅读 diff 也能理解并重新解释这次改动。它是 Explainer 在 PR description 中的被动对应物。

## Skill orchestration

### Dispatch skill
一种工作流依赖把任务委派给随 plugin 发布的 subagents——reviewers、scouts、fixers——而不是由 orchestrator 自己在单一上下文中完成每一遍处理的 Skill。因此它会携带共享 Skill-context directives，防止 harness 默认策略静默剥掉其流程所依赖的 delegation。

是否属于这一类，取决于工作流是否真的依赖 dispatch 发生——例如需要相互独立的上下文来提供有证据价值的一致性，或单一上下文无法提供所需隔离与覆盖。只有当“在自身上下文中直接完成被委派工作”本身就是 first-class path，而不是 degrade 时，一个 skill 才不属于这一集合：纯粹为了并行而存在的 delegation 可以用时间换掉，因此这种 skill 会在本上下文中顺序完成工作，并且不发布 directives。

### Skill-context directives
Dispatch skill 每次 invocation 开始时，以 tool output 形式发出的 counter-directive block——用来在 harness 默认会 gate agent 使用时授权其随附 subagents；禁止把 harness constraint 重新叙述成用户偏好；在 standing autonomy framing 下仍保留确认步骤；并拒绝把同一上下文中完成的不同 lens 工作计入 independence。把它作为 tool output 交付是机制本身，而不是 packaging 细节：同样的文本如果只是静态 skill prose，就无法压过 harness default；某个 Skill 上“prose 足够”的证据也不能迁移到其他 Skill。

### Model tier
对 dispatched sub-agent 的语义化成本分级——extraction（满足能力要求的最低成本，用于 retrieval 和 quoting）、generation（中档，用于 evidence-driven 工作和机械验证）或 ceiling（沿用 orchestrator 自身 model，通过不指定 model 来继承）——每个 Skill 只声明一次，之后通过 tier 名引用，因此 skill 内容里无需硬编码 model 名。

当平台无法为每个 agent 单独选 model 时，所有角色都运行在继承 model 上，成本控制则退回到结构层面：read budgets 和 output caps。

### Evidence dossier
大体量 evidence artifact——由低成本 scout agent 收集的逐字 quotes 及 source pointers——写入 scratch storage，而不是 inline 返回，使 orchestrator 只携带简短 gist，下游 agents 自己读取完整 dossier。

### Load stub
当某个 Skill 中 load-bearing 内容被移到 reference file 后，保留在 inline body 中的最小残余：一条 load instruction，说明 reference 包含什么，以及跳过它会导致什么 failure mode；同时不保留任何 agent 可以凭空补出的细节——这样加载 reference 会成为结构性必要条件，而不只是建议。

### Skill-eval cell
一个评分 scenario：在真实 coding-agent host 上运行某个 Skill，并根据该次运行最终留下的 artifacts 评分——执行过的 actions、写入的 files、以及 Skill 自己声明为“缺失则无法辩护”的 required reads——而不是看模型的 essay 有没有提到某条 command 或打开某个 procedure file。

只有当 always-loaded body 在缺少该文件时无法为决策辩护，required-read miss 才会让 cell 失败。如果 body 本身仍写明 gate，那么省略 probe 才是正确的 negative；真正衡量 extraction 的应是另一条 complementary cell，并且该路径确实由 reference 所有。

### Detached job
把 delegated worker process 启动到自己的 session 中，使其生命周期超过发起它的 shell tool call；其状态——status word、log、identity、result——保存在持久 job directory，orchestrator 会在不同 turn 之间轮询，而不是原地 await。

Launching call 在 job 创建后立即返回；supervision（idle/hard limits、process-tree reaping）在 detached worker 内运行，而 caller 保留自己的 aggregate deadline，超过后就不再等待该 job、继续向前。每个 job 只会以原子方式发布一个 terminal record；detached path 中的任何内容都不能 prompt 用户。Process-tree reaping 是宿主操作系统 process-grouping primitive 提供的保证，不是 job contract 可以凭空假定的保证：如果某种 grouping 无法比领头 process 活得更久，就必须基于能做到这一点的 primitive 重新推导 reaping，否则 descendants 会在 terminal record 之后继续存活。

Liveness 和 progress 是不同信号，idle window 只能检测被监控 stream 实际携带的那一种。Worker 发出的 heartbeat 能证明 supervising process 仍活着，却不能证明 delegate 正在产出；反过来，一个直到完成才 flush 输出的 delegate，看起来和卡死的 delegate 完全一样。某个 delegate 能提供哪类信号，是该 delegate 必须被测量的属性，不能先验假设；只有确认后，idle window 才能被信任用来区分正在工作的 run 和 stalled run。

### Cross-model pass
一种附加 delegated run：把宿主工作流的 review 或 judgment brief 通过不同 model-provider route 发送出去，再把结构化结果折回宿主 synthesis。当 peer 无法运行时，它保持 non-blocking；只有当实际 serving model family 能被验证，而不只是“被请求”，它才算 independent corroboration。

Peer result 只有在已经对所框定问题给出 settled answer 时才能使用——包括带原因、且已 settled 的 Blocked verdict。Settledness 必须由 peer 在 output contract 中显式声明，不能从 prose 推断；如果结果满足 schema，但没有声明 final，它只是 placeholder：可以在同一 time window 内、保持相同 target、model 和 scope，在同一路由上做一次有边界的 retry；如果再次出现，就按实际观察到的原因丢弃这一 voice，而不能把它当成一个立场参与 synthesis。

### Terminalize
由宿主负责的步骤：把已经完成的 external worker working tree 转换成一个可检查的 Transport commit，而不要求 worker 自己 stage 或 commit。

Snapshot 会包含 committed、uncommitted 和 untracked output。Worker 可以编辑和测试；只有 host 会创建 Transport commit，并在之后创建 canonical checkout commit。

### Transport commit
Host 根据 external worker 的完整最终 tree 构造出来的 synthetic、base-parented commit，供 host 检查并折入结果。它是中间证据，不是 canonical checkout commit，也永远不是 worker 自己的 tip。

### Warm checkout
一种 checkout，其 git-ignored inventory 已经包含项目 verification command 运行所需的内容：已安装 dependencies、virtualenvs、build caches。它是开发者 canonical checkout 的常见状态，与 fresh clone 或刚新增的 worktree 相反——后者必须先有人安装这些 artifacts，verification 才能运行。

Warm checkout 中被忽略的状态通常体量大、symlink 多，而且由 controller 从未运行过的 tooling 所拥有。因此 host 侧对这些内容所能作出的保证，只能是 detection 和 disclosure，而不可能是逐字节 custody。

### Model identity receipt
由实际 serving backend 自己报告“哪一个 model 真正处理了 delegated run”的凭据，与 requested model 一起记录，使两者不一致时能够直接看见。Run 的 model identity 只有通过这种 receipt 才算 verified——不能根据 request parameters 或 model 自己的文本判断——没有 receipt 的输出必须标记为 requested-but-unverified；对 cross-model agreement 进行加权的逻辑应跟随 receipt，而不是 request。

### Handoff seam
Calling Skill 中“当前工作完成后，在同一 run 触发 follow-on Skill”的位置——它与 Session handoff 不同，后者负责把 continuity 交给新的 session。如果 seam 只表达意图（例如“auto-invoke X”），caller 的 agent 就容易凭记忆重做 callee mechanics；加固后的 seam 会固定 invocation mechanism（使用平台的 skill-invocation primitive，从而真正加载 callee instructions），而且当 callee 运行 stateful protocol 时，还会明确禁止直接从外部启动该 protocol 的 mechanics。

### Context-absent agent
在没有加载某个 Skill instructions 的情况下执行 Skill-shaped action 的 agent——典型表现是凭半记忆重建 command，参数值偏离 Skill 文档中的默认值。未加载 Skill 中的 prose 无法影响它；真正能触达它的 channel 只有它进入时经过的 seam，以及它所运行 tools 的输出。这也是为什么 bundled CLIs 的 fail-closed refusal 必须自带 recovery path。

## Review 与工作流术语

### Reviewer persona
只采用单一 lens 的 reviewer 角色，从某个明确视角——security、correctness、scope、design 等——评估工作。Review Skills 会把一组 personas 作为 subagents dispatch，并合并它们的 findings。

### Confidence anchor
固定小范围上的离散自评 confidence 值，每一档都绑定模型可以诚实应用的行为标准，用来 gate 和排序 review findings，而不是使用容易制造虚假精度的连续分数。每个 review Skill 都设定自己的 actionable threshold；不同 personas 的 corroboration 可以把 finding 提升一档，但只有当这些 personas 满足 Independence 要求时才可以。

### Independence
Reviewer 或 researcher 运行所在的*执行上下文*属性，而不是它所采用 lens 的属性：只有来自分别 dispatch 的独立上下文，两条 findings 才算 independent。同一个上下文中由两个 personas 推理，只是两个视角，不是两个 witness。

只有这种意义上的 independence 才允许 corroboration——提升 Confidence anchor、计算 agreement，或描述结果为 independently confirmed。如果没有实际 dispatch、工作改为 inline 运行，findings 仍然有效，但 corroboration signal 不存在；run 应说明损失了哪些 coverage，而不是据此提升 confidence。

### Autofix class
按“建议修复能够多安全地应用”对 review finding 分类：静默应用、仅在用户确认后应用、留给人处理，或仅作为 advisory 记录而不执行动作。

### Rendering floor
一个与 surface 无关的统一 contract，规定 Skill 在所有输出 surface——interactive walkthrough、batch report、unattended envelope、one-line preview——中应如何呈现 review finding 供人决策。它固定 decision-first 字段顺序（先 recommendation 和通俗 consequence；mechanism 限长且放最后），并对 opaque tokens 采用领域无关策略：如果某个 identifier 必须打开被 review 的文档或代码才能理解，就按其功能补充说明（navigation、provenance 或 mechanism），或把它移出 decision block。每个 surface 只把自己的 layout 映射到这个 floor，而不是各自复制一份规则，因此强化某个 surface 时不会静默漏掉其他 surface。

### Headless mode
显式 opt-in 的无人值守模式，不向用户提问——它以书面 report 作为 deliverable，对于真正模糊的决策会保守 defer，而不是猜测。当 automation 需要明确 coverage tradeoff 时，Skill 可以在 headless mode 内额外暴露 depth selector；non-interactive contract 与 work depth 仍是两个独立决策。

### Session-settled decision
用户在调用对话中真正查看并选择过的决策——也就是 surfaced tradeoff 或 alternative 出现后，由用户做出选择——它会在 Pipeline 中以带 provenance label 的 constraint 传递（annotation stem `session-settled:`，classes `user-directed` 与 `user-approved`）。下游 skills 可以补充，但不会重新询问；只有有证据时才能反驳。未经审视的 assertion 是 directive，不是 settled decision，只会在 pipeline 内接受一次 challenge；agents 绝不能把自己的未经审视 proposals 标记成 settled。

### Settlement test
Writer skill（`ce-plan`、`ce-brainstorm`）对从对话带入的 decisions 所做的 classification judgment：如果 decision 在 conversation record 中经历过审视，则为 settled；如果只是被断言，则为 directive；如果只由 agent 推断且从未明确出现，则不打 label。Test 的 outcome rules 属于 protocol；实际 classification 本身仍是 agent judgment。

### Feedback source
配置好的 customer/user feedback 来源——Slack channel、GitHub Issues repo、email inbox——以通用 key 声明在仓库 CE config（`config.yaml`，可由 `config.local.yaml` 覆盖）中，因此任何 Skill 都能读取该列表。每个 source entry 都有自己的 identity 和 ingestion cursor；从中摄取内容的 Skill 负责 per-item state，而不是 source declaration 本身。

### Beta skill
稳定 Skill 的平行副本，名称追加 `-beta`，用于在不打扰用户的情况下并行试用新版本。只能手动调用（model auto-invocation 被关闭）；把它 promote 成 stable 不只是改名——所有 caller 必须在同一次 change 中一起迁移，避免任何一方静默继承 stale defaults；同时还必须把退役的 beta 名登记到 stale-artifact cleanup 中，防止升级后的用户仍保留一个已经失效、却与 promoted skill 并存的副本。

### Offered work
用户已经提交出来供 review 的工作，与“仅仅存在于 tree 或 remote 上”的工作不同。Open pull request 中的 commits 属于 offered；uncommitted edits、本地 commits，以及仅为备份或触发 CI 而 push 的 commits 都不属于 offered。

Shipping gate 在 publish 任何内容前检查的正是这个区别，而且 offered 并不等同于 pushed——push 只是把 bytes 移到 remote，review 才让工作变成 offered。因为负责 shipping 的 skill 会 push 整个 branch，而 pull request 会覆盖该 branch 上的每一个 commit，如果 gate 允许 unoffered work，就会把它和原本要求交付的改动一起 publish。

### Fix-owned files
某次 run 为修复它被调用来处理的 bug 而修改的 tests 和 implementation，与 run 开始前就已经 modified 的 files 区分开。

必须在任何 edit 之前记录，这样后续 phase 才能据此限定 scope：commit 只包含 fix-owned files，其他一律不带；quality pass 也会被明确交付该 scope，而不是笼统拿 branch diff，因为会重写输入的 pass 否则可能碰到从未 offered 的 work in progress。如果某个 fix-owned file 在 run 开始前就带有用户自己的 edits，那么 file-level commit 无法把两者拆开；这种 entanglement 是 handoff 唯一需要停下来询问的情况。

### Issue of record
用户作为 bug 入口提供的 tracker 或 monitor item，不论它来自哪套系统，都被视为该 bug 的 canonical record——error-monitor issue 与 tracker ticket 同等有效。

后续 phase 会链接它，而不是在其他地方为同一个 bug 再开第二条记录，也不会询问是否要这么做。发现项目自己的 tracker 只是为了读取过去工作，不是为了给 bug 建一个新 home。如果输入里没有此类引用，就表示没有 issue of record；这是普通状态，不是必须补上的缺口。

### Residual
某次 run 接受或 defer、而没有修复的 review finding。在 run 报告完成前，它必须进入一个持久 sink——例如 pull request body 中的 section，或项目 tracker 中的 ticket。只存在于 session 内的 finding 会随 session 结束而丢失，因此只要还有 accepted residual 没有被记录到人能找到的位置，就不能宣称 merge-ready。
