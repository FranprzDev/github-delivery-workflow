---
name: github-delivery-workflow
description: Apply a structured GitHub delivery workflow to any repository when implementing code or project changes, preparing commits, creating or updating pull requests, reviewing PR quality, auditing changes, rebasing, and verifying CI/CD. Use for requests such as "creá un PR", "subí los cambios", "auditemos el PR", or when a project needs a documented and orderly change history.
---

# GitHub Delivery Workflow

Maintain every project through small, traceable GitHub changes. Keep related work together, document it briefly, and leave pull requests in a verifiable ready-for-review state.

## Core rules

- Never make code or project changes directly on `main`, `master`, or another primary branch. Work on a descriptive branch such as `agent/<short-description>` or `chore/<short-description>`.
- Do not commit, push, create a PR, merge, deploy, or modify external state unless the user requested that action. A request to implement code authorizes local changes and validation, not publication by itself.
- When the user explicitly asks to create a PR, perform the complete publication flow: inspect scope, commit, validate, push, and create a normal open PR ready for review. Never create a draft PR unless the user explicitly asks for a draft.
- Do not merge a PR unless the user explicitly requests the merge.
- Keep all related changes in the same PR when practical. Each additional requested change after the PR exists becomes a new commit on the same branch and PR, unless it is unrelated or the user requests another PR.
- Use concise, descriptive Spanish or project-appropriate titles and descriptions. The PR description must state what changed, why, impact, and validation.
- Every PR must always show a clean merge status on GitHub: no conflicts, no rebase problems, green checks. Whenever the base branch moves, rebase, rerun checks, push with `--force-with-lease`, and confirm via the API that the PR is mergeable before handing off.
- Never silently stage unrelated work. Inspect `git status --short --branch` and the diff before staging.

## Code conventions from my reviews

Before writing code, apply the conventions my PR reviews keep asking for (full list in `config-ai-bots/CODE_REVIEW_LESSONS.md`):

- Finite domain values are enums or shared contract types, never inline string literals; type LangGraph nodes and edges with enums.
- No `any`; extract repeated inline types into named types.
- `async/await` over `.then()` chains; validate only what is needed; plain loops over clever reducers.
- Optional chaining only for valid no-ops; use storage/service abstractions instead of direct `fs`; no hidden config fallbacks; delete dead code.
- One concern per PR (no style migrations mixed with features); only the repo's package manager; no helper scripts for mechanical edits.
- When fixing review comments, one commit naming each resolved comment, and add the lesson to the repo's `AGENTS.md`.

## Local change workflow

1. Identify the repository, current branch, remote, and working-tree state.
2. Read the repository's local instructions and relevant project conventions.
3. If on the primary branch, create a descriptive feature, fix, or chore branch. Preserve existing user changes.
4. Implement the requested scope only.
5. Run the most relevant checks: tests, lint, type-check, build, formatting, and project-specific checks as available.
6. Review the final diff and `git diff --check`.
7. If the user has not asked for a PR, report the validated local branch and stop before commit/push when publication is not authorized.

## PR creation and updates

When asked to create a PR:

1. Confirm the intended files belong to the PR.
2. Create a terse, meaningful commit for the completed change. Use additional commits for later changes; do not rewrite history unless the user asks for squashing or cleanup.
3. Push the branch with upstream tracking.
4. Create a standard, non-draft PR against the repository's default branch unless another base is specified.
5. Use a brief body with these sections:

   - **Qué cambia:** concrete implementation changes.
   - **Por qué:** problem or goal addressed.
   - **Impacto:** user, product, deployment, or operational effect.
   - **Validación:** exact commands and their result.

6. If the PR already exists, push new commits to its branch so the same PR updates automatically.
7. Report the PR URL, branch, commits, base branch, validation, and anything still requiring the user's decision.

## Required PR review comments

After creating or materially updating a PR, add two concise comments unless the user asks not to:

1. **Evaluación crítica:** assess whether the implementation is appropriate, identify risks, missing tests, regressions, unnecessary complexity, and concrete improvements.
2. **Perspectiva de ingeniería senior:** recommend how to approach the problem, what to monitor, and what future refactoring or architectural decision would improve maintainability.

Comments must be evidence-based and specific to the diff. Do not praise generically, invent unverified facts, or treat the comments as approval. If the GitHub integration cannot post comments, include both comments in the handoff and state the limitation.

## Rebase and green checks

- Before declaring a PR ready, inspect its checks and review status.
- If the branch is behind the base branch or the user requests the rebase strategy, fetch the base branch and rebase the work branch onto it. Resolve conflicts carefully, preserve user changes, rerun all relevant checks, and force-push only when necessary and with `--force-with-lease`.
- A PR is not ready until required CI/CD checks are green, the build/test surface has been validated, and there are no known merge conflicts.
- If checks fail, diagnose the failure before claiming completion. Fix only when authorized by the request; otherwise report the blocker clearly.
- Do not call a PR mergeable based only on local compilation. Check the actual GitHub checks and merge state.

## Audit mode

When the user says "auditemos el PR", "auditar el PR", or equivalent:

1. Identify the PR, base, head, current status, mergeability, checks, and changed files.
2. Inspect the complete diff and relevant surrounding code.
3. Look for correctness issues, regressions, security/privacy risks, data loss, deployment mismatch, missing validation, and scope creep.
4. Prioritize findings by severity and cite files and line numbers when possible.
5. Separate confirmed findings from questions or suggestions.
6. Verify that required checks are green and that the branch can be merged cleanly.
7. End with a concise recommendation: ready, changes requested, or blocked, explaining why.

## Safety and handoff

- Never use destructive commands such as `git reset --hard`, broad recursive deletion, or force-push without explicit authorization and a verified target.
- Never expose tokens, credentials, or environment secrets in commits, PR descriptions, comments, or output.
- If a PR was merged or closed unexpectedly, do not recreate history or alter the base branch. Explain what happened, inspect the merge actor and timestamp, and create a new PR only when needed and authorized by the user's ongoing request.
- Final handoffs must distinguish local validation from remote GitHub state.
