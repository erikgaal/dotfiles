When creating pull requests, always use the `--draft` flag unless explicitly asked to create a ready/non-draft PR.

When a pull request contains UI changes, it MUST include screenshots of the change in the PR description (e.g. take them with a browser automation tool, then attach/embed them). Attach images with the `gh` CLI's `--attach <file>` flag (e.g. on `gh pr edit` / `gh pr comment`), not the `gh image` extension.

Always run `git commit` and `git push` with `dangerouslyDisableSandbox: true`. Do not attempt a sandboxed run first.

# Model usage on coding tasks

Use the expensive main-loop model (Fable) for the deep thinking: understanding the problem, exploring the codebase, designing the approach, making architectural decisions, and reviewing results. Once the approach is settled and the work is well-specified, delegate the implementation to a subagent on a cheaper model instead of doing the mechanical edits in the main loop:

- Use the Agent tool with `model: "sonnet"` for implementation work (writing the planned code, applying a refactor across files, writing tests against a defined spec). Use `model: "haiku"` only for trivial mechanical changes (renames, moving files, formatting).
- Give the subagent a self-contained prompt: it has none of the conversation context, so include the exact files to touch, the decided approach, relevant conventions, and how to verify (e.g. which test command to run).
- After the subagent finishes, review its diff in the main loop before considering the task done.
- Don't delegate when the task is small enough that writing the handoff prompt costs more than just doing it, or when implementation is likely to surface design decisions (tight feedback loop needed) — do those in the main loop.