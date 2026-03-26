# Harness Engineering Guide

*Updated: March 2026 · Claude Code v2.1.83 기준*

> **This document is both a human reference and the agent's knowledge source during harness setup.** The kickoff prompt feeds this guide to the agent via `@` reference. After setup, the agent reads only the artifacts created from this guide — not the guide itself.

## Rules

### Interaction Principles

These principles override all phase-specific instructions:

- **Accuracy over speed — overwhelmingly.** Never rush to produce output. Take time to understand deeply before proposing anything
- **Propose in small pieces.** Within each phase, propose section by section — not the entire phase at once. Get confirmation on each piece before moving on
- **When in doubt, ask — never guess.** If you are uncertain about the project's conventions, goals, or preferences, ask the designer (the human setting up the harness) rather than making assumptions
- **Disagreement means rethink, not patch.** If the designer rejects a proposal, do not simply revise the surface. Reconsider the fundamental approach

### Coding Discipline

These principles apply to all code, configuration, and documentation you produce:

- **Think before writing.** State assumptions explicitly. If multiple valid approaches exist, present them — do not pick silently. If a simpler approach exists, suggest it
- **Simplicity first.** No features beyond what was asked. No abstractions for single-use code. No speculative flexibility or configurability. If you wrote 200 lines and it could be 50, rewrite it
- **Surgical changes.** Do not "improve" adjacent code, comments, or formatting. Do not refactor things that are not broken. Match existing style even if you would do it differently. Every changed line must trace to the request
- **Define success before starting.** Transform each task into a verifiable goal. For multi-step tasks, state a brief plan with verification steps. Strong success criteria let you work independently without drifting

### Workflow

When you receive this document, **do not immediately jump to phase selection or execution.** First, understand the project.

0. **Explore the project first.** Browse the file structure, read existing configuration files (CLAUDE.md, package.json, pyproject.toml, etc.), and discuss what you observe with the designer. Only after the designer confirms that you understand the project well enough, proceed to phase selection
1. Read the project description (or ask for one)
2. Select only the phases this project actually needs — skip the rest
3. For each selected phase: propose section by section → get approval on each → execute
4. After completing a phase, ask whether to proceed to the next
5. Never set up everything at once — each phase must be reviewed before moving on

Phase 1 (CLAUDE.md) applies to **every project**. All other phases are conditional.

**Selection criteria for optional phases:**

| Phase | Include when... |
|-------|----------------|
| 2. Project Settings | Specific tool restrictions, bash commands, or mode preferences |
| 3. Skills & Subagents | Domain knowledge packaging (3A), role delegation (3B), or both |
| 4. Hooks | Repetitive pre/post actions should run automatically |
| 5. MCP Servers | External tool access needed (databases, browsers, APIs) |
| 6. Code Structure | Codebase has 3+ distinct domains requiring consistent organization |
| 7. Working Memory, Parallel Work & Agent Teams | Tasks span multiple sessions, require parallel worktrees, or need coordinated multi-agent collaboration |

**Guardrails:**
- If the project is a small script or single-purpose tool, Phase 1 alone is likely sufficient
- Propose at most 3-4 phases for a typical project — justify each
- When in doubt, skip a phase — it can always be added later
- Present your phase selection like this:

```
Based on this project, I recommend:

[Required]
1. CLAUDE.md — every project needs this

[Recommended]
4. Hooks — reason
7. Working Memory — reason

[Skip]
2, 3, 5, 6 — reasons not needed

Proceed with Phase 1?
```

---

## Phase 1: CLAUDE.md

The single highest-leverage file in the harness. It is loaded at the start of every session and shapes every action the agent takes. All projects need one.

### Getting started

Run `/init` to generate a starter CLAUDE.md based on your current project structure, then refine. The generated file often includes obvious things — delete aggressively. Every line that survives must earn its place.

### What to include

| Section | Purpose | Example |
|---------|---------|---------|
| Project summary | What this project is, in 1-2 sentences | "An ML pipeline for document classification using PyTorch and HuggingFace Transformers" |
| Tech stack | Languages, frameworks, key dependencies | Python 3.12, PyTorch, Transformers, pytest |
| Project structure | Top-level directory layout with brief descriptions | `src/` — application code, `tests/` — test suite |
| Commands | Build, test, lint, run — anything the agent cannot guess | `uv run pytest -x`, `make train` |
| Workflow | How changes should be made, including context compaction rules | "Always run tests before committing. When compacting, preserve modified file list and test status." |
| Gotchas | Things that frequently cause mistakes | "Do NOT modify `configs/base.yaml` directly — override in experiment configs" |
| Ubiquitous Language | Domain terms with definitions | "Epoch = one complete pass through the training dataset" |
| Harness evolution | Rule for accumulating improvement candidates | "When corrected or a pattern repeats, log to `.claude/memory/harness-candidates.md`" |

### What NOT to include

- **Code style rules that a linter can enforce** — use PostToolUse hooks for formatters instead (see Phase 4)
- Information the agent can infer by reading the code
- Long explanations — if a section exceeds 20 lines, move it to a separate file and reference it with `@path/to/file`
- Auto-generated content — every line must be manually crafted and intentional
- Anything needed only sometimes — use Skills instead (see Phase 3)

### Design principles

