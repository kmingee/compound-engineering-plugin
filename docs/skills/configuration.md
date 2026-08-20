# Compound Engineering 配置

Compound Engineering 把可选的仓库默认值保存在 `.compound-engineering/config.yaml` 中。普通 key 也可以写在 `.compound-engineering/config.local.yaml` 中；后者按 key 覆盖仓库文件。只要支持的 harness 打开的是同一个 checkout，这两个文件都对它可见。

运行 `/ce-setup` 可以创建 `config.yaml`，并刷新已提交的 `.compound-engineering/config.example.yaml`。Setup 不会创建 `config.local.yaml`。只取消注释你确实想修改的 key。不要在这两个文件中放 credentials、CLI commands 或 harness flags。

## Key 的解析方式

- **普通 keys：**先读 `config.local.yaml`，再读 `config.yaml`。第一个启用（非注释）的值生效。缺失的文件会跳过。无效或空的 scalar 会继续查下一层，最后回退到 skill 默认值。只要 list 或 map 存在（包括空值），就会整体替换该 key。
- **`docs_root`：**只从 `config.yaml` 读取。`config.local.yaml` 中的 `docs_root` 会被忽略。
- **Gitignore 不会改变解析规则。**无论文件被忽略还是提交，解析方式都一样。
- 当前 task 的明确指令仍然优先于 config。已经在上下文中的 session 和 project instructions 也可以覆盖或缩小它的作用范围。

<a id="artifact-root"></a>
## Artifact 根目录

默认情况下，所有由 CE 写入的 artifact 目录都位于 `docs/` 下——例如 `docs/plans/`、`docs/solutions/` 等。对于 `docs/` 已经被其他内容占用的项目（例如 Obsidian vault 或 docs site），`docs_root` 可以把这个根目录迁移到仓库内任意相对路径。未设置时，行为与当前默认实现逐字节一致。

只在受版本控制的 `config.yaml` 中设置 `docs_root`，这样每个 clone 和 worktree 都会共享同一棵 artifact tree。

另外两点使 `docs_root` 不同于其他设置：

- **它是仓库相对路径，并且会被校验。**该值必须解析到仓库内部的某个目录——不能是绝对路径，不能通过 `../` 或 symlink 逃出仓库，不能是仓库根目录本身，也不能位于 `.git/` 下。目录不存在时，会在首次写入时创建。
- **它采用 fail-closed。**如果 `docs_root` 不可用，skill 会报错并停止，因为静默回退到 `docs/` 会把 CE artifacts 写进你明确配置为不使用的位置。`/ce-setup` 会报告最终解析出的根目录。

`docs_root` 不会让 artifacts 脱离 ephemeral workspace 长期存在——根目录仍在仓库内，因此会随 checkout 一起存在或消失。

## Config 与 instructions 的关系

Config 是默认值，不是另一份 agent-instructions 文件：

- 当前 task 的直接指令优先于冲突的 config 偏好。
- Harness 已加载的 active session 和 project/user instructions 可以覆盖或收窄 config。根据 harness 不同，project instructions 可能来自 `AGENTS.md`、`CLAUDE.md` 或其他原生机制。
- 每个 skill 的 runtime contract 仍决定某个 setting 是否适用。例如，pipeline execution 会强制 planning artifacts 使用 Markdown，而 model elevation 只会在能够访问目标 model 的 harness 上生效。
- 某些 skills 会为自身路由定义更具体的偏好顺序；对应 skill 页面会记录该顺序。

已提交的 `config.yaml` 会在同一项目的 worktrees 之间共享。`config.local.yaml` 只对当前 checkout 生效。CE Work 会在创建 detached worker worktrees 之前解析 delegation，因此已选定的 route 会被带入该次执行。

## 配置项

所有设置都是可选的。被注释的示例只是文档，不是启用中的值。

