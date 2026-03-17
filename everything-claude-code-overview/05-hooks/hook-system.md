# Hook System — Everything Claude Code

## Overview

The ECC hook system is the **lifecycle automation layer** of the framework. Hooks intercept specific events in the AI session lifecycle and execute scripts that automate quality checks, session persistence, continuous learning, and developer reminders.

Hooks are defined in `hooks/hooks.json` and executed by Claude Code's hook runtime.

---

## Hook Architecture

### Hook Configuration Format

Each hook entry follows this structure:

```json
{
  "matcher": "Bash|Edit|Write|*",
  "hooks": [
    {
      "type": "command",
      "command": "node \"${CLAUDE_PLUGIN_ROOT}/scripts/hooks/run-with-flags.js\" \"<id>\" \"<script>\" \"<profiles>\"",
      "async": true,      // optional: runs in background
      "timeout": 30       // optional: seconds before kill
    }
  ],
  "description": "Human-readable description"
}
```

### Hook Profile System

Every hook is gated by a **profile** that controls whether it runs:

| Profile | Behavior |
|---|---|
| `minimal` | Runs only essential hooks (session persistence, cost tracking) |
| `standard` | Runs most hooks (format, typecheck, quality gates) |
| `strict` | Runs all hooks including aggressive checks |

Runtime control via environment variables:
- `ECC_HOOK_PROFILE=minimal|standard|strict` — set active profile
- `ECC_DISABLED_HOOKS=hook-id1,hook-id2` — disable specific hooks at runtime

The `run-with-flags.js` script checks these variables before executing any hook script.

---

## Hook Events

### SessionStart

**When triggered:** At the start of a new Claude Code session.

**Hooks registered:**

| Script | Purpose |
|---|---|
| `session-start.js` | Load previous session context (git status, recent changes, package manager, project summary) into the context window |

**Implementation note:** Uses a fallback root-search strategy with `bash -lc` to locate the ECC plugin directory across multiple install locations. This handles cases where `CLAUDE_PLUGIN_ROOT` is not set.

---

### PreToolUse

**When triggered:** Before every tool use (Bash, Edit, Write, etc.).

**Matchers and hooks:**

| Matcher | Script | Profile | Purpose |
|---|---|---|---|
| `Bash` | `auto-tmux-dev.js` | all | Auto-start dev servers in tmux with directory-based session names |
| `Bash` | `pre-bash-tmux-reminder.js` | strict | Remind the AI to use tmux for long-running commands |
| `Bash` | `pre-bash-git-push-reminder.js` | strict | Remind to review changes before `git push` |
| `Write` | `doc-file-warning.js` | standard,strict | Warn about non-standard documentation files (exit 0, warns only) |
| `Edit\|Write` | `suggest-compact.js` | standard,strict | Suggest manual context compaction at logical intervals |
| `*` | `observe.sh` (async) | standard,strict | Capture tool use for continuous learning (non-blocking, 10s timeout) |
| `Bash\|Write\|Edit\|MultiEdit` | `insaits-security-wrapper.js` | standard,strict | Optional AI-to-AI security monitoring (requires `ECC_ENABLE_INSAITS=1`) |

---

### PostToolUse

**When triggered:** After every tool use completes.

**Matchers and hooks:**

| Matcher | Script | Profile | Async | Purpose |
|---|---|---|---|---|
| `Bash` | `post-bash-pr-created.js` | standard,strict | no | Log PR URL after `gh pr create`, provide review command |
| `Bash` | `post-bash-build-complete.js` | standard,strict | yes (30s) | Example async hook for background build analysis |
| `Edit\|Write\|MultiEdit` | `quality-gate.js` | standard,strict | yes (30s) | Run quality gate checks (lint, type check) after file edits |
| `Edit` | `post-edit-format.js` | standard,strict | no | Auto-format JS/TS files (auto-detects Biome or Prettier) |
| `Edit` | `post-edit-typecheck.js` | standard,strict | no | TypeScript check after editing `.ts`/`.tsx` files |
| `Edit` | `post-edit-console-warn.js` | standard,strict | no | Warn if `console.log` statements added |
| `*` | `observe.sh` (async) | standard,strict | yes (10s) | Capture tool use results for continuous learning |

