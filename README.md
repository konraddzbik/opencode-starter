# OpenCode Prototyping Config — User Guide

Complete setup guide for configuring OpenCode with Anthropic models and a curated set of agents, commands, and skills for fullstack web prototyping.

---

## What You Get

This configuration package turns OpenCode into a specialized prototyping environment.

**Models**: Claude Sonnet 4 for coding and planning, Claude Haiku 4.5 for lightweight tasks (titles, summaries). Optional free models via OpenCode Zen for experimentation.

**3 subagents** you can invoke with `@mention`:

| Agent | Purpose | Tools |
|---|---|---|
| `@architect` | Plans architecture, outputs Drizzle schemas and file plans | Read-only |
| `@reviewer` | Reviews code for bugs, security, performance | Read-only |
| `@fixer` | Debugs errors, applies fixes, verifies | Full access |

**6 slash commands** for common workflows:

| Command | Example |
|---|---|
| `/proto` | `/proto nextjs my-saas` — scaffold a new project |
| `/arch` | `/arch user dashboard with analytics` — plan architecture |
| `/review` | `/review` — review uncommitted git changes |
| `/fix` | `/fix Cannot read property 'id' of undefined` — debug an error |
| `/test` | `/test gen src/lib/orders.ts` — generate tests for a file |
| `/deploy` | `/deploy vercel` — prepare app for deployment |

**10 skills** automatically loaded by agents when relevant:

`project-scaffold` · `iterative-ui` · `api-design` · `database-design` · `auth-patterns` · `deployment-ready` · `testing-strategy` · `debug-detective` · `code-review` · `cli-builder`

---

## Prerequisites

Before starting, make sure you have:

1. **OpenCode installed** — if not, run:
   ```bash
   curl -fsSL https://opencode.ai/install | bash
   ```
   Verify with `opencode --version`.

2. **An Anthropic API key** — get one at https://console.anthropic.com/settings/keys

3. **Git** — the config files should be version-controlled.

---

## Step 1: Set Up Anthropic API Key

You have two options. Choose one.

**Option A — Environment variable** (recommended for single machine):
```bash
# Add to your shell profile (~/.bashrc, ~/.zshrc, or ~/.profile)
echo 'export ANTHROPIC_API_KEY="sk-ant-your-key-here"' >> ~/.zshrc
source ~/.zshrc
```

**Option B — OpenCode interactive setup**:
```bash
opencode
# Inside the TUI, type:
#   /connect
# Select "Anthropic" → "Manually enter API Key" → paste your key
```

---

## Step 2: Create the Config Directory

OpenCode looks for global configuration in `~/.config/opencode/`. Create the full directory structure:

```bash
mkdir -p ~/.config/opencode/agents
mkdir -p ~/.config/opencode/commands
mkdir -p ~/.config/opencode/skills/project-scaffold
mkdir -p ~/.config/opencode/skills/iterative-ui
mkdir -p ~/.config/opencode/skills/api-design
mkdir -p ~/.config/opencode/skills/database-design
mkdir -p ~/.config/opencode/skills/auth-patterns
mkdir -p ~/.config/opencode/skills/deployment-ready
mkdir -p ~/.config/opencode/skills/testing-strategy
mkdir -p ~/.config/opencode/skills/debug-detective
mkdir -p ~/.config/opencode/skills/code-review
mkdir -p ~/.config/opencode/skills/cli-builder
```

---

## Step 3: Copy the Config Files

Assuming you extracted the downloaded files to `~/Downloads/opencode-config/`:

```bash
CONFIG_SRC=~/Downloads/opencode-config
CONFIG_DST=~/.config/opencode

# Main config (rename .jsonc → .json)
cp "$CONFIG_SRC/opencode.jsonc" "$CONFIG_DST/opencode.json"

# Global rules
cp "$CONFIG_SRC/AGENTS.md" "$CONFIG_DST/AGENTS.md"

# Agents
cp "$CONFIG_SRC/agents/architect.md" "$CONFIG_DST/agents/"
cp "$CONFIG_SRC/agents/reviewer.md"  "$CONFIG_DST/agents/"
cp "$CONFIG_SRC/agents/fixer.md"     "$CONFIG_DST/agents/"

# Commands
cp "$CONFIG_SRC/commands/proto.md"   "$CONFIG_DST/commands/"
cp "$CONFIG_SRC/commands/arch.md"    "$CONFIG_DST/commands/"
cp "$CONFIG_SRC/commands/review.md"  "$CONFIG_DST/commands/"
cp "$CONFIG_SRC/commands/fix.md"     "$CONFIG_DST/commands/"
cp "$CONFIG_SRC/commands/test.md"    "$CONFIG_DST/commands/"
cp "$CONFIG_SRC/commands/deploy.md"  "$CONFIG_DST/commands/"

# Skills (each is a directory with SKILL.md inside)
for skill in project-scaffold iterative-ui api-design database-design \
             auth-patterns deployment-ready testing-strategy \
             debug-detective code-review cli-builder; do
  cp "$CONFIG_SRC/skills/$skill/SKILL.md" "$CONFIG_DST/skills/$skill/"
done
```

