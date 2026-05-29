# Team Collaboration with Claude & GitHub
**Standard Operating Procedure — Alpha Four Engineering**
Version 1.0 · May 2026

---

## Table of Contents

1. [Purpose & Scope](#1-purpose--scope)
2. [Guiding Principles](#2-guiding-principles)
3. [Branch Strategy](#3-branch-strategy)
4. [Working with Claude — Session Workflow](#4-working-with-claude--session-workflow)
5. [Context Template for Claude Sessions](#5-context-template-for-claude-sessions)
6. [Commit Message Convention](#6-commit-message-convention)
7. [Pull Request Workflow](#7-pull-request-workflow)
8. [Preventing Merge Conflicts](#8-preventing-merge-conflicts)
9. [Code Consistency Standards](#9-code-consistency-standards)
10. [Security & Secrets](#10-security--secrets)
11. [Quick Reference — Do's and Don'ts](#11-quick-reference--dos-and-donts)

---

## 1. Purpose & Scope

This SOP defines how the Alpha Four team uses Claude for AI-assisted development alongside GitHub for version control. It establishes standards to prevent merge conflicts, ensure code consistency, and integrate AI-generated code safely into shared repositories.

> **Who this covers:** All engineers working on any Alpha Four repository who use Claude (via Claude.ai or the API) to generate, edit, or review code.

---

## 2. Guiding Principles

- **Claude is a collaborator, not an authority.** Always review AI-generated code before committing.
- **Small, focused changes.** Each Claude session should target one concern — a function, a module, a bug fix.
- **Context is king.** Claude has no memory between sessions; always re-supply relevant context.
- **Transparency.** Add a note in PR descriptions when significant portions were AI-assisted.
- **Humans own the repo.** Claude never has direct push access; all changes flow through PR review.

---

## 3. Branch Strategy

Use the following branching conventions for all repositories:

| Branch | Purpose | Who creates it |
|---|---|---|
| `main` | Production-ready code only | CI/CD pipeline |
| `develop` | Integration branch for completed features | Repo maintainer |
| `feature/<ticket-id>-<slug>` | New feature work | Any engineer |
| `fix/<ticket-id>-<slug>` | Bug fixes | Any engineer |
| `claude/<ticket-id>-<desc>` | Exploratory AI-generated code | Any engineer |

✅ **Good branch name:** `feature/REC-42-watch-next-cosine` or `claude/REC-42-embedding-schema-draft`

❌ **Avoid:** `claude-stuff` or `natalie-test-branch` or `wip`

---

## 4. Working with Claude — Session Workflow

### 4.1 Before You Start a Session

1. Pull the latest changes from your base branch before opening a Claude session.
2. Create or switch to your working branch: `git checkout -b feature/<ticket-id>-<slug>`
3. Paste the relevant context into Claude at the start of the session (see [Section 5](#5-context-template-for-claude-sessions)).
4. State your goal clearly in the first message — avoid vague openers like "help me with my project".

### 4.2 During the Session

- Work file-by-file. Do not ask Claude to rewrite multiple unrelated files in one prompt.
- After each significant generation, copy the output into your editor and run it locally before continuing.
- If Claude produces a large diff, break it into smaller commits by concern (schema change, service layer, tests).
- Flag uncertainty: if Claude hedges or says "you may need to adjust," treat that as a required manual review step.

### 4.3 After the Session

1. Run your full test suite before staging any files.
2. Do a `git diff` review of everything Claude touched — line by line.
3. Remove any debug code, placeholder comments, or hallucinated imports before committing.
4. Write a clear commit message (see [Section 6](#6-commit-message-convention)).

---

## 5. Context Template for Claude Sessions

Paste this block at the start of every new Claude session. Fill in the blanks; omit irrelevant sections.

```
## Session Context
Project: Alpha Four — RAG Recommendation Engine
Stack: [e.g. TypeScript · Supabase pgvector · Next.js]
Branch: feature/<ticket-id>-<slug>
Ticket: [link or ID]

## Relevant Files (paste key snippets or describe)
- Schema: documents table (content_id, content_type, content, embedding, metadata JSONB, created_at)
- [Paste other relevant schema / types / interfaces]

## Goal for this session
[One specific task — e.g. 'Write the cosine similarity query for Watch Next']

## Constraints / Conventions
[e.g. 'Use snake_case for DB columns', 'No default exports', 'Zod for validation']
```

---

## 6. Commit Message Convention

All commits — AI-assisted or not — follow [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>(<scope>): <short summary>  [max 72 chars]

[optional body — what and why, not how]

[optional footer: Refs #42  |  ai-assisted: yes]
```

| Type | When to use | Example |
|---|---|---|
| `feat` | New feature or capability | `feat(recommendations): add cosine similarity query` |
| `fix` | Bug fix | `fix(embeddings): handle null content_type in upsert` |
| `refactor` | Code change with no behaviour change | `refactor(vector-store): extract similarity fn` |
| `test` | Adding or updating tests | `test(watch-next): add pgvector distance edge cases` |
| `chore` | Tooling, deps, config | `chore: upgrade supabase-js to 2.43` |
| `docs` | Documentation only | `docs(sop): add Claude session context template` |

> 💡 **AI-assisted commits:** Add `ai-assisted: yes` to the commit footer so reviewers know to apply extra scrutiny to the diff.

---

## 7. Pull Request Workflow

### 7.1 PR Checklist — Author

- [ ] Branch is up to date with `develop` (rebase or merge before opening PR).
- [ ] All tests pass locally.
- [ ] No `console.log`s, TODO comments, or dead code left from Claude output.
- [ ] PR description explains what changed and why, not just what Claude said.
- [ ] If AI-assisted: note which files were substantially generated by Claude.
- [ ] Self-review the diff in GitHub before requesting reviewers.

### 7.2 PR Checklist — Reviewer

- [ ] Verify the logic, not just the style — AI-generated code can be syntactically correct but semantically wrong.
- [ ] Check that type signatures and interfaces are consistent with the shared codebase.
- [ ] Confirm any SQL or pgvector queries run the `EXPLAIN ANALYZE` path before merging.
- [ ] Leave inline comments for any concerns; do not merge with unresolved threads.

### 7.3 Merge Rules

| Item | Details |
|---|---|
| Minimum reviewers | 1 for feature branches, 2 for changes to shared types or DB schema |
| Merge strategy | Squash merge into `develop`; rebase merge into `main` |
| Branch deletion | Delete branch after merge |
| CI required | All status checks must pass before merge is enabled |

---

## 8. Preventing Merge Conflicts

### 8.1 High-Risk Files — Extra Caution Required

> ⚠️ **Warning:** These files are conflict hotspots. Coordinate via your team channel before asking Claude to modify them.

- Database migration files (`supabase/migrations/*`)
- Shared type definitions and interfaces (`types/*.ts`)
- Environment config and constants files
- Package manifests (`package.json`, `package-lock.json`)
- Shared utility modules used by multiple features

### 8.2 Coordination Rules

1. Announce in the team channel before starting a session that touches high-risk files.
2. Never run two concurrent Claude sessions targeting the same file. Finish, commit, push — then let a teammate pull before they start.
3. If a conflict occurs during rebase, resolve it manually. Do not ask Claude to auto-resolve merge conflicts without understanding the diff first.
4. Keep migration files immutable after they land on `develop`. Create a new migration instead of editing an existing one.

### 8.3 Daily Sync Habit

Start every working day with:

```bash
git fetch origin && git rebase origin/develop
```

This keeps your branch current and surfaces conflicts early when they are easiest to fix.

---

## 9. Code Consistency Standards

### 9.1 Always Supply to Claude

- Existing type definitions for any entity Claude will touch.
- Naming conventions (e.g. `snake_case` for DB, `camelCase` for TS, `kebab-case` for files).
- The project's validation library (e.g. Zod schemas) so Claude generates compatible code.
- Any shared utility functions Claude might otherwise re-implement.

### 9.2 Review Checklist for AI-Generated Code

| Area | What to check |
|---|---|
| Naming | Matches project conventions (checked against existing files) |
| Imports | No unused imports; no imports of non-existent modules |
| Error handling | Follows the project's error handling pattern (not ad-hoc try/catch) |
| Types | No implicit `any`; types match shared interfaces |
| Logging | Uses project logger, not `console.log` |
| Env vars | Accessed via the project's config module, not `process.env` directly |
| Tests | At minimum a happy path and one error case |

### 9.3 Linting & Formatting

Claude output should be run through the formatter before committing. Set up a pre-commit hook if not already in place:

```bash
# .husky/pre-commit
npx lint-staged  # runs eslint + prettier on staged files
```

---

## 10. Security & Secrets

> 🔒 **Critical:** Never paste real credentials, connection strings, or API keys into a Claude session. Use placeholder values and substitute locally.

- Supabase URLs and keys: use `SUPABASE_URL` and `SUPABASE_ANON_KEY` placeholders in prompts.
- Do not share `.env` files in Claude — describe the shape of the config instead.
- Review Claude output for any hardcoded strings that look like tokens or passwords before committing.
- If you accidentally paste a secret into Claude.ai, rotate the credential immediately.

---

## 11. Quick Reference — Do's and Don'ts

| ✅ Do | ❌ Don't |
|---|---|
| Pull & rebase before every Claude session | Start a session on a stale branch |
| Use one branch per ticket / task | Accumulate multiple unrelated changes in one branch |
| Review every line of AI output before committing | Bulk-commit Claude output without a diff review |
| Supply context: schema, types, conventions | Expect Claude to remember previous sessions |
| Note "ai-assisted" in PR description and commit footer | Hide AI involvement from reviewers |
| Coordinate on shared files in team channel first | Edit migration files or shared types without announcing |
| Run tests and linter after every Claude session | Commit code that hasn't been executed locally |
| Use placeholder values for secrets in prompts | Paste real credentials into Claude |

---

*Alpha Four · Engineering SOP · v1.0 · Review quarterly or after major tooling changes*
