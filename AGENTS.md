# AGENTS.md — ai-coding-config

Canonical operational rules for AI assistants working on THIS repo.
`CLAUDE.md` is no longer a symlink to this file — it adds Claude-only notes on top.

## What this is

This repo IS the **ai-coding-config marketplace**: a Claude Code plugin + Cursor rules pack
distributed via `https://github.com/TechNickAI/ai-coding-config`. Light-Brands maintains a
fork at `git@github.com:Light-Brands/ai-coding-config.git`. We pull upstream (`TechNickAI`)
periodically — see `f4dccbb Merge branch 'TechNickAI:main' into main`. There is **no
runtime, no server, no build** — every artifact is markdown, YAML, or shell. End users
install it via `/plugin marketplace add ...` (Claude Code) or `scripts/bootstrap.sh` (Cursor
et al.). Current version: `9.16.0` (see `.claude-plugin/marketplace.json`).

## Stack

- **Markdown + YAML frontmatter** — every command, agent, skill, rule
- **Shell** — `scripts/bootstrap.sh` (clones repo to `~/.ai_coding_config`), `scripts/validate-marketplace.sh`, `plugins/core/hooks/todo-persist.sh`
- **Python** — `scripts/validate-frontmatter.py` (only Python file, runs in pre-commit)
- **pre-commit** — see `.pre-commit-config.yaml` (frontmatter validation, smartquote fixing, actionlint, large-file block at 500KB, symlink validation)
- **Prettier** — `.prettierrc` enforces prose wrap; agent/skill descriptions use `# prettier-ignore` to keep on one line (see `.claude/CLAUDE.md`)
- **GitHub Actions** — `.github/workflows/{claude.yml, claude-code-review.yml, validate-marketplace.yml}`. The `@claude` mention triggers `anthropics/claude-code-action@v1` with `--permission-mode bypassPermissions`

## Where things live (purpose-keyed map)

```
.claude-plugin/marketplace.json     Marketplace manifest. version=9.16.0. ONE plugin: "ai-coding-config" → ./plugins/core
.claude-plugin/CLAUDE.md            Marketplace dev notes — describes the THREE-PLACE version sync rule
plugins/core/                       The actual plugin (canonical source for everything distributable)
  .claude-plugin/plugin.json        Plugin manifest. version MUST match marketplace.json (sync rule)
  commands/    (21 .md)             Slash commands (/autotask, /troubleshoot, /multi-review, /session, etc.)
  agents/      (28 .md)             Specialized subagents Claude invokes contextually (security-reviewer, debugger, etc.)
  skills/      (13 dirs)            Multi-step skills with SKILL.md (research, brainstorming, kungfu, etc.)
  hooks/       (hooks.json + .sh)   PostToolUse hook on TodoWrite that persists todos to disk
  context.md                        Heart-centered AI identity description
  code-review-standards.md          Shared review criteria
plugins/personalities/  (7 dirs)    Personality variants (samantha, sherlock, bob-ross, etc.) — COPIED into .cursor/rules/personalities/, NOT symlinked
.cursor/rules/    (33 .mdc)         CANONICAL Cursor rules. Use .mdc extension w/ YAML frontmatter (description, alwaysApply, globs)
  ai/, django/, frontend/, observability/, python/, personalities/   subdirectories by domain
.cursor/AGENTS.md                   Cursor-specific dev notes
.claude/                            Symlinks pointing INTO plugins/core/ for local dev (commands → plugins/core/commands, etc.)
.claude/AGENTS.md                   Plugin authoring conventions (YAML, colors, triggers) — READ THIS before editing agents/skills
.claude/CLAUDE.md                   Same content as .claude/AGENTS.md (separate file, not symlink)
rules → .cursor/rules               Symlink for visibility — THIS REPO ONLY, not in installed projects
commands → .claude/commands         Symlink for visibility
skills → .claude/skills             Symlink for visibility
scripts/                            bootstrap.sh, validate-marketplace.sh, validate-frontmatter.py
docs/                               Architecture/ecosystem/contributing guides + docs/plans/
context/                            design-principles.md, optimal-development-workflow.md
knowledge/competitors/              Competitor research (output of /product-intel)
templates/python, templates/typescript    Project scaffolds (referenced by /repo-tooling)
implementation-plan.md              ~22KB working plan — historical, not authoritative
```

## Current focus

Recent commits (3 total — repo is shallow because we forked):

- `f4dccbb` — last upstream merge from TechNickAI/main
- `b50ab29`-era work (in upstream): see README for v9.x feature set

Active dev pattern: track upstream, occasionally fork-specific tweaks. **No current open
work on `main`.** A sibling branch `claude/doctrine-bootstrap` adds a generic QIE doctrine
block to `AGENTS.md` only — leave it alone unless explicitly asked.

## Conventions (real patterns, observed)

- **Plugin-first.** Anything distributable lives in `plugins/`. Other locations symlink
  there. Editing the symlink target updates everywhere.
