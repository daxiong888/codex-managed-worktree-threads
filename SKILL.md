---
name: codex-managed-worktree-threads
description: Explicit-only workflow for coordinating one Codex App background thread per task in Codex-managed worktrees. Use only when the user invokes $codex-managed-worktree-threads. Do not use for natural-language-only parallel work requests, ordinary subagent delegation, manual git worktrees, tmux workflows, Codex CLI worker orchestration, or vague requests to work in parallel.
compatibility: Requires Codex App background-thread tools discoverable through tool_search; not intended for CLI/IDE-only sessions, manual git worktrees, tmux, or Codex CLI worker orchestration.
---

# Codex Managed Worktree Threads

Coordinate parallel implementation tasks through Codex App background threads, letting the Codex App create and manage the worktrees.

This skill is a coordination workflow. It must not implement the child tasks in the main thread, create manual worktrees, or replace full source-of-truth task context with summaries.

## Operating Boundaries

- Proceed only when the user explicitly invoked `$codex-managed-worktree-threads`.
- If this skill is loaded without explicit invocation, tell the user to invoke `$codex-managed-worktree-threads` and stop.
- Use Codex App thread tools after discovering them with `tool_search`. Relevant tool names may include `create_thread`, `list_threads`, `read_thread`, `send_message_to_thread`, `set_thread_title`, `set_thread_pinned`, and `set_thread_archived`.
- Do not run `git worktree add`.
- Do not create replacement worktrees under `/private/tmp`, `.worktrees/`, `$CODEX_HOME/worktrees`, or any guessed path.
- Do not assume Codex-managed worktree paths are readable from the main thread or from clean reviewers.
- Do not use subagents for implementation. Use background Codex App threads for implementation.
- Use a clean reviewer only for read-only review or scoring, and only from a review package, issue body, diff text, and validation output.
- Do not stage, commit, push, open PRs, close issues, archive threads, or change remote state unless the user explicitly asks.
- Treat UI visibility as non-authoritative. Verify thread delivery and status with `read_thread`.
- Redact secrets, private payloads, credentials, cookies, personal data, private URLs, customer-sensitive data, and unnecessary full logs from prompts and durable artifacts.

## Thread Tool Capabilities

Tool names may change, so discover tools with `tool_search` and map available tools to these capabilities.
This workflow requires Codex App background-thread capabilities. Do not use it in CLI-only or IDE-only sessions unless equivalent thread capabilities are discoverable.

| Capability | Required | Examples |
|---|---|---|
| Create a Codex App background thread | Yes | `create_thread` |
| Send the initial prompt or follow-up message | Yes | `create_thread`, `send_message_to_thread` |
| Read the thread timeline and latest status | Yes | `read_thread` |
| Name, list, pin, or archive threads | No | `set_thread_title`, `list_threads`, `set_thread_pinned`, `set_thread_archived` |

## CHECKPOINTS

- 🔴 CHECKPOINT · 🛑 STOP: If the user did not explicitly invoke `$codex-managed-worktree-threads`, do not use this workflow. Tell the user to invoke `$codex-managed-worktree-threads`, then stop.
- 🔴 CHECKPOINT · 🛑 STOP: Before creating threads, show the task matrix and confirm that each task has full source context, acceptance criteria, and a validation signal.
- 🔴 CHECKPOINT · 🛑 STOP: Before archiving threads, staging, committing, pushing, opening PRs, closing issues, or changing remote state, ask for explicit user approval for that exact action.

## Workflow

1. Confirm the request is explicit.
   If the request is ambiguous or only asks for generic parallel work, stop at the explicit-invocation checkpoint.
2. Discover thread tools with `tool_search`.
   If required thread capabilities are unavailable, report the missing capabilities, the discovered tool names, and why no discovered tool satisfies each missing capability; then stop.
3. Parse the task matrix into stable records:
   - `task_id` or issue ID
   - short label
   - required action
   - thread title: `{task_id} · {short_label} · {required_action}`
   - source-of-truth location
   - expected touched files, modules, routes, schemas, public APIs, generated artifacts, and shared test fixtures
   - acceptance criteria
   - fastest relevant validation signal
   Keep thread titles at or below 80 characters. Put `task_id` first, use a business noun for `short_label`, truncate the action if needed, and keep titles free of secrets, credentials, tokens, private URLs, customer-sensitive payloads, names, and personal data.
4. Run conflict preflight before thread creation.
   If two tasks may touch the same file, route, schema, migration, public API, generated artifact, shared test fixture, or cross-task contract, do not run both as write tasks in parallel without explicit user approval.
   Prefer serializing one task, converting one task to read-only review or planning, or assigning an explicit merge order with revalidation after each merge.
