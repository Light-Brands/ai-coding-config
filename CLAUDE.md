# CLAUDE.md — ai-coding-config

Claude-specific notes for THIS repo. **Read [`AGENTS.md`](./AGENTS.md) first** — it has the
full picture. This file only adds things that matter when the assistant is Claude Code.

## Session start checklist

1. Read `AGENTS.md` (project rules, stack, gotchas).
2. Skim recent commits: `git log --oneline -10`. Repo merges from upstream
   `TechNickAI/ai-coding-config` periodically — fork-only edits should be flagged.
3. There is **no `CHANGELOG.md` or `TODO.md`**. The closest things are `implementation-plan.md`
   (historical scratch, not authoritative) and `docs/future-ideas.md`.
4. If editing agents/skills/commands, read `.claude/CLAUDE.md` for YAML frontmatter and
   color conventions. If editing Cursor rules, read `.cursor/AGENTS.md`.

## Worktree rule

Use `bin/qie worktree auto <slug>` (from QIE root, NOT from inside this repo) before any
non-trivial change when other Claude sessions might be open against this repo. Skip it for:

- Read-only exploration / questions / reviews.
- A single doc-only edit you can ship in <30 seconds.
- This doctrine work itself (single-session, single-edit-cycle).

If you do create a worktree, all `cd`, edits, and commits happen inside it.

## Commit conventions (Claude-specific reminders)

- Commit format: `{emoji} {imperative verb} {description}`. See `rules/git-commit-message.mdc`.
- Stage explicit paths (`git add path/to/file`). Never `git add -A` or `git add .`.
- Run `git diff --cached --stat` in the **same bash block** as `git commit`, gated on
  expected file count — concurrent sessions can rewrite the index between calls.
- Never use `--no-verify`. Pre-commit catches real things (smartquotes, frontmatter, large
  files at >500KB, symlinks, GitHub Actions lint). Fix the underlying issue.
- Don't push or merge to `main` without explicit user permission.

## No build, no test, no dev server

This repo has nothing to run. Validation is the only "test":

- `pre-commit run --all-files` — runs the full hook chain locally
- `python scripts/validate-frontmatter.py <files>` — frontmatter-only check
- `bash scripts/validate-marketplace.sh` — marketplace structure
- `actionlint` — runs via pre-commit, lints `.github/workflows/*.yml`

If `pre-commit` is not installed: `pip install pre-commit && pre-commit install`.

## Harness skill false positives to ignore

- **`vercel:bootstrap` / `vercel:knowledge-update`** auto-fire on words like `bootstrap` and
  on reading `README.md`. This repo's `bootstrap.sh` is a shell installer for the AI coding
  config, not Vercel/Next.js bootstrapping. Ignore those skills here.
- **`vercel-plugin:react-best-practices`** triggers after editing TSX. There is no TSX in
  this repo.
- **`vercel:nextjs`, `vercel:vercel-cli`, etc.** — never relevant here. Pure markdown repo.

## Things that look broken but aren't

- `CLAUDE.md` was a symlink to `AGENTS.md` until this commit. Now they're separate files.
  `.cursor/CLAUDE.md → .cursor/AGENTS.md` is still a symlink — leave it.
- `rules/`, `commands/`, `skills/` at repo root are visibility-only symlinks. They don't
  ship to user projects.
- `plugins/core/agents/CLAUDE.md` and `plugins/core/skills/CLAUDE.md` are dev notes for
  contributors; Claude does not auto-load them.
- README counts (18/24/6/33) are stale. Real counts: 21 commands, 28 agents, 13 skills, 33
  rules. Fix opportunistically.