Or, if you prefer a single command:

```bash
cp "$CONFIG_SRC/opencode.jsonc" "$CONFIG_DST/opencode.json"
cp "$CONFIG_SRC/AGENTS.md" "$CONFIG_DST/"
cp "$CONFIG_SRC"/agents/*.md "$CONFIG_DST/agents/"
cp "$CONFIG_SRC"/commands/*.md "$CONFIG_DST/commands/"
for d in "$CONFIG_SRC"/skills/*/; do
  skill=$(basename "$d")
  cp "$d/SKILL.md" "$CONFIG_DST/skills/$skill/"
done
```

---

## Step 4: Verify the Installation

Check the file structure matches what OpenCode expects:

```bash
find ~/.config/opencode -type f | sort
```

Expected output (22 files):

```
~/.config/opencode/AGENTS.md
~/.config/opencode/agents/architect.md
~/.config/opencode/agents/fixer.md
~/.config/opencode/agents/reviewer.md
~/.config/opencode/commands/arch.md
~/.config/opencode/commands/deploy.md
~/.config/opencode/commands/fix.md
~/.config/opencode/commands/proto.md
~/.config/opencode/commands/review.md
~/.config/opencode/commands/test.md
~/.config/opencode/opencode.json
~/.config/opencode/skills/api-design/SKILL.md
~/.config/opencode/skills/auth-patterns/SKILL.md
~/.config/opencode/skills/cli-builder/SKILL.md
~/.config/opencode/skills/code-review/SKILL.md
~/.config/opencode/skills/database-design/SKILL.md
~/.config/opencode/skills/debug-detective/SKILL.md
~/.config/opencode/skills/deployment-ready/SKILL.md
~/.config/opencode/skills/iterative-ui/SKILL.md
~/.config/opencode/skills/project-scaffold/SKILL.md
~/.config/opencode/skills/testing-strategy/SKILL.md
```

---

## Step 5: Launch OpenCode and Confirm

```bash
cd ~/some-project    # navigate to any git repo
opencode
```

Inside the TUI, verify:

1. **Model is correct**: The status bar should show `claude-sonnet-4`. If not, run `/models` and select `anthropic/claude-sonnet-4-20250514`.

2. **Agents are loaded**: Press `Tab` — you should cycle between **Build** and **Plan** modes. Type `@` — you should see `architect`, `reviewer`, and `fixer` in the autocomplete.

3. **Commands work**: Type `/proto` — it should appear with the description "Scaffold a new project." Type `/arch`, `/review`, `/fix`, `/test`, `/deploy` — all should be listed.

4. **Skills are discovered**: Ask the agent: "What skills are available?" — it should list all 10 skills.

---

## Step 6 (Optional): Add Free Models via OpenCode Zen

If you want to experiment with free models alongside Anthropic:

```bash
# Inside OpenCode TUI:
/connect
# Select "OpenCode Zen"
# Create account at https://opencode.ai/zen, get API key, paste it
```

Then switch models on the fly:
```bash
/models
# Select a free model like "big-pickle" or "glm-4.7" for casual tasks
# Switch back to Claude Sonnet for quality work
```

Free Zen models do not require changes to `opencode.json` — they are available after `/connect`.

---

## Quick Reference: Daily Usage

### Starting a New Project
```
/proto nextjs my-saas
```
The build agent loads the `project-scaffold` skill, generates the directory structure, installs dependencies, and verifies the dev server starts.

### Planning Before Building
```
Press Tab → switch to Plan mode
Describe what you want to build
Plan agent analyzes without making file changes
Press Tab → switch back to Build mode
"Go ahead and implement the plan"
```

### Architecture Review
```
/arch payment processing with Stripe
```
The architect subagent reads your codebase and outputs: data models (Drizzle schema), API surface (tRPC procedures), file plan, dependencies, and risks.

### Code Review Before Commit
```
/review
```
Reviews your `git diff` and categorizes issues as Critical / Important / Suggestions.

### Debugging
```
/fix Error: SQLITE_ERROR: no such table: users
```
The fixer agent reproduces the error, traces the root cause, applies a fix, and verifies.

### Adding Tests
```
/test gen src/lib/orders.ts     # generate tests for a file
/test run                        # run all tests, fix failures
/test setup                      # set up Vitest from scratch
```

### Deploying
```
/deploy vercel
```
Runs pre-deploy checks, generates platform config, verifies the build, and sets up CI.

---

## Customization

### Adding Your Own Skills

Create a directory under `~/.config/opencode/skills/` with a `SKILL.md` file:

```bash
mkdir -p ~/.config/opencode/skills/my-custom-skill
```

```markdown
---
name: my-custom-skill
description: What this skill does and when to use it (max 1024 chars)
---

## Instructions for the agent...
```