| 使用者 | 配置项 | 用途与取值 |
|---|---|---|
| 所有会写 artifact 的 skills | `docs_root` | 所有 CE artifact 子目录所在的仓库相对文件夹。只能在 `config.yaml` 中设置。未设置 -> `docs`。详见 [Artifact 根目录](#artifact-root)。 |
| [`ce-ideate`](./ce-ideate.md), [`ce-brainstorm`](./ce-brainstorm.md), [`ce-plan`](./ce-plan.md) | `ideate_output`, `brainstorm_output`, `plan_output` | Artifact 格式：`md` 或 `html`。默认 ideation 使用 HTML，brainstorm/plan 使用 Markdown。Pipeline 上下文会强制 Markdown。 |
| [`ce-plan`](./ce-plan.md) | `plan_skip_scoping_confirm` | `true` 会跳过通常的 pre-plan scope confirmation；默认 `false`。它不会压制真正的 blocker，也不会隐藏 post-plan menu。 |
| [`ce-plan`](./ce-plan.md), [`ce-brainstorm`](./ce-brainstorm.md) | `plan_model`, `brainstorm_model` | Model elevation：把 reasoning-heavy 步骤发送给指定 model（例如 `fable`、`opus`），而不是 session model。值是 model alias；prompt 中的请求或 orchestrator 的 `plan_model:<alias>` carrier（例如来自 `lfg`，即使在 pipeline mode 也会遵守）优先于它。在所有 harness 上都可生效——宿主原生提供 model 时直接使用，否则通过 Claude CLI，再否则 inline。默认不设置（关闭 elevation）。 |
| [`ce-work`](./ce-work.md), [`lfg`](./lfg.md) | `work_engine_mode`, `work_engine_preferences` | 有序的 implementation-author 偏好。Mode 为 `off`、`prefer` 或 `require`；每个 entry 包含 `harness` 和可选 `model`。详见[实现路由](#implementation-routing)。 |
| [`ce-code-review`](./ce-code-review.md), [`ce-doc-review`](./ce-doc-review.md) | `cross_model_review_mode` | 控制自动 cross-model pass 是否可以把 review 内容发送给第二个 provider：`auto`（默认，保持当前行为）或 `off`。`off` 会在解析任何 peer 或 route 之前生效，同时保留所有本地 reviewers 和本地 adversarial fallback；报告原因会写成“disabled by checkout config”，而不是 route 不可用。对话中直接请求 peer 会在本次运行覆盖 `off`；对话中的禁止则覆盖 `auto`。 |
| [`ce-code-review`](./ce-code-review.md), [`ce-doc-review`](./ce-doc-review.md) | `cross_model_peer` | 首选 cross-model review 目标：`codex`、`claude`、`grok`、`cursor` 或 `composer`。Review skills 仍会执行 host-independence 与 route-availability gates。 |
| [`ce-code-review`](./ce-code-review.md), [`ce-doc-review`](./ce-doc-review.md) | `cross_model_model`, `cross_model_effort` | 固定已解析 peer target 的 model（例如 alias `fable`，或完整 id `claude-opus-5`，并且必须与 target 属于同一 family）以及 reasoning effort（claude `low`..`max`、codex `minimal`..`xhigh`、grok `low`..`high`；cursor-agent routes 不接受 effort）。未设置时保持 skills 自己的 editorial mapping。Peer 无法满足某个值时，会说明原因并跳过该 pass，而不是擅自替换；对话中的请求优先于两项配置。 |
| [`ce-commit-push-pr`](./ce-commit-push-pr.md) | `pr_teaching_section`, `pr_teaching_archive`, `auto_babysit` | 开关 PR concept teaching、选择是否归档 explainer，或关闭默认 babysit handoff。默认分别为 `true`、`false`、`true`。 |
| [`ce-product-pulse`](./ce-product-pulse.md) | `pulse_product_name`, `pulse_lookback_default`, `pulse_primary_event`, `pulse_value_event`, `pulse_completion_events` | Product identity、报告窗口，以及代表 engagement、value 和 completion 的 events。Setup interview 会写入这些值。 |
| [`ce-product-pulse`](./ce-product-pulse.md) | `pulse_quality_scoring`, `pulse_quality_dimension`, `pulse_analytics_source`, `pulse_tracing_source`, `pulse_payments_source`, `pulse_db_enabled` | 可选 quality scoring 与只读 data-source 路由。 |
| [`ce-product-pulse`](./ce-product-pulse.md) | `pulse_metric_sources`, `pulse_pending_metrics`, `pulse_excluded_metrics` | 每个 metric 的 source override，以及应该显示为 pending 或直接排除的 strategy metrics。 |
| [`ce-promote`](./ce-promote.md) | `ce_promote_spiral_optout` | `true` 会关闭一次性的 Spiral setup offer；删除该 key 即可再次启用。 |
| [`ce-sweep`](./ce-sweep.md) | `feedback_sources`, `sweep_state_path`, `sweep_ack_cap`, `sweep_lease_ttl_minutes`, `sweep_shared_branch` | Feedback connectors、持久 state 位置、acknowledgment circuit breaker、lease 过期时间，以及可选的、受 push gate 约束的 shared-branch coordination。Setup interview 会写入这些值。 |

<a id="implementation-routing"></a>
## 实现路由

Work engine 列表是相对于宿主的，而不是绑定到当前 checkout 平时使用的 harness：

```yaml
work_engine_mode: prefer
work_engine_preferences:
  - harness: cursor
    model: composer
  - harness: codex
    model: "gpt-5.6"
  - harness: claude
```

支持的 harnesses 为 `codex`、`claude`、`grok` 和 `cursor`。省略 `model` 时使用该 harness 配置的默认 model。Composer 是通过 Cursor 访问的 model family，因此应使用 `harness: cursor` 和 `model: composer` 请求。

`ce-work` 会按顺序遍历列表，并跳过与当前 host/default model 等价的 entry。同一 harness 中显式指定的不同 model 仍然符合条件。使用 `prefer` 时，如果列表中的 route 都不可用，会在明确说明后回退到原生 implementation。使用 `require` 时，交互式 CE Work 会在弱化 route 前询问，而 LFG 和其他 headless callers 会直接阻塞。

当前 task 的文字指令可以只针对本次运行选择不同 route，而无需修改 config，例如“use Codex for implementation”或“only use Composer for implementation”。该 assignment 只作用于 implementation；validation、integration、commits 以及调用方工作流中的其他环节仍由宿主负责。

## 安全维护

- 如果希望团队共享默认值，请 commit `config.yaml`。如果 `config.local.yaml` 只保存个人或 checkout-specific 选择，请不要提交它（`/ce-setup` 可以添加 `.compound-engineering/*.local.yaml`）。
- 持久的团队级 *instructions* 应放在项目通常使用的 agent-instructions 机制中；CE keys 的团队 *defaults* 可以放在 `config.yaml`。
- 一次性选择优先使用 per-run instructions。
- Plugin 升级后重新运行 `/ce-setup`，以刷新已提交的 example，并诊断已废弃或格式错误的设置。
