# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin** - a collection of production-ready agents, skills, hooks, commands, rules, and MCP configurations. The project provides battle-tested workflows for software development using Claude Code.

## Production AI Workflow Architecture

Every feature, agent, or automation built here must follow this end-to-end workflow. AI should operate **inside** a workflow, not outside it. Every action needs context, validation, and accountability.

### Step 1 — Start With the Workflow (not the model)

Before writing any code or prompt, answer these seven questions:

| # | Question | Maps to |
|---|----------|---------|
| 1 | What starts the workflow? | **Input / Trigger** |
| 2 | What information does AI need? | **Context** |
| 3 | What should AI decide, classify, summarize, or generate? | **Decision** |
| 4 | What system should it touch? | **Tool / Action** |
| 5 | How will the result be checked? | **Validation** |
| 6 | Where is human judgement required? | **Human Approval** |
| 7 | What does success look like? | **Output** |

### Step 2 — Eight-Layer Real Workflow Architecture

#### A · Trigger Layer — how the workflow begins
Accepted entry points in this project:
- User query (slash command or chat)
- Uploaded file or pasted content
- Hook event (PreToolUse / PostToolUse / Stop)
- GitHub webhook / CI event

#### B · Context Assembly Layer — gather the right inputs
Claude must pull relevant context before acting:
- Project docs (`CLAUDE.md`, `CONTRIBUTING.md`, skill files)
- Rules (`rules/`) and policies
- Past session history (hooks persist state)
- Vector search / examples (skills, agent prompts)

#### C · AI / Decision Layer — interpret, reason, generate
Agents and commands in this repo perform one of:
- **Classify** — route to the right agent or skill
- **Summarize** — condense context for downstream steps
- **Route** — delegate to a subagent (`agents/`)
- **Draft / Extract** — produce code, plans, or structured output

#### D · Tool Orchestration Layer — take action through systems
Allowed tool surfaces (configured in `mcp-configs/` and `settings.json`):
- **GitHub** — PRs, issues, code search
- **Filesystem** — Read / Edit / Write (scoped to workspace)
- **Shell / Bash** — Node.js scripts only, no destructive flags
- **External APIs** — via approved MCP servers listed in `mcp-configs/`

New tool integrations require a new entry in `mcp-configs/` and an updated permission allowlist.

#### E · Validation & Guardrails — check before action
Every automated action must pass at least one check:
- **Schema / syntax check** — `node tests/run-all.js` passes
- **Policy check** — complies with `rules/` guardrails
- **Confidence threshold** — agent signals uncertainty → escalate to human
- **Source / citation check** — external data marked `<untrusted_external_data>`
- **Test / verification** — new scripts require a matching test in `tests/`

Hooks (`hooks/`) are the enforcement mechanism: PreToolUse hooks can block unsafe actions; PostToolUse hooks validate results.

#### F · Human-in-the-Loop — approval where risk is high
Always pause and ask the user before:
- Destructive operations (delete files, drop data, force-push)
- External sends (push to remote, post comments, send emails)
- Security-sensitive changes (credentials, permissions, CI pipelines)
- Customer-impacting actions (production deployments, schema migrations)

Use `AskUserQuestion` rather than proceeding autonomously.

#### G · Output & Action — what the workflow produces
Concrete deliverables this project produces:
- Code changes (committed to the correct branch)
- Draft PR descriptions / review comments
- Reports and alerts (hook notifications)
- Updated config files (`settings.json`, MCP configs)
- Dashboard / README updates

Every output must map back to the original trigger and pass E and F before delivery.

#### H · Feedback & Improvement — close the loop
After each session or release:
- Logs written by hooks to stderr with `[HookName]` prefix
- User feedback captured and reflected in skill/agent updates
- Evaluations run against test suite (`node tests/run-all.js`)
- Prompt / policy updates committed and reviewed via PR
- Memory updates persisted through session-start hooks

### Step 3 — Production Readiness Checklist

Before merging any new agent, skill, command, or hook:

- [ ] **Clear Inputs** — trigger and required data are defined
- [ ] **Reliable Context** — pulls from docs, history, and rules
- [ ] **Action Boundaries** — tool access is scoped in `settings.json`
- [ ] **Validation** — outputs verified before delivery (tests pass, guardrail hooks active)
- [ ] **Human Oversight** — high-risk paths escalate to the user
- [ ] **Continuous Improvement** — feedback loop closes (logs, evals, memory)

---

## Running Tests

```bash
# Run all tests
node tests/run-all.js

# Run individual test files
node tests/lib/utils.test.js
node tests/lib/package-manager.test.js
node tests/hooks/hooks.test.js
```

## Architecture

The project is organized into several core components that map directly to the workflow layers above:

| Directory | Workflow Layer |
|-----------|---------------|
| **hooks/** | A · Trigger + E · Validation & Guardrails |
| **rules/** | E · Validation & Guardrails |
| **agents/** | C · AI / Decision Layer |
| **skills/** | B · Context Assembly + C · AI / Decision |
| **commands/** | A · Trigger Layer (user-invoked) |
| **mcp-configs/** | D · Tool Orchestration Layer |
| **scripts/** | D · Tool Orchestration + H · Feedback |
| **tests/** | E · Validation & Guardrails |

## Key Commands

- `/tdd` - Test-driven development workflow
- `/plan` - Implementation planning
- `/e2e` - Generate and run E2E tests
- `/code-review` - Quality review
- `/build-fix` - Fix build errors
- `/learn` - Extract patterns from sessions
- `/skill-create` - Generate skills from git history

## Development Notes

- Package manager detection: npm, pnpm, yarn, bun (configurable via `CLAUDE_PACKAGE_MANAGER` env var or project config)
- Cross-platform: Windows, macOS, Linux support via Node.js scripts
- Agent format: Markdown with YAML frontmatter (name, description, tools, model)
- Skill format: Markdown with clear sections for when to use, how it works, examples
- Skill placement: Curated in skills/; generated/imported under ~/.claude/skills/. See docs/SKILL-PLACEMENT-POLICY.md
- Hook format: JSON with matcher conditions and command/notification hooks

## Contributing

Follow the formats in CONTRIBUTING.md:
- Agents: Markdown with frontmatter (name, description, tools, model)
- Skills: Clear sections (When to Use, How It Works, Examples)
- Commands: Markdown with description frontmatter
- Hooks: JSON with matcher and hooks array

**New contributions must satisfy the production readiness checklist in Step 3 above.**

File naming: lowercase with hyphens (e.g., `python-reviewer.md`, `tdd-workflow.md`)

## Skills

Use the following skills when working on related files:

| File(s) | Skill |
|---------|-------|
| `README.md` | `/readme` |
| `.github/workflows/*.yml` | `/ci-workflow` |

When spawning subagents, always pass conventions from the respective skill into the agent's prompt.
