---
name: codex-managed-worktree-threads
description: Explicit-only workflow for coordinating one Codex App background thread per write implementation task in Codex-managed worktrees. Supports the Groundwork PRD -> Issues -> per-issue child Goal Mode flow. Use only when the user invokes $codex-managed-worktree-threads. Do not use for read-only review-only work, natural-language-only parallel work requests, ordinary subagent delegation, manual git worktrees, tmux workflows, Codex CLI worker orchestration, or vague requests to work in parallel.
compatibility: Requires Codex App background-thread tools discoverable through tool_search; optional model and reasoning routing applies only when thread tools expose those controls. Not intended for CLI/IDE-only sessions, manual git worktrees, tmux, or Codex CLI worker orchestration.
---

# Codex Managed Worktree Threads

Coordinate parallel implementation tasks through Codex App background threads, letting the Codex App create and manage the worktrees.

For the Groundwork flow, treat `to-prd` output as product context and each `to-issues` issue as a candidate child goal. Only write implementation issues should become Codex App worktree threads. Read-only tasks such as multi-perspective review, scoring, critique, planning, or issue triage must be routed outside Codex-managed worktree creation.

This skill is a coordination workflow. It must not implement the child tasks in the main thread, create manual worktrees, replace full source-of-truth task context with summaries, or enter Goal Mode in the main thread.

## Operating Boundaries

- Proceed only when the user explicitly invoked `$codex-managed-worktree-threads`.
- If this skill is loaded without explicit invocation, tell the user to invoke `$codex-managed-worktree-threads` and stop.
- Use Codex App thread tools after discovering them with `tool_search`. Relevant tool names may include `create_thread`, `list_threads`, `read_thread`, `send_message_to_thread`, `set_thread_title`, `set_thread_pinned`, and `set_thread_archived`.
- Codex App background worktree threads are for write implementation tasks only. Do not create a Codex-managed worktree thread for read-only review, multi-perspective critique, scoring, planning-only work, issue triage, PRD review, or other tasks that should not edit files.
- For read-only tasks, use the main thread or a clean read-only reviewer from the provided package. Do not force the task through this worktree-thread workflow merely because it is part of a task matrix.
- When a child task should use Goal Mode, place the Goal Mode directive only inside the child-thread prompt. The main/coordinator thread must never start, announce, or execute Goal Mode for itself.
- Do not run `git worktree add`.
- Do not create replacement worktrees under `/private/tmp`, `.worktrees/`, `$CODEX_HOME/worktrees`, or any guessed path.
- Do not assume Codex-managed worktree paths are readable from the main thread or from clean reviewers.
- Do not use subagents for implementation. Use background Codex App threads for write implementation.
- Use a clean reviewer only for read-only review or scoring, and only from a review package, issue body, diff text, and validation output.
- Do not stage, commit, push, open PRs, close issues, archive threads, or change remote state unless the user explicitly asks.
- Treat UI visibility as non-authoritative. Verify thread delivery and status with `read_thread`.
- Redact secrets, private payloads, credentials, cookies, personal data, private URLs, customer-sensitive data, and unnecessary full logs from prompts and durable artifacts.

## Thread Tool Capabilities

Tool names may change, so discover tools with `tool_search` and map available tools to these capabilities.
This workflow requires Codex App background-thread capabilities. Do not use it in CLI-only or IDE-only sessions unless equivalent thread capabilities are discoverable.

| Capability | Required | Examples |
|---|---|---|
| Create a Codex App background thread | Yes, for write implementation | `create_thread` |
| Send the initial prompt or follow-up message | Yes, for write implementation | `create_thread`, `send_message_to_thread` |
| Read the thread timeline and latest status | Yes, for write implementation | `read_thread` |
| Name, list, pin, or archive threads | No | `set_thread_title`, `list_threads`, `set_thread_pinned`, `set_thread_archived` |
| Select a model for a child thread | No, use if exposed | `model`, `set_model`, `create_thread(model=...)` |
| Select reasoning or thinking effort | No, use if exposed | `reasoning_effort`, `thinking`, `create_thread(reasoning_effort=...)` |

If model or reasoning controls are unavailable, keep the execution profile in the child prompt and report that the tooling did not expose a hard selector.

## CHECKPOINTS