1. **Under 150 lines.** Frontier models follow ~150-200 instructions with reasonable consistency. The agent's system prompt already uses ~50 of those. A bloated CLAUDE.md causes the agent to ignore your actual instructions uniformly — not just the later ones. If it grows beyond 150 lines, split it
2. **Every line must earn its place.** For each line, ask: "Would removing this cause the agent to make mistakes?" If not, cut it
3. **Use emphasis for critical rules.** `IMPORTANT:`, `NEVER`, `ALWAYS` — the agent pays more attention to emphasized text
4. **Pointers over copies.** Reference files with `@path/to/file` syntax instead of duplicating content. CLAUDE.md supports `@import` references that are resolved at load time:
   ```markdown
   See @README.md for project overview
   Git workflow: @docs/git-instructions.md
   ```
5. **Evolve organically.** Start minimal. When the agent makes incorrect assumptions, add the correction. The file compounds in value through real usage
6. **Treat like code.** Review it when things go wrong, prune it regularly, and test changes by observing whether the agent's behavior actually shifts

### Context compaction rule

Add a standing instruction to the Workflow section of CLAUDE.md so the agent preserves critical state when context is compressed:

```markdown
## Workflow
- When compacting, preserve the full list of modified files and current test status.
- Manually /compact at ~50% context usage. Do not wait for automatic compaction.
- PreCompact/PostCompact hooks can automate state backup around compaction (see Phase 4).
- Use /clear when switching to a completely unrelated task mid-session.
```

This prevents loss of working state during long sessions.

### Skeleton template

```markdown
# Project Name

One-sentence project description.

## Tech Stack
- Language / framework / key dependencies

## Project Structure
- `src/` — application code
- `tests/` — test suite
- `docs/` — documentation

## Commands
# Run
command-to-run-main-task

# Testing
command-to-run-tests

## Workflow
- Step 1 before making changes
- Step 2 after making changes
- On session start: read .claude/memory/ for active plans/checklists, check git status
- When compacting, preserve: modified file list, current test status, active branch

## Ubiquitous Language
- **Term**: Definition in project context

## Gotchas
- IMPORTANT: Thing that causes frequent mistakes
- NEVER do X because Y

## Harness Evolution
- When corrected or a pattern repeats, log to `.claude/memory/harness-candidates.md`
```

### Splitting a large CLAUDE.md

When the file exceeds 150 lines, use this pattern:

```
CLAUDE.md              ← index (~60-80 lines): summary, commands, critical rules
.claude/rules/
  testing.md           ← testing conventions and patterns
  architecture.md      ← architectural decisions and constraints
  domain-patterns.md   ← domain-specific conventions
```

Files in `.claude/rules/` are automatically loaded without explicit imports. This approach reduces token consumption by 40-60% compared to a single large file while keeping all guidance accessible.

### Phase 1 artifacts

