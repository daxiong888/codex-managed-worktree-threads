# Rationale

Read this only when the boundary is disputed or when deciding whether another parallel-worker workflow can substitute for Codex App background threads with Codex-managed worktrees.

## Boundary

This skill coordinates Codex App background threads. The Codex App creates and manages the worktrees for those threads.

Do not replace that with:

- manual `git worktree add`
- tmux panes or terminal multiplexing
- Codex CLI worker orchestration
- subagents doing implementation work
- guessed filesystem paths for child thread worktrees

Those workflows can be useful in their own tools, but they do not provide the same Codex App thread, diff, review, and lifecycle surface.

## Why read-only work should not create worktrees

Codex-managed worktrees are valuable when a child thread may edit files and produce a diff. Read-only scenarios such as multi-perspective review, architecture critique, PRD review, scoring, issue triage, or planning-only work do not need an isolated filesystem. Creating worktrees for those tasks adds lifecycle and reconciliation overhead without producing a mergeable artifact.

When a task has both read-only investigation and possible implementation, split it first. Run the read-only part as review/planning, then create a Codex-managed worktree thread only for the concrete write task that remains.

## Why Goal Mode belongs only in child prompts

Groundwork `to-issues` output makes each issue a natural candidate goal, but the coordinator thread is not the implementation worker. The coordinator should route, monitor, review, and sequence results. Goal Mode directives therefore belong inside the child-thread prompt for the specific issue, not in the current/main conversation.

This reduces the chance that the user says "use goal mode" and the coordinator starts executing the goal itself instead of launching the intended child thread.

## Why execution profiles are per task

Different issues benefit from different tradeoffs. Small localized edits are usually better routed to faster/lower-cost profiles, while cross-cutting changes, migrations, security-sensitive edits, and clean reviews benefit from stronger reasoning. The skill records model profile, reasoning effort, cost/latency bias, and routing reason per task so the orchestration can optimize cost, speed, and quality instead of using one fixed profile for every child thread.

If the thread tools expose model or reasoning selectors, use them. If they do not, keep the execution profile as an explicit prompt-level preference and report that it was not tool-enforced.

## Reference Signals

These are contrast and community signals, not authoritative runtime contracts. The hard boundary comes from the requested Codex App managed-thread workflow, current thread-tool availability, and the requirement that the Codex App manages the background thread worktrees.

Do not refresh these external references during normal execution. Re-check them only when the boundary is disputed, the user asks for current source verification, or a changed Codex App/thread-tool behavior would alter this skill's workflow.

- `moonshotai/kimi-cli@codex-worker`: closest known skill-shaped comparison, but it uses Codex CLI, tmux, and manual `git worktree add`; treat it as a contrast case, not an implementation template.
  https://skills.sh/moonshotai/kimi-cli/codex-worker
- `dmux-workflows`: useful for task boundaries and pre-merge summary ideas, but it is still an external tmux/worktree workflow.
  https://skills.sh/affaan-m/everything-claude-code/dmux-workflows
- `workmux`: mature git worktree plus tmux lifecycle workflow; borrow lifecycle caution, not its worktree creation model.
  https://github.com/raine/workmux
- OpenAI Codex community discussion 16440: community evidence that manual worktrees can desynchronize Codex UI, diff, and review state; this supports the hard ban on manual `git worktree add`.
  https://github.com/openai/codex/discussions/16440

## Review Package Rule

Main-thread and clean-review agents may not be able to read a child thread's Codex-managed worktree path. The child thread must return a redacted, self-contained review package instead of asking reviewers to inspect the managed path directly.

Self-contained means enough information to review the change without filesystem access. It does not mean copying secrets, private payloads, credentials, personal data, private URLs, unnecessary full logs, or customer-sensitive content.
