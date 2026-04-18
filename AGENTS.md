# Project Context for AI Assistants

AI coding configuration marketplace providing plugin-based setup for Claude Code and
Cursor.

<!-- BEGIN QIE DOCTRINE -->
<!-- Managed by `bin/qie doctrine apply`. Content outside the markers is preserved. -->

**Canonical rules live in [`AGENTS.md`](./AGENTS.md).** Read it first. This file adds Claude-specific notes only.

## Prime directives (the four that matter most)

1. **Read `AGENTS.md`** — every session, before any file edit.
2. **Read `CHANGELOG.md` and `TODO.md`** — before starting any task, if they exist. Never duplicate completed work.
3. **Isolate with `bin/qie worktree auto <slug>`** before modifying code when other Claude sessions may be active in this repo. The command is idempotent; it no-ops inside an existing worktree.
4. **Never log personal info, never commit secrets.** See AGENTS.md "Secrets" for the full rule.

## Session hygiene

- Use `bin/qie checkpoint "<note>"` at meaningful milestones so other sessions see what you did.
- Use `bin/qie brief <topic>` to pull RAG context from the QIE corpus before large decisions.
- End with `bin/qie worktree finish` (push + PR) when the feature is shippable, or `drop` if abandoning.

## Bash etiquette (learned from production races)

- **Always** run `git diff --cached --stat` **in the same bash block** as `git commit`, gated on an expected file count. Other concurrent sessions can rewrite the shared index between `git add` and `git commit`.
- Stage files explicitly (`git add path/to/file`), never `git add -A` or `git add .` unless you've just reviewed every listed change.

## Slash commands

Project-specific slash commands (if any) live under `.claude/commands/`. The QIE hub's master agent roster (`/bmad-agent-*`, `/quinn`) is available from any repo that carries the `_qie` symlink.

<!-- END QIE DOCTRINE -->

## Always Apply Rules

Core project rules that apply to all tasks:

@rules/git-interaction.mdc @rules/prompt-engineering.mdc

## Tech Stack

- **Claude Code** - Plugin marketplace (`.claude-plugin/marketplace.json`)
- **Cursor** - Rules and configurations (`.cursor/rules/`)
- **Bash** - Bootstrap and installation scripts
- **Markdown** - All rules, commands, and agents

## Project Structure

**Plugin-first architecture** - Everything distributable lives in `plugins/`:

- `.claude-plugin/marketplace.json` - Plugin marketplace manifest
- `plugins/core/` - Commands, agents, skills, and context (canonical source)
- `plugins/personalities/` - Personality variants
- `.cursor/rules/` - Cursor rules (canonical location)
- `rules/` - Symlink to `.cursor/rules/` for visibility (THIS REPO ONLY)
- `.claude/` - Symlinks to plugin content for local development
- `scripts/` - Installation and bootstrap scripts

## Commands

**Setup and Installation:**

- `/plugin marketplace add https://github.com/TechNickAI/ai-coding-config` - Add this
  marketplace
- `/plugin install <name>` - Install specific plugin
- `/ai-coding-config` - Interactive setup for projects
- `curl -fsSL https://raw.githubusercontent.com/TechNickAI/ai-coding-config/main/scripts/bootstrap.sh | bash` -
  Bootstrap for Cursor

## Code Conventions

**DO:**

- Create commits only when user explicitly requests
- Check for `alwaysApply: true` in rule frontmatter - these apply to ALL tasks
- Use `/load-rules` to get task-specific context
- Follow heart-centered AI philosophy (unconditional acceptance, presence before
  solutions)

**DON'T:**

- Use `--no-verify` flag (bypasses quality checks) unless explicitly requested for
  emergencies
- Commit changes without explicit user permission
- Push to main or merge into main without confirmation
- Stage files you didn't modify in current session

## Git Workflow

**Commit format:** `{emoji} {imperative verb} {concise description}`

Example: `✨ Add plugin marketplace support`

**Critical constraints:**

- Never use `--no-verify` - fix underlying issues instead (linting, tests, formatting)
- Only stage files modified in current session
- Use `git add -p` for partial staging when needed
- Push/merge to main requires explicit confirmation
- Read `git-commit-message.mdc` before generating commit messages

**Philosophy:** AI makes code changes but leaves version control to user. Commits are
permanent records requiring explicit permission.

## Important Notes

- Rules with `alwaysApply: true` are CRITICAL - currently: `git-interaction.mdc`,
  `heart-centered-ai-philosophy.mdc`
- **Plugin-first**: All content lives in `plugins/`, other locations symlink there
- `.claude/commands/` → `plugins/core/commands/` (symlink)
- `.claude/agents/` → `plugins/core/agents/` (symlink)
- `.claude/skills/` → `plugins/core/skills/` (symlink)
- `rules/` → `.cursor/rules/` (symlink for visibility, THIS REPO ONLY)
- `.cursor/rules/` contains the canonical Cursor rules
- `.cursor/rules/personalities/` → copied from `plugins/personalities/` (not symlinked -
  needs editing)
- **Note**: Personality files are copied, not symlinked, because `/personality-change`
  edits frontmatter
- **Architecture**: In THIS repo, `.cursor/rules/` is canonical and `rules/` symlinks to
  it. In user projects, only `.cursor/rules/` exists (no root symlink)
- Context in `plugins/core/context.md` describes identity and philosophy
- Bootstrap script clones repo to `~/.ai_coding_config`
