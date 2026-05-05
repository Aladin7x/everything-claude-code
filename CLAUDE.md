# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**everything-claude-code** (`ecc-universal` v1.8.0) is a production-ready Claude Code plugin — a curated collection of agents, skills, hooks, commands, rules, and MCP configurations evolved over 10+ months of intensive daily use. It works across multiple AI harnesses: Claude Code, Codex, Cursor IDE, and OpenCode.

- MIT License · Node.js ≥ 18 required
- CLI entry points: `npx ecc` and `npx ecc-install`
- Primary dependency: `sql.js` (state persistence)

## Running Tests

```bash
# Full suite (CI validators + unit tests)
npm test

# Unit tests only
node tests/run-all.js

# Individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js

# Coverage (requires 80% threshold)
npm run coverage

# Linting
npm run lint          # ESLint + markdownlint
```

CI validators run first and check structural integrity across agents, commands, rules, skills, hooks, install manifests, and personal path leakage.

## Architecture

```
everything-claude-code/
├── agents/          # 25 specialized subagents (Markdown + YAML frontmatter)
├── skills/          # 108 skill directories, each with SKILL.md
├── commands/        # 57 slash command definitions (Markdown)
├── hooks/           # hooks.json config + README
├── rules/           # 50 coding-standard files (common/ + 8 language dirs)
├── mcp-configs/     # mcp-servers.json with 22 server definitions
├── scripts/         # 163 Node.js/Shell utilities
│   ├── ci/          # Structural validators run in CI
│   ├── hooks/       # Hook implementation scripts
│   └── lib/         # Shared libraries (package-manager, session-manager, etc.)
├── tests/           # Test suite (hooks/, scripts/, ci/, integration/)
├── contexts/        # Context templates (dev.md, review.md, research.md)
├── examples/        # Example CLAUDE.md files for project types
├── schemas/         # JSON Schema definitions for validation
├── manifests/       # Installation manifests and profiles
├── docs/            # Documentation + translations (ja-JP, ko-KR, zh-CN, zh-TW)
├── .claude/         # Claude Code plugin config
├── .codex/          # Codex harness support
├── .cursor/         # Cursor IDE support
└── .opencode/       # OpenCode harness (TypeScript, full config)
```

### agents/

25 Markdown files with YAML frontmatter. Each agent is a specialized subagent delegated to by Claude Code.

**Available agents:**
- `architect.md` — system design and architecture planning
- `build-error-resolver.md` — generic build failure triage
- `chief-of-staff.md` — orchestration and task decomposition
- `code-reviewer.md` — general-purpose code review
- `cpp-build-resolver.md`, `go-build-resolver.md`, `java-build-resolver.md`, `kotlin-build-resolver.md`, `rust-build-resolver.md` — language-specific build fixers
- `cpp-reviewer.md`, `go-reviewer.md`, `java-reviewer.md`, `kotlin-reviewer.md`, `python-reviewer.md`, `rust-reviewer.md` — language-specific code reviewers
- `database-reviewer.md` — database schema and query review
- `doc-updater.md` — documentation maintenance
- `docs-lookup.md` — documentation retrieval
- `e2e-runner.md` — end-to-end test execution
- `harness-optimizer.md` — Claude Code harness performance tuning
- `loop-operator.md` — autonomous loop management
- `planner.md` — implementation planning
- `refactor-cleaner.md` — code cleanup and refactoring
- `security-reviewer.md` — security audit
- `tdd-guide.md` — test-driven development coaching

**Agent format:**
```yaml
---
name: agent-name
description: One-sentence description for agent selection heuristics.
tools: ["Read", "Grep", "Glob", "Bash"]
model: sonnet   # haiku | sonnet | opus
---
# Role
...
# Workflow
...
# Output Format
...
```

Model selection: `haiku` for fast/simple tasks, `sonnet` for general work, `opus` for complex reasoning.

### skills/

108 directories, each containing `SKILL.md`. Skills are invoked by users or referenced by agents.

**Skill categories:**
- Language patterns: `golang-patterns`, `python-patterns`, `rust-patterns`, `kotlin-patterns`, `swift-patterns`, `java-patterns`
- Framework patterns: `laravel-patterns`, `django-patterns`, `springboot-patterns`
- Testing/quality: `tdd-workflow`, `verification-loop`, `ai-regression-testing`, `eval-harness`, `e2e-testing`
- Architecture: `backend-patterns`, `frontend-patterns`, `api-design`, `mcp-server-patterns`, `deployment-patterns`
- LLM/AI: `claude-api`, `deep-research`, `continuous-agent-loop`, `autonomous-loops`, `cost-aware-llm-pipeline`
- Infrastructure: `docker-patterns`, `postgres-patterns`, `clickhouse-io`, `database-migrations`
- Domain-specific: `energy-procurement`, `logistics-exception-management`, `customs-trade-compliance`
- Tools: `security-scan`, `security-review`, `exa-search`, `fal-ai-media`, `claude-devfleet`