- **Personalities are COPIED, not symlinked**, into `.cursor/rules/personalities/` because
  `/personality-change` rewrites frontmatter (`alwaysApply: true/false`).
- **YAML frontmatter is validated** by `scripts/validate-frontmatter.py` on every commit.
  It checks `(.claude|plugins/core)/{commands,agents}/*.md`,
  `(.claude|plugins/core)/skills/*/SKILL.md`, and `.cursor/rules/**/*.mdc`. Top-level
  `CLAUDE.md` files are excluded.
- **Long descriptions use `# prettier-ignore`** above the `description:` field so prettier
  doesn't wrap them. Wrapping breaks Claude's semantic matching for "Use when..." triggers.
- **Agent colors are categorical**: red=security, orange=bugs, yellow=perf, green=tests,
  cyan=observability, blue=style, purple=UX, magenta=meta. See `.claude/CLAUDE.md`.
- **Skill triggers are an array of natural-language phrases** in `SKILL.md` frontmatter,
  not regex. Match what users actually say.
- **Three-place version sync** when bumping: `.claude-plugin/marketplace.json` (×2 fields)
  + `plugins/core/.claude-plugin/plugin.json`. Mismatch = updates don't propagate. See
  `.claude-plugin/CLAUDE.md`.
- **Commit format**: `{emoji} {imperative verb} {description}` (e.g. `✨ Add plugin
  marketplace support`). See `rules/git-commit-message.mdc`.
- **Always-apply rules** (loaded every session): `git-interaction.mdc`,
  `prompt-engineering.mdc`, `heart-centered-ai-philosophy.mdc`. Check the `alwaysApply:
  true` frontmatter field — that's the source of truth, not docs.

## Gotchas (only learnable from reading the code)

- `CLAUDE.md` at repo root **was a symlink** to `AGENTS.md`. We broke that symlink so this
  doctrine work could exist. `.cursor/CLAUDE.md` is still a symlink to `.cursor/AGENTS.md`
  — don't break that one without a reason.
- `rules/`, `commands/`, `skills/` at repo root are symlinks that exist **only in this
  repo** for human navigation. Bootstrap.sh / `/plugin install` does NOT create them in
  user projects. Don't reference them in docs that ship to users.
- `plugins/core/agents/CLAUDE.md` and `plugins/core/skills/CLAUDE.md` are dev-notes inside
  the plugin tree — they do NOT load as project context (Claude only reads CLAUDE.md from
  cwd ancestors). They exist for contributors browsing those folders.
- The pre-commit `validate-frontmatter` step excludes any path matching `CLAUDE\.md$`. If
  you add YAML frontmatter to a CLAUDE.md, it won't be checked. Fine, but be aware.
- `bootstrap.sh` requires being inside a git repo and only supports macOS/Linux. It clones
  to `~/.ai_coding_config` and pulls on subsequent runs.
- The `@claude` GitHub Action runs in `--permission-mode bypassPermissions`. PRs from
  forks should not be able to trigger it (default action permissions), but be careful
  about lowering that bar.
- README claims 18 commands / 24 agents / 6 skills / 33 rules. Actual counts on disk:
  **21 commands, 28 agents, 13 skills, 33 rules**. README is stale — fix opportunistically.
- The `.gitignore` ignores `bin/`, `lib/`, `dist/`, `build/`, `node_modules/` — these are
  for downstream test installs, not artifacts of this repo.
- `implementation-plan.md` (21KB at root) is historical scratch, not authoritative spec.

## Don'ts (hard bans for this codebase)

- **Don't `git add -A` or `git add .`** — pre-commit and other concurrent sessions can
  produce surprises. Stage explicit paths.
- **Don't use `--no-verify`** to bypass pre-commit. Fix the underlying issue.
- **Don't push or merge to `main`** without explicit user permission. Same for force-push.
- **Don't bump the version in only one place.** All three (see Conventions) or none.
- **Don't duplicate distributable content outside `plugins/`.** Symlink, don't copy
  (personalities are the documented exception).
- **Don't add real Cursor IDE settings or runtime code** to this repo — it's pure
  configuration distribution. No package.json, no node_modules in git, no build step.
- **Don't edit upstream-tracked files with fork-only assumptions** without flagging it in
  the PR — this repo merges from `TechNickAI:main` periodically, so divergent edits cause
  conflicts.

## Secrets

- No runtime secrets exist or should exist in this repo. The only credential reference
  is `secrets.CLAUDE_CODE_OAUTH_TOKEN` in `.github/workflows/claude.yml` — that's a repo
  secret managed in GitHub UI, never committed.
- Never commit `.env*` files. The `.gitignore` does not list them — if a contributor
  introduces one, **add an explicit ignore line in the same commit.**
- Never log or commit user paths, API keys, or session data. `.claude/sessions/` is
  already gitignored.
