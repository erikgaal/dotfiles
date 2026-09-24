# Personal defaults

Apply these preferences across projects unless I explicitly instruct otherwise. Follow project-specific conventions for implementation and verification.

## Pull requests

- Create PRs with `gh pr create --draft` unless I explicitly request a ready/non-draft PR.
- For UI changes, include screenshots of the changed UI in the PR description. Capture them with a browser automation tool and upload them with `gh pr edit <pr> --attach <file>`. A screenshot in a PR comment alone does not satisfy this requirement. Do not use the `gh image` extension.
- If screenshot capture or upload is blocked, report the blocker and what remains to be done; do not claim the PR is complete.

## Git execution

- When running `git commit` or `git push`, use `dangerouslyDisableSandbox: true` if the execution tool supports it and permissions allow it. In that case, do not attempt a sandboxed run first. Otherwise, use the environment's supported permission flow.
- This execution preference does not itself authorize committing or pushing.

## Code comments

- Default to no new comments. Prefer clear names and structure; use a short, single-line comment only when needed to explain non-obvious intent or constraints.
- Do not add multi-paragraph docstrings or multi-line comment blocks unless required by project conventions or tooling, or explicitly requested. Preserve required documentation and license notices.

## Model usage on coding tasks

- Use the main-loop model to understand the problem, explore the codebase, design the approach, make architectural decisions, and review results.
- Once the approach is settled, delegate well-specified implementation to a cheaper model when the handoff saves meaningful effort. Use the Agent tool with `model: "sonnet"` for planned code changes, refactors, and tests against a defined spec. Use `model: "haiku"` only for trivial mechanical changes such as renames, file moves, or formatting.
- Make each handoff self-contained; do not assume the subagent inherits conversation context. Include the objective, files or scope, decided approach, relevant conventions, acceptance criteria, and verification commands.
- Review the resulting diff and verification results in the main loop before considering the task complete. Resolve any issues found.
- Keep small tasks and work likely to surface design decisions in the main loop. If the Agent tool or requested models are unavailable, complete the work in the main loop.