**Skill format:**
```yaml
---
name: skill-name
description: One-sentence summary.
origin: ECC
---
# Skill Name
## When to Use
## How It Works
## Examples
## Best Practices
```

### commands/

57 slash command Markdown files. Users invoke these as `/command-name` inside Claude Code.

**Key commands:**
- `/tdd` — test-driven development workflow
- `/plan` — implementation planning
- `/e2e` — generate and run E2E tests
- `/code-review` — quality review
- `/build-fix` — fix build errors
- `/refactor-clean` — clean up code
- `/skill-create` — generate skills from git history
- `/learn` — extract patterns from sessions
- `/loop-start`, `/loop-status` — manage autonomous loops
- `/harness-audit` — audit Claude Code configuration
- `/quality-gate` — run quality checks
- `/model-route` — intelligent model routing

**Command format:**
```yaml
---
description: Brief description shown in /help
---
# Command Name
## Purpose
## Usage
## Workflow
## Output
```

### hooks/

`hooks.json` defines event-driven automations. Hook scripts live in `scripts/hooks/`.

**Hook lifecycle events:**
- `PreToolUse` — runs before any tool call (Bash, Edit, Write, etc.)
- `PostToolUse` — runs after tool call completes
- `SessionStart` — fires when a Claude Code session opens
- `PreCompact` — fires before context compaction
- `Stop` — fires when Claude finishes responding
- `SessionEnd` — fires when the session closes

**Active hooks (summary):**
- Auto-tmux dev environment setup
- Git push reminders
- Documentation file warnings
- Strategic compaction suggestions
- Continuous learning observer
- InsAIts security monitoring
- PR activity logging
- Build analysis after Bash
- Quality gate enforcement
- Prettier auto-format on edit
- TypeScript check post-edit
- Console.log detection warnings
- Session context loading on start
- State saving before compact
- Session summary and pattern extraction on stop
- Cost tracking

**Hook format in hooks.json:**
```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [{"type": "command", "command": "node scripts/hooks/my-hook.js"}],
      "description": "What this hook does"
    }
  ]
}
```

Exit codes: `0` = success/allow, `2` = block the tool call.

### rules/

50 Markdown files organized into common principles and language-specific extensions.

```
rules/
├── common/           # Universal rules (9 files)
│   ├── coding-style.md
│   ├── git-workflow.md
│   ├── testing.md
│   ├── performance.md
│   ├── patterns.md
│   ├── hooks.md
│   ├── agents.md
│   ├── security.md
│   └── development-workflow.md
├── python/           # Python-specific (5 files each)
├── golang/
├── typescript/
├── kotlin/
├── cpp/
├── perl/
├── php/
└── swift/
```

Language-specific files extend common rules:
```markdown
> This file extends [common/coding-style.md](../common/coding-style.md) with Python-specific content.
```

### mcp-configs/

`mcp-servers.json` configures 22 MCP servers:

| Server | Purpose |
|--------|---------|
| github | GitHub operations |
| firecrawl | Web scraping |
| supabase | Database |
| memory | Persistent memory |
| sequential-thinking | Chain-of-thought |
| vercel / railway | Deployments |
| cloudflare-docs | CF documentation |
| cloudflare-workers-* | Workers bindings |
| clickhouse | Analytics queries |
| exa-web-search | Research |
| context7 | Live docs lookup |
| magic | UI components |
| filesystem | File operations |
| insaits | AI security monitoring |
| playwright | Browser automation |
| fal-ai | Image/video/audio generation |
| browserbase | Cloud browser sessions |
| browser-use | AI browser agent |
| devfleet | Multi-agent orchestration |
| token-optimizer | Context reduction |
| confluence | Confluence integration |

### scripts/

**CLI tools:**
- `ecc.js` — main CLI (`npx ecc`)
- `doctor.js` — health check
- `status.js` — status reporting
- `install-apply.js` / `install-plan.js` — installation
- `harness-audit.js` — configuration audit
- `repair.js` — fix broken state
- `uninstall.js` — remove installation
- `list-installed.js` — show installed components
- `orchestrate-worktrees.js` / `orchestrate-codex-worker.sh` — multi-agent orchestration
- `skill-create-output.js` — skill generation from git history
- `session-inspect.js` — session state inspection
- `sessions-cli.js` — session management CLI
- `claw.js` — claw utilities