- 🔴 CHECKPOINT · 🛑 STOP: If the user did not explicitly invoke `$codex-managed-worktree-threads`, do not use this workflow. Tell the user to invoke `$codex-managed-worktree-threads`, then stop.
- 🔴 CHECKPOINT · 🛑 STOP: Before creating threads, show the task matrix and confirm that each write task has full source context, task type, child Goal Mode routing, execution profile, acceptance criteria, and a validation signal.
- 🔴 CHECKPOINT · 🛑 STOP: Before creating threads, remove or separately route every read-only or planning-only task from the Codex-managed worktree creation set.
- 🔴 CHECKPOINT · 🛑 STOP: Before archiving threads, staging, committing, pushing, opening PRs, closing issues, or changing remote state, ask for explicit user approval for that exact action.

## Groundwork PRD -> Issues -> Child Goals

When the user has already used Groundwork:

1. Treat the PRD generated by `Groundwork to-prd` as product and acceptance context.
2. Treat each issue generated by `Groundwork to-issues` as a candidate task record and, for write implementation issues, as a candidate child goal.
3. Fetch the complete PRD, full issue body, relevant comments, linked files, and validation notes before creating any child thread.
4. Do not rely on issue titles alone. An issue is eligible for a child worktree thread only when it includes or can be paired with acceptance criteria, non-goals, relevant source context, and a validation signal.
5. Set `child_goal_mode: yes` for eligible write implementation issues by default. The child prompt must say that the issue is the child thread's goal.
6. Set `child_goal_mode: no` for read-only or planning-only issues and do not create a Codex-managed worktree thread for them.
7. If an issue is too broad, cross-cutting, or lacks validation, mark it `blocked` or `needs_split` and ask for decomposition before thread creation.

## Task Classification and Worktree Routing

Classify every candidate task before thread creation:

| Task type | Worktree thread? | Typical examples | Default routing |
|---|---:|---|---|
| `write_implementation` | Yes | feature issue, bug fix, tests, migration, repo docs/config updates | Codex App background thread with Codex-managed worktree |
| `read_only_review` | No | multi-perspective review, architecture critique, security review, scoring, PRD review | Main thread or clean read-only reviewer from a package |
| `planning_only` | No | issue triage, decomposition, implementation plan, risk analysis without edits | Main thread or read-only planning pass |
| `hybrid` | Split first | investigation plus likely code edits | Run the read-only part first; create a worktree thread only for the resulting write task |

If the user explicitly asks for multiple read-only perspectives, create reviewer perspectives in the main thread or through read-only delegation if available. Do not use Codex-managed worktree threads unless the task may edit files.

## Execution Profile Routing

Assign an execution profile per task before thread creation. User-provided model or thinking preferences override these defaults.

Each task record must include:

- `task_type`
- `child_goal_mode`: `yes` or `no`
- `model_profile`: requested model class or concrete model if the user/tooling specifies one
- `reasoning_effort`: `low`, `medium`, or `high` when the tool supports it, otherwise a prompt-level preference
- `cost_latency_bias`: `fast`, `balanced`, or `quality`
- `routing_reason`: one sentence explaining the assignment

Default routing guidance:

| Task shape | Model profile | Reasoning effort | Cost/latency bias |
|---|---|---|---|
| Small localized bug, simple test/doc/config change | fast coding model | low or medium | fast |
| Normal feature issue with clear acceptance criteria | balanced coding model | medium | balanced |
| Cross-cutting feature, schema/API/migration, concurrency, security, data correctness, or unclear architecture | strongest coding/reasoning model available | high | quality |
| Clean review of high-risk or cross-task changes | strongest reviewer/reasoning model available | high | quality |
| Read-only multi-perspective review | no worktree; assign explicit reviewer perspectives | medium or high based on risk | balanced or quality |

If thread creation supports model or reasoning parameters, pass them through the tool call. If it does not, include the execution profile in the child prompt and report the selector as unavailable in the status table.

## Workflow

1. Confirm the request is explicit.
   If the request is ambiguous or only asks for generic parallel work, stop at the explicit-invocation checkpoint.
2. Discover thread tools with `tool_search`.
   If required write-thread capabilities are unavailable, report the missing capabilities, the discovered tool names, and why no discovered tool satisfies each missing capability; then stop.
   Also discover whether model and reasoning controls are available. These are optional, but the availability affects execution-profile enforcement.
3. Expand Groundwork context when present.
   Read the PRD and full generated issues. Convert each issue into a candidate task record, preserving PRD links, issue IDs, acceptance criteria, non-goals, and validation notes.
