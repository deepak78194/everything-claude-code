# Tool Invocation Design in ECC

## Overview

ECC's tool invocation design is built around a few core principles: safety through hooks, performance through async execution, and observability through structured logging. This document explains the design decisions behind how ECC integrates with Claude Code's tool system.

---

## Tool Types Available to ECC Agents

Claude Code provides agents with these tool types:

| Tool | Purpose | ECC Usage |
|---|---|---|
| `Read` | Read file contents | Used by all read-only agents (planner, architect, reviewers) |
| `Write` | Create new files | Used by tdd-guide (test scaffolding), doc-updater |
| `Edit` | Modify existing files | Used by implementation agents, refactor-cleaner |
| `MultiEdit` | Batch file modifications | Used for large refactoring operations |
| `Bash` | Execute shell commands | Used by review and build agents for running tests/linters |
| `Grep` | Search file contents | Used by all search-heavy agents |
| `Glob` | Find files by pattern | Used by agents needing to discover file sets |

### Tool Assignment by Agent Role

ECC assigns tools according to the **principle of least privilege**:

| Agent | Read | Write | Edit | Bash | Grep | Glob |
|---|---|---|---|---|---|---|
| `planner` | ✓ | | | | ✓ | ✓ |
| `architect` | ✓ | | | | ✓ | ✓ |
| `code-reviewer` | ✓ | | | ✓ | ✓ | ✓ |
| `tdd-guide` | ✓ | ✓ | ✓ | ✓ | ✓ | |
| `security-reviewer` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `build-error-resolver` | ✓ | | ✓ | ✓ | ✓ | |
| `refactor-cleaner` | ✓ | | ✓ | ✓ | ✓ | ✓ |

Reasoning-only agents (`planner`, `architect`) get no write or bash access — they should not be modifying code.

---

## Hook Interception Architecture

Every tool invocation is intercepted by ECC's hook system:

```
Agent calls tool
  ↓
Claude Code fires PreToolUse event
  ↓
run-with-flags.js
  ↓ Check ECC_HOOK_PROFILE
  ↓ Check ECC_DISABLED_HOOKS
  ↓ If enabled → execute hook script
  ↓
Tool executes
  ↓
Claude Code fires PostToolUse event
  ↓
run-with-flags.js
  ↓
PostToolUse hook scripts execute
  ↓
Tool result returned to agent
```

### Synchronous vs. Asynchronous Hooks

ECC carefully classifies hooks by their blocking behavior:

**Synchronous hooks** (block tool execution until complete):
- `suggest-compact.js` — must run before the edit to be useful
- `post-edit-format.js` — must complete before the AI sees the result
- `post-edit-typecheck.js` — must complete before the AI continues

**Asynchronous hooks** (run in background, do not block):
- `observe.sh` — learning capture (10s timeout, non-blocking)
- `session-end.js` — session persistence (10s timeout)
- `evaluate-session.js` — pattern extraction (10s timeout)
- `cost-tracker.js` — metrics recording (10s timeout)
- `quality-gate.js` — comprehensive quality checks (30s timeout)
- `post-bash-build-complete.js` — build analysis (30s timeout)

The `"async": true` flag in `hooks.json` tells Claude Code to run the hook in the background. Timeouts prevent runaway hooks from hanging the session.

---

## Tool Result Injection

Some PostToolUse hooks inject information back into the tool result:

**`post-bash-pr-created.js`** — Detects when `gh pr create` was run. Parses the PR URL from stdout and appends:
```
[ECC] PR created: https://github.com/owner/repo/pull/123
Review: gh pr review 123
```

**`session-start.js`** — Returns a context block that Claude Code prepends to the session context. This is how ECC achieves session memory — by injecting a structured summary at the start of every session.

---

## Bash Tool Safety

ECC implements multiple safety layers around Bash tool use:

### 1. Tmux Reminder (`pre-bash-tmux-reminder.js`)

When the AI is about to run a long-running command (detected via pattern matching on the command string), this hook injects a reminder:

```
[ECC] Long-running command detected. Consider using tmux to avoid timeout:
  tmux new-session -d -s dev "npm run dev"
```

### 2. Git Push Reminder (`pre-bash-git-push-reminder.js`)

Before `git push`, injects:
```
[ECC] About to push. Ensure:
  - All tests pass
  - No console.log statements
  - No hardcoded secrets
```

### 3. Dev Server Auto-Start (`auto-tmux-dev.js`)

Detects dev server start commands and automatically wraps them in a named tmux session to prevent blocking the main Claude Code session.

---

## Quality Gate Design

The `quality-gate.js` hook implements a comprehensive post-edit quality check:

```javascript
// Triggered on: Edit | Write | MultiEdit
// Profile: standard, strict
// Async: yes (30s timeout)

async function qualityGate(editedFiles) {
  const results = await Promise.allSettled([
    runLinter(editedFiles),
    runTypeCheck(editedFiles),
    checkTestCoverage(editedFiles)
  ]);
  
  // Summarize and inject findings
  // Only report if quality gate would fail
}
```

The async design means quality gate results arrive after the AI has already responded. The next tool use sees the quality gate output in the hook's return value.

---

## MCP Tool Integration

MCP servers extend Claude Code's tool set. ECC's `mcp-servers.json` configures 20+ MCP servers, each providing additional tools:

| MCP Server | Tool Examples |
|---|---|
| `github` | `create_pull_request`, `list_issues`, `get_file_contents` |
| `supabase` | `execute_sql`, `list_tables`, `get_schema` |
| `playwright` | `browser_navigate`, `browser_click`, `browser_screenshot` |
| `memory` | `store_memory`, `retrieve_memory`, `search_memories` |
| `exa-web-search` | `search`, `find_similar`, `get_contents` |
| `sequential-thinking` | `analyze`, `plan`, `reason` |

MCP tools are invoked identically to native tools from the agent's perspective. ECC hooks do not currently intercept MCP tool invocations (except via the optional InsAIts security monitor).

---

## Tool Use Observability

Every tool use generates an observation (via `observe.sh`):

```json
{
  "session_id": "abc123",
  "timestamp": "2026-03-16T19:00:00Z",
  "tool": "Edit",
  "file": "src/auth/middleware.ts",
  "action": "insert",
  "lines_changed": 12,
  "context": "tdd-guide: implementing JWT validation"
}
```

Observations feed the continuous learning pipeline, enabling ECC to understand:
- Which tools are used most frequently per task type
- Which agents produce the most Edit/Write operations
- Session cost vs. tool use correlation
- Patterns in what gets edited together (file co-change analysis)
