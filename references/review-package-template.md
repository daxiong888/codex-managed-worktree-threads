# Review Package Template

Ask each child thread to return this package before clean review or merge sequencing.
Clean review should use this package as the task source and must not require direct filesystem access to the child thread's managed worktree.

```text
Task:
- Status: ready_for_clean_review | needs_remediation | blocked
- Status rules:
  - ready_for_clean_review: validation passed, or validation is not applicable and the reason is reviewable.
  - ready_for_clean_review also applies when a focused remediation fixed the blocker and the relevant validation now passes; include the remediation diff and validation evidence.
  - needs_remediation: validation failed, or the blocker remains unresolved after a focused remediation attempt.
  - blocked: source, tool, permission, environment, thread delivery, or review evidence is incomplete.
  - Child threads must not output review_passed; only the main thread may assign it after clean review.
- ID:
- Type: write_implementation | read_only_review_discovered | planning_only_discovered | hybrid_discovered
- Child Goal Mode used: yes | no
- Source link or path, or redacted source identifier if private or sensitive:
- Redacted self-contained source package:
- If source is too long or contains sensitive content, exact excerpt or redacted package used:
- Acceptance criteria covered by excerpt:
- Omitted source context:
- Why the excerpt is sufficient for clean review:
- Redactions applied, including private URLs replaced with stable non-secret identifiers:
- Acceptance criteria:
- Acceptance criteria checked:
- Non-goals:
- Base branch:
- Base commit:

Execution profile:
- Model profile requested:
- Actual model/profile used, if visible:
- Reasoning effort requested:
- Actual reasoning effort used, if visible:
- Cost/latency bias:
- Tool-enforced selectors, if known:
- Routing reason:

Thread:
- Thread identifier:
- Thread title:
- Worktree type: Codex-managed

Changes:
- Changed files:
- Diff summary:
- Diff base:
- Diff completeness: complete | redacted_complete | redacted_partial
- Completeness assertion: all behavior-changing hunks are included or summarized; omitted hunks are non-behavioral or sensitive and listed below.
- Redacted patch or detail with enough surrounding context to review each changed behavior:
- Omitted diff hunks and why omission is safe:

Validation:
- Applicability: applicable | not_applicable
- Skipped category: not_applicable | missing_dependency | permission | tool_unavailable | environment_failure | none
- Skipped/not-applicable reason:
- Commands run:
- Results:
- Redacted relevant output transcript:
- Checks not run and reason:

Risk:
- Remaining risks:
- Blockers:
- Suggested merge order:
```