5. Fetch complete source context before thread creation, then construct a redacted, self-contained source package for each child thread.
   For GitHub issues, read the full issue body and relevant comments. Do not rely on titles, summaries, or issue lists.
   Do not paste raw secrets, credentials, cookies, private URLs, customer-sensitive payloads, personal data, or unnecessary full logs into child prompts.
   If raw source is sensitive or too long, include exact relevant excerpts plus omitted-context notes and why the excerpt is sufficient.
6. Stop at the thread-creation checkpoint.
   After explicit approval, create one Codex App background thread per task.
   Use this thread title fallback order:
   - If thread creation accepts a title, pass the task-matrix title during creation.
   - Else if `set_thread_title` is available, set the task-matrix title immediately after creation.
   - Else put `Expected thread title: {task_id} · {short_label} · {required_action}` as the first line of the child prompt and maintain a main-thread `thread_id -> expected_title` mapping in the status table.
   Each thread must receive enough context to work independently without guessing from a title.
7. Send each child prompt with:
   - thread title
   - redacted self-contained source package or complete non-sensitive task context
   - exact action and non-goals
   - acceptance criteria
   - required validation command or expected validation signal
   - no subagents for implementation
   - no manual worktrees
   - no stage, commit, push, PR, or issue close
   - the current contents of `references/review-package-template.md` as the required output schema
8. Monitor with `read_thread`.
   If a child thread appears idle or invisible in the UI, check the actual thread timeline before resending.
9. Require every child thread to return a review package:
   - The package must exactly use the current `references/review-package-template.md` shape.
   - The package must be self-contained, redacted, and include validation evidence.
   - The package must include thread identity, task source, acceptance coverage, diff evidence, validation evidence, risks, blockers, and suggested merge order.
   Child threads must not mark themselves `review_passed`; only the main thread may assign that state after clean review.
10. Run a clean review round from artifacts, not from managed worktree paths.
   Review against the redacted source package, diff text, validation evidence, and review package.
11. Summarize in the main thread with merge order and risk.
12. Archive background threads only after the user asks for cleanup and passes the remote-state checkpoint.

## Templates

Load only the template needed for the current step:

- Use `references/thread-prompt-template.md` when creating or messaging child threads.
- Use `references/review-package-template.md` when asking a child thread for final output or preparing a clean review.
- Use `references/rationale.md` only when boundary disputes arise, such as whether this workflow should use manual worktrees, tmux workers, Codex CLI workers, or subagents.

## Status Summary

Report orchestration status in this shape:

| Task | Thread | State | Changed files | Validation | Clean review | Risks | Merge order |
|---|---|---|---|---|---|---|---|

`Thread` must include the actual thread identifier, the expected title, and the latest `read_thread` state when available. This keeps title fallback, UI invisibility, resend decisions, and partial-create reconciliation tied to the same thread identity.

Use these transient state values before clean-review decision:

- `planned`
- `awaiting_user_approval`
- `thread_created`
- `in_progress`
- `awaiting_review_package`
- `awaiting_clean_review`

Use these review and merge-decision state values:

- `ready_for_clean_review`: complete review package and validation passed, or validation is not applicable and the reason is reviewable.
- `review_passed`: clean review passed from the review package and validation evidence.
- `needs_remediation`: validation failed or clean review found a blocker.
- `blocked`: source context, thread delivery, review package evidence, or required tooling is still incomplete after the allowed recovery step.

Only `review_passed`, `needs_remediation`, and `blocked` are terminal states. Only `review_passed` tasks are merge-ready.
Use `not_run` for `Clean review` when a clean review could not run, and state the blocker.
Use `pass`, `pass_with_notes`, `blocker`, or `not_run` for `Clean review`; do not report numeric review scores unless the user explicitly asks for a separate scoring rubric.
Do not report the whole orchestration as complete unless every task is `review_passed`; group partial results by `review_passed`, `needs_remediation`, and `blocked`.

## Failure Handling

- If thread tools are missing, stop and report which capabilities are unavailable.
- If source context is incomplete, fetch it or ask the user for it before creating child threads.
- If some threads were created or messaged before a later create/send failure, list the created threads, reconcile their latest state with `read_thread`, and stop before creating duplicates.
- If a child thread worked from a title or summary only, treat the output as untrusted and ask it to re-check against the complete source context.
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
- Do not assume tmux panes, Codex CLI workers, or external worktree managers are equivalent to Codex App background threads.
- Do not assume background thread paths are portable across threads.
- Do not assume clean reviewers can read child worktree files.
- Do not assume issue titles are acceptance criteria.
- Do not assume archive immediately removes a thread from the visible sidebar.
