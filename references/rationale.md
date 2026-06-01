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