4. Classify every candidate task as `write_implementation`, `read_only_review`, `planning_only`, or `hybrid`.
   Route read-only and planning-only tasks outside Codex-managed worktree creation. Split hybrid tasks before creating write threads.
5. Parse the write-task matrix into stable records:
   - `task_id` or issue ID
   - short label
   - task type
   - required action
   - child Goal Mode routing
   - execution profile: model profile, reasoning effort, cost/latency bias, routing reason
   - thread title: `{task_id} · {short_label} · {required_action}`
   - source-of-truth location
   - expected touched files, modules, routes, schemas, public APIs, generated artifacts, and shared test fixtures
   - acceptance criteria
   - fastest relevant validation signal
   Keep thread titles at or below 80 characters. Put `task_id` first, use a business noun for `short_label`, truncate the action if needed, and keep titles free of secrets, credentials, tokens, private URLs, customer-sensitive payloads, names, and personal data.
6. Run conflict preflight before thread creation.
   If two tasks may touch the same file, route, schema, migration, public API, generated artifact, shared test fixture, or cross-task contract, do not run both as write tasks in parallel without explicit user approval.
   Prefer serializing one task, converting one task to read-only review or planning, or assigning an explicit merge order with revalidation after each merge.
7. Fetch complete source context before thread creation, then construct a redacted, self-contained source package for each child thread.
   For GitHub issues, read the full issue body and relevant comments. Do not rely on titles, summaries, or issue lists.
   Do not paste raw secrets, credentials, cookies, private URLs, customer-sensitive payloads, personal data, or unnecessary full logs into child prompts.
   If raw source is sensitive or too long, include exact relevant excerpts plus omitted-context notes and why the excerpt is sufficient.
8. Stop at the thread-creation checkpoint.
   Show the complete routing matrix, including read-only tasks that will not receive worktrees, write tasks that will receive child threads, `child_goal_mode`, execution profiles, conflicts, and validation signals.
9. After explicit approval, create one Codex App background thread per write implementation task only.
   Apply model and reasoning selections through the thread tool if supported. If unsupported, include the execution profile in the prompt.
   Use this thread title fallback order:
   - If thread creation accepts a title, pass the task-matrix title during creation.
   - Else if `set_thread_title` is available, set the task-matrix title immediately after creation.
   - Else put `Expected thread title: {task_id} · {short_label} · {required_action}` as the first line of the child prompt and maintain a main-thread `thread_id -> expected_title` mapping in the status table.
   Each thread must receive enough context to work independently without guessing from a title.
10. Send each child prompt with:
    - thread title
    - redacted self-contained source package or complete non-sensitive task context
    - exact action and non-goals
    - child Goal Mode directive only when `child_goal_mode: yes`
    - execution profile: model profile, reasoning effort, and cost/latency bias
    - acceptance criteria
    - required validation command or expected validation signal
    - no subagents for implementation
    - no manual worktrees
    - no stage, commit, push, PR, or issue close
    - the current contents of `references/review-package-template.md` as the required output schema
11. Monitor with `read_thread`.
    If a child thread appears idle or invisible in the UI, check the actual thread timeline before resending.
12. Require every child thread to return a review package:
    - The package must exactly use the current `references/review-package-template.md` shape.
    - The package must be self-contained, redacted, and include validation evidence.
    - The package must include thread identity, task source, task type, child Goal Mode status, execution profile, acceptance coverage, diff evidence, validation evidence, risks, blockers, and suggested merge order.
    - Child threads must not mark themselves `review_passed`; only the main thread may assign that state after clean review.
13. Run a clean review round from artifacts, not from managed worktree paths.
    Review against the redacted source package, diff text, validation evidence, and review package.
14. Summarize in the main thread with merge order and risk.
15. Archive background threads only after the user asks for cleanup and passes the remote-state checkpoint.

## Templates

Load only the template needed for the current step:

- Use `references/thread-prompt-template.md` when creating or messaging child threads.
- Use `references/review-package-template.md` when asking a child thread for final output or preparing a clean review.
- Use `references/rationale.md` only when boundary disputes arise, such as whether this workflow should use manual worktrees, tmux workers, Codex CLI workers, subagents, or read-only tasks without worktrees.

## Status Summary

Report orchestration status in this shape:

