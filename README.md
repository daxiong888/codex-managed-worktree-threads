# Codex Managed Worktree Threads

[![skills.sh](https://skills.sh/b/daxiong888/codex-managed-worktree-threads)](https://skills.sh/daxiong888/codex-managed-worktree-threads)

Codex App-only skill for coordinating one Codex App background thread per task in Codex-managed worktrees.

This skill is intentionally explicit-only: invoke it as `$codex-managed-worktree-threads` when you want Codex to coordinate parallel implementation tasks through Codex App-managed background threads and worktrees.

## Install

Install globally for Codex:

```bash
npx skills add daxiong888/codex-managed-worktree-threads -a codex -g
```

Preview the skill before installing:

```bash
npx skills add daxiong888/codex-managed-worktree-threads --list
```

## Compatibility

Requires Codex App background-thread tools discoverable through `tool_search`.

This skill is not intended for CLI-only sessions, IDE-only sessions, manual `git worktree` workflows, tmux orchestration, or Codex CLI worker orchestration.

## Usage

Invoke the skill explicitly:

```text
$codex-managed-worktree-threads
```

Then provide the task matrix, source-of-truth locations, acceptance criteria, and validation signals for each child thread.

## Files

- `SKILL.md`: skill entry point and operating workflow.
- `agents/openai.yaml`: Codex UI metadata and explicit-invocation policy.
- `references/thread-prompt-template.md`: child-thread prompt template.
- `references/review-package-template.md`: child-thread review package contract.
- `references/rationale.md`: design rationale and boundary references.