Important rules for skill names:
- Lowercase letters, numbers, and single hyphens only
- Must match the directory name exactly
- 1–64 characters, no leading/trailing hyphens
- Regex: `^[a-z0-9]+(-[a-z0-9]+)*$`

### Adding Project-Specific Config

For per-project overrides, create files in your project root:

```
my-project/
  .opencode/
    opencode.json       # Project config (overrides global)
    AGENTS.md           # Project-specific rules (merged with global)
    agents/             # Project-specific agents
    commands/           # Project-specific commands
    skills/             # Project-specific skills
```

Run `/init` inside a project to auto-generate a project-level `AGENTS.md` based on your codebase.

### Changing Models

Edit `~/.config/opencode/opencode.json` and update the `model` field:

```jsonc
{
  "model": "anthropic/claude-sonnet-4-20250514",   // main model
  "small_model": "anthropic/claude-haiku-4-5"       // titles, summaries
}
```

Or override per-agent:

```jsonc
{
  "agent": {
    "plan": {
      "model": "anthropic/claude-sonnet-4-20250514"
    },
    "build": {
      "model": "opencode/big-pickle"  // free Zen model for build
    }
  }
}
```

### Adjusting Permissions

The default config sets all permissions to `"allow"` for maximum autonomy. To require confirmation before file changes:

```jsonc
{
  "permission": {
    "bash": "ask",
    "edit": "ask",
    "write": "ask"
  }
}
```

---

## Troubleshooting

**"Model not found" or "Unauthorized"**
- Verify your API key: `echo $ANTHROPIC_API_KEY` — should start with `sk-ant-`
- Run `/connect` → Anthropic → re-enter your key

**Skills not showing up**
- Check that `SKILL.md` is uppercase: `SKILL.md`, not `skill.md`
- Verify the frontmatter has both `name:` and `description:` fields
- Confirm the `name` in frontmatter matches the directory name exactly

**Commands not appearing**
- Check the `.md` file is in `~/.config/opencode/commands/`
- Verify the frontmatter has a `description:` field
- Restart OpenCode (quit and relaunch)

**Agent not found when using @mention**
- Check the `.md` file is in `~/.config/opencode/agents/`
- Verify `mode: subagent` is set in the frontmatter
- Type `@` and check the autocomplete list

**Config not taking effect**
- Global config: `~/.config/opencode/opencode.json`
- Project config (higher priority): `./opencode.json` in project root
- Run `opencode` with `--verbose` flag to see which config files are loaded

---

## File Inventory

| File | Purpose |
|---|---|
| `opencode.jsonc` | Main config — models, providers, permissions, formatters, watcher ignore. Rename to `.json` when installing. |
| `AGENTS.md` | Global system prompt — identity, tech stack defaults, coding standards, prototyping workflow, skill references. |
| `agents/architect.md` | Subagent for architecture planning. Read-only tools. Outputs data models, API surfaces, file plans. |
| `agents/reviewer.md` | Subagent for code review. Read-only tools. Outputs Critical/Important/Suggestions/Praise. |
| `agents/fixer.md` | Subagent for debugging. Full tool access. Follows reproduce→trace→fix→verify protocol. |
| `commands/proto.md` | `/proto` — scaffold new projects (Next.js, Hono, Express, CLI, monorepo). |
| `commands/arch.md` | `/arch` — plan architecture via architect subagent. |
| `commands/review.md` | `/review` — code review via reviewer subagent. |
| `commands/fix.md` | `/fix` — debug errors via fixer subagent. |
| `commands/test.md` | `/test` — run, generate, or set up tests. |
| `commands/deploy.md` | `/deploy` — pre-deploy checks, platform config, CI setup. |
| `skills/project-scaffold/` | Full project scaffolding — Next.js, Hono, Express, CLI, Turborepo. |
| `skills/iterative-ui/` | 5-layer UI building method: structure → data → design → interactions → edge cases. |
| `skills/api-design/` | API contract design — REST, tRPC, CLI interfaces. Zod validation, error handling. |
| `skills/database-design/` | Drizzle ORM schemas, relations, migrations, seed scripts, SQLite→Postgres migration. |
| `skills/auth-patterns/` | Authentication setup — Better-auth, Lucia, Auth.js, Clerk. Sessions, OAuth, RBAC. |
| `skills/deployment-ready/` | Production prep — env validation, Docker, GitHub Actions CI, security headers, health checks. |
| `skills/testing-strategy/` | Focused testing for prototypes — Vitest, Playwright, what to test vs skip. |
| `skills/debug-detective/` | Systematic debugging protocol — reproduce, trace, hypothesize, fix, verify. |
| `skills/code-review/` | Code review checklist — correctness, security, types, performance. |
| `skills/cli-builder/` | CLI tool patterns — Commander.js + Clack (TS), Cobra (Go). Structure, output, exit codes. |