| Task | Type | Thread / Reviewer | Goal mode | Execution profile | State | Changed files | Validation | Clean review | Risks | Merge order |
|---|---|---|---|---|---|---|---|---|---|---|

`Thread / Reviewer` must include the actual thread identifier for write implementation threads, the expected title, and the latest `read_thread` state when available. For read-only tasks routed outside worktree creation, use `no_worktree_needed` plus the reviewer or main-thread source. This keeps title fallback, UI invisibility, resend decisions, partial-create reconciliation, and read-only routing tied to the same task identity.

Use these transient state values before clean-review decision:

- `planned`
- `routed_read_only`
- `awaiting_user_approval`
- `thread_created`
- `in_progress`
- `awaiting_review_package`
- `awaiting_clean_review`

Use these review and merge-decision state values:

- `ready_for_clean_review`: complete review package and validation passed, or validation is not applicable and the reason is reviewable.
- `review_passed`: clean review passed from the review package and validation evidence.
- `needs_remediation`: validation failed or clean review found a blocker.
- `blocked`: source context, thread delivery, review package evidence, required tooling, task splitting, or routing evidence is still incomplete after the allowed recovery step.

Only `review_passed`, `needs_remediation`, and `blocked` are terminal states. Only `review_passed` write tasks are merge-ready.
Use `not_run` for `Clean review` when a clean review could not run, and state the blocker.
Use `pass`, `pass_with_notes`, `blocker`, or `not_run` for `Clean review`; do not report numeric review scores unless the user explicitly asks for a separate scoring rubric.
Do not report the whole orchestration as complete unless every task is `review_passed`; group partial results by `review_passed`, `needs_remediation`, and `blocked`.

## Failure Handling

- If thread tools are missing for write implementation, stop and report which capabilities are unavailable.
- If model or reasoning selectors are missing, continue only when write-thread tools exist; record model/reasoning as prompt-level preferences rather than hard settings.
- If source context is incomplete, fetch it or ask the user for it before creating child threads.
- If an issue lacks acceptance criteria or validation, do not create a child worktree thread; mark it `blocked` or `needs_split`.
- If a read-only or planning-only task was about to receive a worktree thread, stop, reroute it, and update the task matrix before any thread creation.
- If some threads were created or messaged before a later create/send failure, list the created threads, reconcile their latest state with `read_thread`, and stop before creating duplicates.
- If a child thread worked from a title or summary only, treat the output as untrusted and ask it to re-check against the complete source context.
- If a child thread entered non-child Goal Mode, broadened beyond its issue goal, or ignored its execution profile, ask it to restate the task boundaries and redo only the focused child goal.
- If a child thread returns an incomplete review package, request exactly the missing fields once before clean review.
- If the package is still incomplete after one missing-field request, mark that task `blocked` and do not run clean review for it.
- If validation is skipped because of missing dependencies, permissions, unavailable tools, or environment failure, mark the task `blocked` unless the review package shows validation is not applicable to the change.
- If validation fails for a child task, mark that task `needs_remediation`; continue clean review only for tasks with complete review packages, and keep failed tasks out of merge order until remediated.
- If some tasks are complete and others are blocked or need remediation, summarize them separately in the status table and ask before sending any follow-up remediation request.
- If the main thread cannot access a managed worktree path, request a review package from the child thread instead of trying to inspect the path directly.
- If clean review finds a blocker, summarize the blocker and ask the user whether to send a focused remediation request back to that child thread.
- Any remediation request must quote the specific blocker, preserve the original non-goals, require only the narrow fix for that blocker, and require rerunning the relevant validation.
- If `send_message_to_thread` appears successful but no UI bubble appears, verify with `read_thread` before retrying.
- If `set_thread_archived` succeeds, describe it as archive only. Do not claim the sidebar entry or underlying session was hard-deleted.

## Do Not Assume

- Do not assume "worktree" means shell `git worktree`.
- Do not assume every issue needs a worktree; read-only and planning-only tasks do not.
- Do not assume tmux panes, Codex CLI workers, or external worktree managers are equivalent to Codex App background threads.
- Do not assume background thread paths are portable across threads.
- Do not assume clean reviewers can read child worktree files.
- Do not assume issue titles are acceptance criteria.
- Do not assume a Goal Mode instruction belongs to the main thread; Goal Mode directives are child-prompt payload only.
- Do not assume model or reasoning selection exists unless the thread tools expose it.
- Do not assume archive immediately removes a thread from the visible sidebar.