---

### PreCompact

**When triggered:** Before Claude Code's context compaction runs.

**Hooks registered:**

| Script | Profile | Purpose |
|---|---|---|
| `pre-compact.js` | standard,strict | Save full session state to the SQLite state store before context is compressed |

**Why this matters:** Context compaction discards detailed session history. This hook ensures important state (git changes, decisions, progress) is persisted before compaction.

---

### Stop

**When triggered:** After each AI response completes (the "Stop" event carries a `transcript_path`).

**Hooks registered:**

| Script | Profile | Async | Purpose |
|---|---|---|---|
| `check-console-log.js` | standard,strict | no | Scan all modified files for `console.log` statements |
| `session-end.js` | minimal,standard,strict | yes (10s) | Persist session transcript and metadata to state store |
| `evaluate-session.js` | minimal,standard,strict | yes (10s) | Evaluate session transcript for extractable patterns/instincts |
| `cost-tracker.js` | minimal,standard,strict | yes (10s) | Record token usage and cost metrics per session |

---

### SessionEnd

**When triggered:** When the Claude Code session fully terminates.

**Hooks registered:**

| Script | Profile | Async | Purpose |
|---|---|---|---|
| `session-end-marker.js` | minimal,standard,strict | yes (10s) | Write session end marker (non-blocking) for lifecycle tracking |

---

## Hook Script Implementation Details

### `run-with-flags.js`

The central hook dispatcher. Before running any hook script, it:
1. Reads `ECC_HOOK_PROFILE` and `ECC_DISABLED_HOOKS` environment variables
2. Checks if the hook's required profiles match the active profile
3. Checks if the hook ID is in the disabled list
4. If enabled, executes the target script via `require()`

### `session-start.js`

1. Reads the last session summary from the SQLite state store
2. Reads current git status (`git status --short`)
3. Detects the project's package manager
4. Injects a formatted context block into the session

### `suggest-compact.js`

Tracks edit/write operations since last compaction. When a threshold is crossed (configurable), injects a suggestion into the AI's output: "Consider running `/checkpoint` to compact context before continuing."

### `evaluate-session.js`

Reads the session transcript (provided via `transcript_path` in the Stop event). Uses heuristics to identify whether the session produced patterns worth capturing as instincts. If so, saves the pattern to the state store.

### `cost-tracker.js`

Parses token usage from the session transcript and stores cumulative cost metrics in the state store. Enables the `/sessions` command to show cost breakdowns.

---

## Cursor Hook Adapter

The `.cursor/hooks/` directory contains JavaScript adapters that map Cursor IDE's hook events to ECC's hook scripts:

| Cursor Event | ECC Equivalent | Script |
|---|---|---|
| `session-start` | `SessionStart` | `session-start.js` |
| `session-end` | `SessionEnd` | `session-end.js` |
| `before-shell-execution` | `PreToolUse[Bash]` | `before-shell-execution.js` |
| `after-file-edit` | `PostToolUse[Edit]` | `after-file-edit.js` |
| `before-mcp-execution` | `PreToolUse[MCP]` | `before-mcp-execution.js` |
| `pre-compact` | `PreCompact` | `pre-compact.js` |
| `stop` | `Stop` | `stop.js` |

The `adapter.js` file provides the shared bridge between Cursor's hook API and ECC's script conventions.

---

## Security Monitoring Hook (Optional)

The `insaits-security-wrapper.js` hook integrates with **InsAIts** (AI-to-AI security monitor):
- Enabled via `ECC_ENABLE_INSAITS=1`
- Requires `pip install insa-its`
- Monitors for 23 anomaly types including credential exposure, hallucination, and prompt injection
- Covers OWASP MCP Top 10
- Runs locally — no data sent externally