**CI validators (`scripts/ci/`):**
- `validate-agents.js` — check agent frontmatter
- `validate-commands.js` — check command format
- `validate-rules.js` — check rule structure
- `validate-skills.js` — check skill structure
- `validate-hooks.js` — check hooks.json schema
- `validate-install-manifests.js` — check install manifests
- `validate-no-personal-paths.js` — reject leaked personal paths
- `catalog.js` — generate component catalog

**Libraries (`scripts/lib/`):**
- `package-manager.js` — detect npm/pnpm/yarn/bun
- `session-manager.js` — session state management
- `project-detect.js` — project type detection
- `install-executor.js` — installation logic
- `state-store/` — SQL.js-based persistent state
- `skill-evolution/` — skill versioning and tracking
- `install-targets/` — installation target strategies

## Development Workflows

### Adding a New Agent

1. Create `agents/your-agent-name.md`
2. Add YAML frontmatter: `name`, `description`, `tools`, `model`
3. Write Role, Workflow, Output Format, Examples sections
4. Run `node scripts/ci/validate-agents.js` to verify
5. Run `npm test` to pass full suite

### Adding a New Skill

1. Create directory `skills/your-skill-name/`
2. Create `skills/your-skill-name/SKILL.md` with frontmatter
3. Include: When to Use, How It Works, Examples, Best Practices
4. Keep under 500 lines
5. Run `node scripts/ci/validate-skills.js`

### Adding a New Command

1. Create `commands/your-command.md`
2. Add frontmatter with `description` field
3. Include: Purpose, Usage, Workflow, Output sections
4. Run `node scripts/ci/validate-commands.js`

### Adding a Hook

1. Write the hook script in `scripts/hooks/`
2. Add entry to `hooks/hooks.json` with correct matcher and lifecycle event
3. Run `node scripts/ci/validate-hooks.js`
4. Use exit code `0` (allow) or `2` (block)

### Updating Rules

- Modify `rules/common/` for universal changes
- Modify `rules/<language>/` for language-specific additions
- Language files must reference their parent common file

### Running Validation Only

```bash
node scripts/ci/validate-agents.js
node scripts/ci/validate-commands.js
node scripts/ci/validate-skills.js
node scripts/ci/validate-hooks.js
node scripts/ci/validate-no-personal-paths.js
```

## File Naming Conventions

- All Markdown files: `lowercase-with-hyphens.md`
- Skill directories: `lowercase-with-hyphens/`
- JavaScript files: `lowercase-with-hyphens.js`
- Skill content files: `SKILL.md` (uppercase)
- Documentation: `CLAUDE.md`, `README.md`, `CONTRIBUTING.md`, `CHANGELOG.md` (uppercase)
- Schema files: `name.schema.json`

## Cross-Harness Support

| Harness | Directory | Notes |
|---------|-----------|-------|
| Claude Code | `.claude/` | Primary target |
| Codex (OpenAI) | `.codex/`, `.agents/` | Subset of skills |
| Cursor IDE | `.cursor/` | Rules + skills subset |
| OpenCode | `.opencode/` | Full TypeScript config |

When adding skills, consider mirroring to `.agents/skills/` for Codex and `.cursor/skills/` for Cursor if broadly applicable.

## Package Manager Support

Detection order: `CLAUDE_PACKAGE_MANAGER` env var → project config → auto-detect (npm, pnpm, yarn, bun).

Configure via `.claude/package-manager.json` or the `CLAUDE_PACKAGE_MANAGER` environment variable.

## Schemas

JSON Schema files in `schemas/` validate all structured data:
- `hooks.schema.json` — hooks.json structure
- `ecc-install-config.schema.json` — install configuration
- `plugin.schema.json` — plugin manifest
- `state-store.schema.json` — persistent state
- `package-manager.schema.json` — package manager config
- `install-components.json`, `install-modules.json`, `install-profiles.json` — installation manifests

## Contributing

See `CONTRIBUTING.md` for full guidelines. Summary:

- PR title format: `feat(skills): add rust-patterns skill`
- File naming: lowercase with hyphens
- No personal paths or sensitive data in any file
- Test before submitting: `npm test`
- Keep skills under 500 lines
- Include practical examples in every skill and agent

Component-specific checklists:
- **Agents**: frontmatter complete, tools list accurate, model appropriate, workflow clearly structured
- **Skills**: frontmatter + When to Use + How It Works + Examples + Best Practices
- **Commands**: description frontmatter + Purpose + Usage + Workflow + Output
- **Hooks**: valid JSON, correct lifecycle event, exit code documented, script tested