In addition to CLAUDE.md itself, create these files during Phase 1 — they support the Harness Evolution loop (see [Harness Evolution](#harness-evolution)):

- `.claude/memory/harness-candidates.md` — evidence buffer where the agent logs corrections and repeated patterns (initialize with the template in Harness Evolution)
- `.claude/commands/harness-review.md` — review command for processing accumulated candidates (see [Review command](#review-command))
- `.claude/commands/harness-health-check.md` — quarterly hygiene command (see [Health check command](#health-check-command))

---

## Phase 2: Project Settings

Configure `.claude/settings.local.json` to control agent behavior at the project level.

### What it controls

| Setting | Purpose |
|---------|---------|
| `allowedBashCommands` | Whitelist of bash commands the agent can run without asking permission |
| `permissions` | Tool-level permission overrides. Skills can be disabled via `/permissions` using `Skill(name)` deny patterns |
| `hooks` | Automatic pre/post actions (see Phase 4) |
| `sandbox` | Sandbox mode and `failIfUnavailable` option (exit with error when sandbox cannot start) |
| `managed-settings.d/` | Drop-in directory for team-level independent policy files. Enterprise and team environments |

### Permission management

Three approaches, from least to most permissive:

- **`/permissions`** — interactive permission management. Recommended starting point. Use this to allowlist specific commands and deny dangerous ones
- **`/sandbox`** — OS-level isolation. The agent runs in a sandboxed environment where it cannot affect the host system
- **`auto` mode** — a classifier handles approvals automatically. Use only when you trust the agent's judgment for the project

### Recommended permission baseline

모든 프로젝트에 적용할 수 있는 권장 permission 설정. 도구 수준에서 허용/차단을 관리하며, 위험한 bash 명령만 명시적으로 차단한다:

```json
{
  "permissions": {
    "allow": [
      "Read",
      "Write",
      "Edit",
      "Glob",
      "Grep",
      "WebSearch",
      "WebFetch",
      "Agent",
      "NotebookEdit",
      "Bash(*)"
    ],
    "deny": [
      "Bash(rm -rf /)",
      "Bash(rm -rf /*)",
      "Bash(mkfs:*)",
      "Bash(dd :*)",
      "Bash(shutdown:*)",
      "Bash(reboot:*)",
      "Bash(poweroff:*)",
      "Bash(halt:*)",
      "Bash(git push --force:*)",
      "Bash(git push -f:*)",
      "Bash(git reset --hard:*)",
      "Bash(git clean:*)"
    ]
  }
}
```

**설계 원칙:** `Bash(*)`로 모든 bash 명령을 허용하되, deny 목록으로 시스템 파괴(`rm -rf /`, `mkfs`, `dd`, `shutdown`) 및 git 위험 명령(`push --force`, `reset --hard`, `clean`)을 차단한다. deny는 allow보다 우선하므로, 넓은 허용 + 좁은 차단 조합이 가장 실용적이다.

프로젝트별로 deny 목록을 확장할 수 있다. 예: 프로덕션 DB 접근 차단 `"Bash(psql -h prod:*)"`.

### Additional: allowedBashCommands

`permissions` 대신 `allowedBashCommands`를 사용할 수도 있다. 이 방식은 허용 목록만 관리하며, 목록에 없는 명령은 매번 승인을 요청한다:

```json
{
  "allowedBashCommands": [
    "ls", "find", "tree", "cat", "head", "tail", "wc", "file", "diff",
    "grep", "rg", "which",
    "pwd", "echo", "date", "whoami", "uname", "du", "df", "env"
  ]
}
```

이 방식은 더 보수적이지만, 새 명령마다 승인이 필요해 작업 흐름이 느려진다. 프로젝트 신뢰도가 높으면 위의 permission baseline을 권장한다.

Subprocess 환경의 인증 정보 보호가 필요하면 `CLAUDE_CODE_SUBPROCESS_ENV_SCRUB=1`을 설정한다. Bash tool, hooks, MCP 서버의 subprocess에서 Anthropic 및 클라우드 인증 정보가 자동 제거된다.

As with all phases, do not provide a generic template. Analyze the project's actual tech stack and commands, then generate a settings file tailored to it.

---

## Phase 3: Skills & Subagents

Skills and Subagents are how the agent gains domain expertise and delegates specialized work.

**This phase has two independent parts.** Not every project needs both:
- **3A only** — most projects start here. Add Skills when you have domain knowledge or workflows to package
- **3A + 3B** — add Subagents when you need independent role delegation (code reviewer, QA, architect) or parallel background work

### 3A. Skills

Skills are markdown files (`SKILL.md`) in `.claude/skills/` that provide domain-specific knowledge the agent loads on demand.

> **Note:** Custom commands(`.claude/commands/`)는 Skills로 통합되었다. 기존 commands 파일은 그대로 동작하며, 새 워크플로우는 Skills로 작성을 권장한다.

**Core mechanism — Progressive Disclosure:** The agent reads only the YAML frontmatter (`name` and `description`) of each skill at session start. The full content loads only when the agent determines the skill is relevant. This keeps context lean — unlike CLAUDE.md which loads every session.

**SKILL.md format:**

```markdown
---
name: data-visualization
description: >
  Standards for data visualization. Use when creating charts,
  plots, or dashboards. Covers font settings, color palettes,
  interactive visualization library selection.
---

# Data Visualization Standards

## Font Configuration
- Always configure Korean font fallback for CJK support
- ...

## Library Selection
- Static plots: matplotlib with seaborn styling
- Interactive: Plotly
- ...
```

Use `${CLAUDE_SKILL_DIR}` to reference supporting files (templates, scripts, reference docs) relative to the skill directory.

**Key frontmatter fields:**

| Field | Purpose |
|-------|---------|
| `name` | Display name and `/slash-command` trigger |
| `description` | When to invoke — **write this carefully**, the agent uses it to decide activation |
| `argument-hint` | Autocomplete hint (e.g., `[issue-number]`) |
| `disable-model-invocation` | Set `true` to prevent automatic invocation — manual `/name` only |
| `user-invocable` | Set `false` to hide from slash menu (background knowledge only) |
| `allowed-tools` | Tools allowed without permission prompts when skill is active |
| `context` | `fork` to run in an isolated subagent. Skill body becomes the subagent prompt |
| `agent` | Subagent type for fork execution (Explore, Plan, general-purpose, or custom) |
| `effort` | Override model effort level when this skill is invoked |
| `hooks` | Lifecycle hooks scoped to this skill's activation |

**Using Skills as independent execution units (`context: fork`):**

Setting `context: fork` transforms a Skill from passive knowledge injection into an active worker. The Skill body becomes the subagent's prompt, executing in a separate context, and only the result returns to the main session.

Practical patterns: research Skills (Explore agent scans codebase and returns summary), security review Skills (read-only tools for vulnerability scanning), documentation generation Skills (writes docs without polluting main context).

`context: fork` only makes sense when the Skill contains an explicit task. Do not use it for guideline-only Skills (e.g., "follow these API conventions") — those need to stay in the main context to influence ongoing work.

**Where to place skills:**

| Location | Scope |
|----------|-------|
| `.claude/skills/<skill-name>/SKILL.md` | Project-specific |
| `~/.claude/skills/<skill-name>/SKILL.md` | Global (all projects) |

**Skill vs CLAUDE.md — when to use which:**

| Content type | Put in... | Reason |
|-------------|-----------|--------|
| Every session needs it (commands, workflow, gotchas) | CLAUDE.md | Always loaded |
| Specific situations only (visualization rules, deployment checklist) | Skill | Loaded on demand, saves tokens |

**Building your own skills:**

Don't write skills upfront. Let them emerge from real usage through the Harness Evolution loop (see [Harness Evolution](#harness-evolution)). The pattern: you give a correction → it happens again → it gets logged as a candidate → at review, situation-specific candidates become Skills. This ensures every skill matches your actual workflow.

**Warning:** More skills is not better. A skill that activates in the wrong context can steer work in an unwanted direction. Only keep skills that match your actual workflow. If a skill keeps activating when it shouldn't, either narrow its `description` or set `disable-model-invocation: true`.

### 3B. Subagents

Subagents are specialized roles defined as markdown files in `.claude/agents/`. Each subagent gets its own system prompt, tool restrictions, and optionally its own model.

**Built-in subagents**

Before creating custom subagents, know what's already available:

| Name | Model | Tools | Purpose |
|------|-------|-------|---------|
| Explore | Haiku | Read, Grep, Glob | Fast codebase search. "Where is X defined?" |
| Plan | — | Read-only | Scans code and returns distilled summary in plan mode |
| general-purpose | Inherits | All | Full read/write/execute capability |
| Claude Code Guide | Haiku | Docs lookup | Answers questions about Claude Code features |

Claude invokes these automatically based on context. For example, in plan mode it spawns Explore to scan the repo so your main conversation stays lean. Create custom subagents only when these are insufficient.

**When to use a custom Subagent instead of a Skill:**

| Situation | Use |
|-----------|-----|
| Provide domain knowledge or patterns | Skill |
| Define a repeatable workflow | Skill |
| Run an isolated task with result returned (`context: fork`) | Skill |
| Delegate an independent role (reviewer, QA, architect) | Subagent |
| Run work in parallel without blocking the main conversation | Subagent (background) |
| Isolate work in its own git worktree | Subagent (isolation: worktree) |

**Subagent format:**

```markdown
---
name: code-reviewer
description: >
  Staff-engineer-level code review. Invoke when reviewing
  changes before PR submission.
tools: Read, Glob, Grep
model: sonnet
---

You are a senior code reviewer. Focus on:
1. Critical issues first (security, data integrity, race conditions)
2. Informational issues second (style, naming, minor optimizations)

For each issue:
- State the file and line
- Explain the risk
- Suggest a fix

Do NOT auto-fix. Report findings only.
```

**Key frontmatter fields:**

| Field | Purpose |
|-------|---------|
| `name` | Identifier, used with `@agent-name` mention |
| `description` | When to invoke |
| `tools` | Allowed tools (restricts what the subagent can do) |
| `disallowedTools` | Explicitly denied tools |
| `model` | Model override (`opus`, `sonnet`, `haiku`, or full model ID) |
| `memory` | `user` for persistent memory directory across sessions, omit for stateless |
| `background` | `true` to run without blocking the main conversation |
| `isolation` | `worktree` to run in its own git worktree |
| `permissionMode` | Permission handling (`default`, `acceptEdits`, `dontAsk`, `bypassPermissions`, `plan`). Critical for worktree-isolated agents |
| `initialPrompt` | Auto-submit a first turn on invocation. Useful for automated workflows |
| `skills` | Skills to preload into this subagent's context |
| `effort` | Override model effort level |
| `maxTurns` | Maximum conversation turns before stopping |
| `hooks` | Lifecycle hooks scoped to this subagent |

**Foreground vs Background:**
- **Foreground** (default): blocks the main conversation until complete. Permission prompts and questions pass through to you. Use for tasks where you need to see the result before continuing
- **Background**: runs concurrently. The main conversation continues immediately. Use for independent tasks like running a full test suite, generating documentation, or performing a codebase audit while you work on something else

**How to invoke subagents:**

- **Natural language**: "Use the code-reviewer to check my changes" — Claude delegates automatically
- **@mention**: Type `@code-reviewer` to force a specific subagent
- **Session-wide**: `claude --agent code-reviewer` to use the subagent for the entire session

**Where to place subagents:**

| Location | Scope |
|----------|-------|
| `.claude/agents/` | Project-specific |
| `~/.claude/agents/` | Global (all projects) |

---

## Phase 4: Hooks

### What

Hooks are shell commands, model prompts, or agent invocations that execute automatically at specific points in the agent's workflow. Unlike CLAUDE.md (advisory, ~80% adherence), hooks are **deterministic and guaranteed** to run.

### Hook events

Hooks you will configure most often when designing a harness:

| Event | When it fires | Use for | Notes |
|-------|--------------|---------|-------|
| **PreToolUse** | Before the agent calls a tool | Validating inputs, injecting context, security gates | **Only event that can block execution.** Returns `approve`, `deny`, or `suppressOutput` |
| **PostToolUse** | After a tool succeeds | Formatting, linting, logging modifications | PostToolUseFailure handles failed tool calls separately |
| **Stop** | When the agent finishes a turn | Committing work, running verification, notifications | StopFailure handles abnormal termination |
| **SessionStart** | When a new session begins | Loading recent context, setting environment variables | Env vars via `CLAUDE_ENV_FILE` |
| **PreCompact** | Before context compaction | Backing up working state, memory snapshots | PostCompact fires after compaction for state restoration |
| **Notification** | When the agent needs attention | Desktop notifications, sound alerts | Fires on permission prompts, idle state, auth events |
| **SubagentStop** | When a subagent completes | Post-processing subagent results, logging | SubagentStart fires at subagent launch |

Additional events exist for specialized use cases: UserPromptSubmit, PermissionRequest, Setup, SessionEnd, CwdChanged, FileChanged, WorktreeCreate/WorktreeRemove, and others. Full event list and specs: [Hooks reference](https://code.claude.com/docs/en/hooks)

### Handler types

| Type | Executor | Use for | Example |
|------|----------|---------|---------|
| `command` | Shell process | Mechanical tasks — formatting, git, file validation | `npx prettier --write "$CLAUDE_FILE_PATH"` |
| `prompt` | Model (no tools) | Semantic judgment — "Does this change violate our security policy?" | Security policy compliance check |
| `agent` | Model (with tool access) | Deep analysis — reads files, understands context, then decides | Architecture rule violation detection |

`command` is fast and deterministic. `prompt` and `agent` cost tokens but handle judgments that mechanical rules cannot express. Most harnesses need only `command` hooks; reserve `prompt`/`agent` for high-value gates like security reviews.

### Configuration format

Hooks are defined in `.claude/settings.local.json` (or `.claude/settings.json` for shared hooks):

```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Edit|Write",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_FILE_PATH\"",
            "timeout": 30
          }
        ]
      }
    ]
  }
}
```

**Key configuration options:**

| Option | Purpose |
|--------|---------|
| `matcher` | Tool name pattern to match (regex). E.g., `Edit\|Write` |
| `type` | `"command"`, `"prompt"`, or `"agent"` |
| `command` | The shell command to execute (for `command` type) |
| `timeout` | Maximum seconds before the hook is killed |
| `async` | `true` to run in background without blocking the agent |

### Practical hook patterns

**Pattern 1: Auto-format after edit**

The most common hook. Ensures code style without putting formatting rules in CLAUDE.md:

```json
{
  "PostToolUse": [
    {
      "matcher": "Edit|Write",
      "hooks": [
        {
          "type": "command",
          "command": "npx prettier --write \"$CLAUDE_FILE_PATH\""
        }
      ]
    }
  ]
}
```

**Pattern 2: Auto WIP commit on stop**

Uses a Stop hook to automatically commit after every agent turn. Runs asynchronously so the user doesn't wait:

```json
{
  "Stop": [
    {
      "hooks": [
        {
          "type": "command",
          "command": "\"$CLAUDE_PROJECT_DIR\"/.claude/hooks/commit-session.sh",
          "async": true,
          "timeout": 120
        }
      ]
    }
  ]
}
```

Create `.claude/hooks/commit-session.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

# 1. Find git root (works inside worktrees)
GIT_ROOT=$(git rev-parse --show-toplevel 2>/dev/null) || GIT_ROOT="$CLAUDE_PROJECT_DIR"
cd "$GIT_ROOT"

# 2. Stage all changes
git add -A

# 3. Exit if nothing to commit
git diff-index --quiet HEAD -- && exit 0

# 4. Generate commit message via Claude headless mode
DIFF=$(git diff --cached | head -2000)
COMMIT_MSG=$(echo "$DIFF" | claude -p \
  "Write a git commit message for this diff.
   First line: WIP(scope): short summary (max 72 chars).
   Always use WIP as prefix. Add bullet details if needed." 2>/dev/null) || true

# 5. Fallback if claude -p fails
if [ -z "$COMMIT_MSG" ]; then
  FILE_COUNT=$(git diff --cached --name-only | wc -l | tr -d ' ')
  COMMIT_MSG="wip: update $FILE_COUNT files"
fi

# 6. Commit
echo "$COMMIT_MSG" | git commit -F - --no-verify

# Optional: auto-update CHANGELOG if docs/CHANGELOG.md exists
# See codefactory-co/kimoring-ai-skills for reference implementation
```

**Trade-off:** Every agent turn triggers a `claude -p` API call. A 20-turn session means 20 additional calls. This is worthwhile for complex parallel work or team collaboration where decision history matters, but overkill for simple solo projects. To reduce cost, remove the `claude -p` block from the script and rely on the fallback message only. Note that `claude -p` requires Claude Code CLI authentication to be configured on the machine.

**Why this matters:** WIP commits record the *decision history*, not just code snapshots. When merging later, the agent can read these commits to understand why changes were made, making conflict resolution much more informed.

**Pattern 3: Remote policy enforcement (HTTP hook)**

Validates tool calls against a central policy server. Enables team-wide rules without distributing local scripts:

```json
{
  "PreToolUse": [
    {
      "matcher": "Bash",
      "hooks": [
        {
          "type": "http",
          "url": "http://policy-server:8080/validate",
          "timeout": 30
        }
      ]
    }
  ]
}
```

The server receives the same JSON that command hooks get via stdin, as the POST request body. Return `approve` or `deny` in the response. Useful for organization-scale harnesses where policies change independently of project code.

### Key design lessons

- **Format on edit, lint on completion.** Running full linter checks after every file save wastes resources — code is naturally unstable mid-task. Auto-format is cheap; full lint is not
- **Use `async: true` for non-blocking work.** Commits, notifications, and logging should never make the user wait
- **`claude -p` in hooks.** Headless mode lets you call the agent from within hooks — powerful for generating commit messages or validation reports. Keep the piped content short to control cost
- **PreToolUse is your security gate.** It is the only hook that can block execution. Use it for file protection, command validation, and mandatory review enforcement

---

## Phase 5: MCP Servers

Model Context Protocol servers connect the agent to external tools and data sources — databases, browsers, APIs, design tools, observability platforms. Set up when the project actively uses external services during development — experiment trackers, databases for queries, observability dashboards, or APIs like Notion/Figma/Jira.

**Key principle:** More MCP connections is not better. Each connection adds tool descriptions to the context window and consumes tokens. Only connect what the project actively uses. Start with zero and add as specific needs emerge.

**Practical tip:** Use `npx`-based MCP commands for zero-install setup. Asking the agent "set up GitHub MCP" will typically generate a working `mcp.json` — you only need to provide API keys. For most projects, a single MCP (e.g., GitHub) is a reasonable starting point.

---

## Phase 6: Code Structure Guide

A structural blueprint for the codebase organized around project domains (Domain-Driven Design). Set up when the project has 3+ distinct domains, multiple contributors (human or AI), and consistency across modules matters more than flexibility.

**Key principles:**

- **Domain-first organization**: Structure code by project concept, not by technical layer
- **Ubiquitous Language**: File names and folder names must match domain terminology exactly. Mismatched names cause the agent to guess, which leads to hallucination
- **Write the first domain by hand.** The agent recognizes patterns — once one domain is well-structured, it replicates the pattern for new domains with high accuracy
- **Focused context**: Domain boundaries naturally limit what the agent needs to read

---

## Phase 7: Working Memory, Parallel Work & Agent Teams

Systems that preserve task state across sessions and enable parallel development.

### Working memory documents

For tasks that span multiple sessions, create three documents before writing any code:

1. **Plan** — What to build, from start to finish (the blueprint)
2. **Context notes** — Why decisions were made, where related resources live (the rationale)
3. **Checklist** — What is done, what remains (the progress tracker)

Store in `.claude/memory/` or `.tasks/`. Have the agent propose a plan, review it carefully, then save before implementation. Never assign all tasks at once — assign 1-2, verify results, update the checklist, then assign the next batch.

Combine with session management: load the plan at session start, update the checklist before session end, and continue with `/resume` in the next session.

### Parallel work with worktrees

Claude Code has built-in worktree support for running multiple sessions in parallel.

**Built-in basics (no setup needed):**

```bash
# Start Claude in an isolated worktree
claude -w feature-auth

# Start another parallel session
claude -w bugfix-123

# With tmux for background work
claude -w refactor-api --tmux
```

Each worktree gets its own directory (`.claude/worktrees/<name>/`), its own branch, and its own independent file state. Sessions don't interfere with each other.

Subagents can also use worktree isolation:
```yaml
---
name: refactor-agent
description: Handles large-scale refactoring
isolation: worktree
---
```

On session exit, Claude handles cleanup — worktrees with no changes are auto-removed; for worktrees with changes, you choose to keep or remove.

**Advanced: Auto WIP commits for decision tracking**

The built-in worktree support handles creation and cleanup, but does not automatically commit work-in-progress. For projects where decision history matters (e.g., understanding *why* changes were made during merge conflict resolution), add the Stop Hook from Phase 4 (Pattern 2).

The workflow becomes:
1. `claude -w feature-x` — create worktree and start session
2. Work progresses, Stop Hook auto-commits after each turn with AI-generated `WIP(scope): summary` messages
3. When done, squash merge to main: all WIP commits collapse into one clean commit, but the decision history remains in the branch for reference
4. Exit session, remove worktree

**Squash merge for clean history:**

WIP commits are valuable as working history but should not pollute the main branch. Use `git merge --squash` or PR-based squash merge to combine all worktree commits into a single, comprehensive commit.

**Automating the merge:** A `/merge-worktree` skill can automate this entire flow — validate the worktree environment, analyze commit history, perform `git merge --squash`, and craft a comprehensive commit message. See `prompts/examples/merge-worktree-SKILL.md` in the starter kit for a ready-to-use template.

**Scaling guideline:** The number of parallel worktrees you can manage effectively is limited by your ability to review and direct — not by the tool. Start with 2-3 parallel sessions. Add more only after you've established a comfortable review cadence.

### Agent Teams (experimental)

Worktrees and Subagents provide **independent** parallel execution. Agent Teams provide **collaborative** parallel execution — teammates communicate directly with each other, share a task list, and self-coordinate without routing everything through a lead.

**When to use:**

| Situation | Tool |
|-----------|------|
| Independent task, only result needed | Subagent |
| File isolation needed | Worktree |
| Workers need real-time communication and coordination | Agent Teams |
| Sequential single-track work | Single session |

**Trade-off:** Token usage ~7x compared to a single session in plan mode. Each teammate is a separate Claude instance. Best for 2-3 independent tracks within a feature (e.g., API layer + frontend + tests). The recommended pattern is "plan first, team second" — use plan mode to finalize the approach cheaply, then hand the plan to a team for parallel execution.

Agent Teams are not suitable when teammates would need to edit the same files simultaneously — file conflicts remain a fundamental limitation.

**Constraints:** Experimental; requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`. No session resume for in-process teammates. Maximum 16 agents per team. No nested teams. Until the feature stabilizes, prefer subagent + worktree combinations for critical work.

---

## Starting Development

Harness 구축이 끝나면 일상 개발로 전환한다. 이 섹션은 에이전트와 사용자 모두를 위한 운영 가이드다.

### Session start routine

매 세션 시작 시 에이전트는 다음을 수행한다:

1. CLAUDE.md를 읽고 프로젝트 컨텍스트를 로딩
2. `.claude/memory/`의 plan, checklist, context notes가 있으면 읽기
3. `git status`와 최근 커밋 로그로 현재 상태 파악
4. 진행 중인 작업이 있으면 요약, 없으면 사용자 지시 대기

이 루틴은 CLAUDE.md의 Workflow 섹션에 명시하여 매 세션 자동으로 따르게 한다.

### Working rhythm

| 시점 | 에이전트가 하는 것 | 사용자가 하는 것 |
|------|------------------|----------------|
| 작업 시작 전 | 성공 기준을 명시하고 확인 요청 | 기준을 검토하고 승인/조정 |
| 코드 변경 후 | 관련 테스트 실행, hook에 의한 자동 포매팅 | 결과 확인 |
| context ~50% | `/compact` 실행 또는 제안 | 승인 |
| 교정 발생 시 | `.claude/memory/harness-candidates.md`에 기록 | (자동, 개입 불필요) |
| 세션 종료 전 | checklist 업데이트, 후보 축적 알림 | 다음 세션 계획 확인 |

### Parallel development

병렬 작업이 필요할 때:

```bash
# 독립 작업: worktree
claude -w feature-auth
claude -w bugfix-payment

# 협업 작업: Agent Teams (experimental)
# 세션 안에서 "agent team을 만들어서 API, frontend, tests를 병렬 처리해줘"
```

### Session management

| 명령 | 용도 |
|------|------|
| `claude -n "name"` | 세션에 이름 부여 |
| `/resume` | 이전 세션 재개 |
| `/branch` | 현재 대화를 분기하여 다른 접근 시도 |
| `/rewind` | 특정 지점으로 되감기 |
| `/compact` | 수동 context 압축 |
| `/clear` | 완전히 다른 작업으로 전환 시 |

---

## Harness Evolution

Phases 1-7 cover initial setup. This section covers what happens after — how the harness improves through real usage. The core mechanism is a **candidate buffer**: instead of modifying the harness immediately when a problem is noticed, evidence is accumulated and reviewed periodically.

### The loop

```
Signal detected → Log candidate → Review trigger → Propose changes → User approves → Apply to harness
```

### Signal types

| Signal | Example | Candidate target |
|--------|---------|-----------------|
| User correction (2+ times) | "Use absolute imports" repeated | CLAUDE.md Code Style |
| Repeated manual instruction | "Run the linter" every session | Hook (PostToolUse) |
| Agent self-correction | Notices own code doesn't match project style | CLAUDE.md Gotchas |
| Environment change | New dependency added to pyproject.toml | CLAUDE.md Tech Stack |
| Painful mistake | Editing a config file that breaks everything | CLAUDE.md Gotchas |
| Recurring situation-specific instruction | Same visualization rules given 2+ times | Skill |
| Repeated role delegation | "Review this as a senior engineer" every PR | Subagent |
| Same Skill/Agent copied to 3+ projects | Copy-pasting `.claude/` between repos | Plugin (see Sharing) |

### Candidate file: `.claude/memory/harness-candidates.md`

Initialize this file during Phase 1 setup. The agent appends to it when signals are detected.

```markdown
# Harness Improvement Candidates

Review with `/project:harness-review` when candidates accumulate.

## Pending

### CLAUDE.md candidates

- **[correction ×2]** Add to Code Style: "Use absolute imports"
  - 2026-02-20: user correction "don't use relative imports"
  - 2026-02-22: same correction repeated

### Hook candidates

- **[repeated ×3]** PostToolUse: auto-run `uv run ruff check` after Edit
  - 2026-02-20, 21, 22: manual lint instruction each session

### Skill candidates

- **[repeated ×2]** Data visualization rules (font, color, library selection)
  - 2026-02-21, 22: same visualization instructions given manually

### Subagent candidates

- **[repeated ×3]** Senior engineer code review before PR
  - 2026-02-20, 21, 22: "review this as a staff engineer" instruction

## Applied (history)

- ~~Add "prefer dataclasses" to Code Style~~ → applied 2026-02-19
```

**Design rules for the candidate file:**
- **Record occurrence count.** 1 occurrence may be one-off. 2+ is a pattern
- **Record date and brief context.** Enables informed review decisions
- **Classify by target phase.** CLAUDE.md / Hook / Skill / Subagent — faster review
- **Keep applied history.** Prevents the same candidate from resurfacing

### Review command

Create `.claude/commands/harness-review.md` during Phase 1 setup:

```markdown
Review harness improvement candidates and propose concrete changes.

1. Read `.claude/memory/harness-candidates.md`
2. Read current harness files: CLAUDE.md, `.claude/settings.local.json`, skills and agents in `.claude/`
3. For each candidate with 2+ occurrences:
   - Propose a specific, minimal change (exact text to add or modify)
   - State which file to change and why (CLAUDE.md vs Skill vs Subagent vs Hook)
   - Cite the evidence (occurrence count and dates)
   - **Classification guide:** Candidates that apply *every session* → CLAUDE.md or Hook. Candidates that are *situation-specific* (not needed every session) → Skill. Candidates that involve *role delegation* (independent judgment, restricted tools) → Subagent. This is the recommended path for building Phase 3 — not writing Skills upfront.
4. For candidates with 1 occurrence:
   - List as "monitoring" — do not propose changes yet
   - Exception: if impact is high (e.g., data loss, broken builds), propose anyway
5. Present proposals as a numbered list. Wait for user to approve by number
6. Apply only approved changes to the actual harness files
7. Move applied candidates to "Applied (history)" section
8. Keep rejected candidates with a "deferred" tag, or remove if user says so
```

### When to review

| Trigger | Method | Recommended for |
|---------|--------|----------------|
| Manual | User runs `/project:harness-review` | Simplest. Weekly habit |
| Threshold alert | Agent says "5+ candidates accumulated — review recommended" when logging a new candidate | Low-friction nudge |
| Milestone | Sprint end, before release, after onboarding | Teams with regular cadences |

**Recommended combination**: manual + threshold alert. The agent counts candidates when logging, and suggests a review at 5+.

### Phase re-entry triggers

| Trigger | Re-enter phase | Action |
|---------|---------------|--------|
| Same correction 2+ times | Phase 1 (CLAUDE.md) | Add to Gotchas or Workflow |
| Same manual workflow 3+ times | Phase 4 (Hooks) or Phase 3 (Skills) | Automate it |
| New dependency or tool added | Phase 1 (Tech Stack) + Phase 5 (MCP) | Update stack, evaluate MCP need |
| Codebase grows past 3 domains | Phase 6 (Code Structure) | Create structure guide |
| CLAUDE.md exceeds 150 lines | Phase 1 (Splitting) | Split into `.claude/rules/` |
| Tasks span multiple sessions | Phase 7 (Working Memory) | Set up memory documents |
| Same Skill/Agent in 3+ projects | Sharing (Plugins) | Package as a plugin |

### Health check command

Create `.claude/commands/harness-health-check.md` during Phase 1 setup (alongside the review command):

```markdown
Quarterly harness hygiene — remove stale rules and update outdated settings.

1. Read all harness files: CLAUDE.md, .claude/rules/, .claude/skills/, .claude/agents/, .claude/settings.local.json
2. For each rule/setting, evaluate:
   - Would removing this line cause the agent to make mistakes? If not, it should go
   - Has this rule been triggered in the last 3 months?
   - Is a linter or hook already enforcing this, making the CLAUDE.md entry redundant?
3. Present removal candidates as a numbered list
4. If CLAUDE.md exceeds 150 lines, propose splitting into .claude/rules/
5. If tech stack changes (new dependencies, removed tools) are not reflected, propose updates
6. Wait for user approval, then apply
```

### Harness hygiene

Adding rules is easy. Removing them is what keeps the harness effective.

- **When adding**: record the evidence (occurrence count, dates) in the commit message or as a comment
- **Periodic cleanup**: quarterly, run `/project:harness-health-check`. Remove rules that haven't been relevant for 3+ months
- **Deletion criterion**: if removing a line would NOT cause the agent to make mistakes, that line should not exist

---

## Sharing Harness Across Projects

When your harness stabilizes and you find yourself copying the same Skills, Subagents, or Hooks to multiple projects, package them as a **Plugin** for reuse.

A Plugin bundles Skills, Agents, Hooks, and MCP configurations into a single distributable package:

```
my-team-plugin/
├── .claude-plugin/
│   └── plugin.json       # Plugin manifest
├── skills/
│   └── my-skill/
│       └── SKILL.md
├── agents/
│   └── reviewer.md
├── hooks/
│   └── hooks.json
└── .mcp.json             # Optional MCP server definitions
```

Install with `/plugin` in Claude Code, or load locally during development with `--plugin-dir ./my-team-plugin`.

**When to create a plugin:**
- The same Skill/Agent is copied across 3+ projects
- A team needs consistent tooling without per-project setup
- You want to version-control your harness separately from project code

**When NOT to bother:**
- Your harness is still evolving rapidly — wait for it to stabilize
- Each project needs significantly different configuration
- You're a solo developer with 1-2 projects

---

## Reference

### Harness Engineering — Core Concepts

**Harness** = the complete infrastructure surrounding an AI agent: context files, tool connections, skills, hooks, subagents, architectural constraints, and verification systems. The model is the engine; the harness is everything else.

**Key insight**: The same model produces dramatically different results depending on its harness. When the agent struggles, the problem is almost always the harness, not the model. Treat failures as signals — identify what is missing (context, tools, guardrails, documentation) and feed it back.

**Guiding principles:**
- **Context is the fundamental constraint.** Curate ruthlessly what enters the context window
- **Deterministic guardrails complement probabilistic agents.** Use linters, tests, and hooks for what can be mechanically verified
- **Documentation is executable infrastructure.** CLAUDE.md and SKILL.md are not just for humans — agents consume them to orient themselves
- **Build to delete.** As models improve, harness components become obsolete. Design for easy removal
- **Verification closes the loop.** Tests, linter output, screenshots — give the agent the ability to check its own work

### Sources

- [Anthropic: Claude Code Best Practices](https://code.claude.com/docs/en/best-practices)
- [Anthropic: CLAUDE.md Documentation](https://docs.anthropic.com/en/docs/claude-code/claude-md)
- [Anthropic: Hooks Reference](https://code.claude.com/docs/en/hooks)
- [Anthropic: Skills Documentation](https://code.claude.com/docs/en/skills)
- [Anthropic: Create Subagents](https://code.claude.com/docs/en/sub-agents)
- [Anthropic: Agent Teams](https://code.claude.com/docs/en/agent-teams)
- [Anthropic: Create Plugins](https://code.claude.com/docs/en/plugins)
- [Anthropic: Common Workflows (Worktrees)](https://code.claude.com/docs/en/common-workflows)
- [OpenAI: Harness Engineering](https://openai.com/index/harness-engineering/)
- [Martin Fowler: Harness Engineering](https://martinfowler.com/articles/exploring-gen-ai/harness-engineering.html)
- [Phil Schmid: The Importance of Agent Harness in 2026](https://www.philschmid.de/agent-harness-2026)
- [HumanLayer: Writing a Good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [Builder.io: How to Write a Good CLAUDE.md](https://www.builder.io/blog/claude-md-guide)
- [Builder.io: 50 Claude Code Tips and Best Practices](https://www.builder.io/blog/claude-code-tips-best-practices)