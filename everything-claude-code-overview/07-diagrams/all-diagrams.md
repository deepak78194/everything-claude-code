# ECC Diagrams

This directory contains Mermaid diagrams for all major aspects of the Everything Claude Code system.

All diagrams use Mermaid syntax and render natively in GitHub Markdown.

---

## 1. System Architecture

```mermaid
graph TB
    subgraph Harness["AI Harness (Claude Code / Cursor / OpenCode / Codex)"]
        User["👤 User Request"]
        AI["🤖 AI Engine"]
        
        subgraph ECC["ECC Plugin Components"]
            Agents["🤖 Agents\n(planner, tdd-guide,\ncode-reviewer, architect\n+ 14 more)"]
            Skills["📚 Skills\n(tdd-workflow, security-review\napi-design, e2e-testing\n+ 80 more)"]
            Commands["⚡ Commands\n(/tdd /plan /code-review\n/e2e /build-fix /security\n+ 40 more)"]
            Rules["📋 Rules\n(security, testing, patterns\ngit-workflow, coding-style\n× 8 languages)"]
        end
        
        subgraph HookRuntime["Hook Runtime (event-driven)"]
            SessionStart["SessionStart\n↓ load context"]
            PreToolUse["PreToolUse\n↓ checks + reminders"]
            PostToolUse["PostToolUse\n↓ format + quality gate"]
            PreCompact["PreCompact\n↓ save state"]
            Stop["Stop\n↓ persist + evaluate + cost"]
        end
    end
    
    subgraph Persistence["Persistence Layer"]
        StateStore["🗄️ State Store\n(SQLite / sql.js)\n- session summaries\n- cost metrics\n- instincts"]
    end
    
    subgraph MCP["MCP Servers (20+)"]
        GitHub["GitHub"]
        Supabase["Supabase"]
        Playwright["Playwright"]
        Exa["Exa Search"]
        Memory["Memory"]
        More["+ 15 more"]
    end
    
    User -->|"types request"| Commands
    User -->|"types request"| AI
    Commands -->|"activates"| Agents
    Commands -->|"injects"| Skills
    Rules -->|"always applied"| AI
    AI -->|"delegates to"| Agents
    Agents -->|"uses"| Skills
    
    SessionStart -->|"reads"| StateStore
    SessionStart -->|"enriches"| AI
    Stop -->|"writes"| StateStore
    PreCompact -->|"saves"| StateStore
    
    AI -->|"calls tools"| MCP
    MCP -->|"returns data"| AI
```

---

## 2. Agent Interaction Sequence (TDD Workflow)

```mermaid
sequenceDiagram
    participant U as User
    participant CC as Claude Code (AI)
    participant P as planner
    participant T as tdd-guide
    participant R as code-reviewer
    participant S as security-reviewer
    participant FS as File System / Tools

    U->>CC: /tdd add user authentication
    CC->>P: Activate planner agent
    P->>FS: Read codebase structure
    FS-->>P: File tree + existing patterns
    P-->>CC: Implementation plan (phases, steps, risks)
    
    CC->>T: Activate tdd-guide agent
    T->>FS: Read existing auth code
    T->>FS: Write auth.test.ts (failing tests)
    T->>FS: Run: npm test (RED phase)
    FS-->>T: Tests fail ✓ (expected)
    
    T->>FS: Write auth.ts (implementation)
    Note over FS: PostToolUse hooks:<br/>auto-format, typecheck,<br/>quality-gate
    T->>FS: Run: npm test (GREEN phase)
    FS-->>T: Tests pass ✓
    
    T->>FS: Edit auth.ts (refactor)
    T->>FS: Run: npm test (REFACTOR phase)
    FS-->>T: Tests pass ✓
    T->>FS: Run: npm run coverage
    FS-->>T: Coverage: 92% ✓
    T-->>CC: TDD complete
    
    CC->>R: Activate code-reviewer
    R->>FS: git diff --staged
    FS-->>R: Changes
    R-->>CC: Review report (HIGH: missing rate limiting)
    
    CC->>S: Activate security-reviewer (auth code)
    S->>FS: Scan for hardcoded secrets, OWASP issues
    FS-->>S: Scan results
    S-->>CC: Security report (PASS)
    
    CC-->>U: Implementation complete + review summary
```

---

## 3. Hook Execution Pipeline

```mermaid
flowchart TD
    A["Tool Use Requested\n(Bash / Edit / Write / Read)"] --> B["run-with-flags.js"]
    B --> C{ECC_HOOK_PROFILE\ncheck?}
    C -->|"disabled"| Skip["Skip hook"]
    C -->|"enabled"| D{ECC_DISABLED_HOOKS\ncheck?}
    D -->|"hook in list"| Skip
    D -->|"not disabled"| E["Execute hook script"]
    
    subgraph PreToolUse["PreToolUse Hooks (before tool)"]
        F1["auto-tmux-dev.js\n(Bash only)"]
        F2["suggest-compact.js\n(Edit/Write)"]
        F3["pre-bash-tmux-reminder.js\n(Bash, strict)"]
        F4["observe.sh async\n(all tools)"]
        F5["insaits-security-wrapper.js\n(optional)"]
    end
    
    subgraph Tool["Tool Executes"]
        G["Bash / Edit / Write / Read\nactual execution"]
    end
    
    subgraph PostToolUse["PostToolUse Hooks (after tool)"]
        H1["post-edit-format.js\n(Edit only)"]
        H2["post-edit-typecheck.js\n(Edit .ts/.tsx)"]
        H3["post-edit-console-warn.js\n(Edit)"]
        H4["quality-gate.js async\n(Edit/Write/MultiEdit)"]
        H5["post-bash-pr-created.js\n(Bash)"]
        H6["observe.sh async\n(all tools)"]
    end
    
    subgraph StopHooks["Stop Hooks (after AI response)"]
        I1["check-console-log.js"]
        I2["session-end.js async"]
        I3["evaluate-session.js async"]
        I4["cost-tracker.js async"]
    end
    
    E --> PreToolUse
    PreToolUse --> Tool
    Tool --> PostToolUse
    PostToolUse --> StopHooks
```

---

## 4. Continuous Learning Pipeline

```mermaid
flowchart LR
    subgraph Session["During Session"]
        O1["PreToolUse\nobserve.sh"] --> OStore["Observation Store\n(filesystem)"]
        O2["PostToolUse\nobserve.sh"] --> OStore
    end
    
    subgraph Stop["At Stop Event"]
        Eval["evaluate-session.js\n(reads transcript_path)"] --> Worthy{Patterns\nextractable?}
        Worthy -->|"yes"| Extract["Extract instincts\n(patterns, learnings)"]
        Worthy -->|"no"| Discard["Discard"]
        Extract --> StateDB["SQLite State Store\n(instincts table)"]
    end
    
    subgraph NextSession["Next SessionStart"]
        Load["session-start.js\nloads instincts"] --> Inject["Inject into\ncontext window"]
        StateDB -->|"reads"| Load
    end
    
    subgraph Manual["Manual Evolution"]
        Learn["/learn command"] --> SkillCreate["/skill-create\ncommand"]
        SkillCreate --> NewSkill["New SKILL.md\n(added to skills/)"]
        Evolve["/evolve command"] --> UpdateSkill["Updated SKILL.md"]
    end
    
    OStore -->|"feeds"| Eval
```

---

## 5. Component Dependency Graph

```mermaid
graph LR
    subgraph Core["Core ECC"]
        hooks["hooks/hooks.json"]
        agents["agents/*.md"]
        skills["skills/*/SKILL.md"]
        commands["commands/*.md"]
        rules["rules/**/*.md"]
    end
    
    subgraph Runtime["Runtime (scripts/)"]
        ecc["ecc.js\n(CLI)"]
        install_plan["install-plan.js"]
        install_apply["install-apply.js"]
        hook_scripts["scripts/hooks/*.js"]
        state_store["lib/state-store/"]
        session_mgr["lib/session-manager.js"]
        pkg_mgr["lib/package-manager.js"]
        skill_evo["lib/skill-evolution/"]
    end
    
    subgraph External["External"]
        sqljs["sql.js\n(SQLite)"]
        mcp["MCP Servers"]
        claude_code["Claude Code\nHook Runtime"]
    end
    
    hooks -->|"invokes"| hook_scripts
    hook_scripts -->|"uses"| state_store
    hook_scripts -->|"uses"| session_mgr
    hook_scripts -->|"uses"| pkg_mgr
    state_store -->|"uses"| sqljs
    
    ecc -->|"calls"| install_plan
    install_plan -->|"calls"| install_apply
    
    commands -->|"activate"| agents
    commands -->|"inject"| skills
    rules -->|"always loaded"| claude_code
    
    claude_code -->|"fires events"| hooks
    claude_code -->|"connects to"| mcp
    claude_code -->|"reads"| agents
    claude_code -->|"reads"| skills
```

---

## 6. Install System Flow

```mermaid
flowchart TD
    A["User runs:\nnpx ecc typescript"] --> B["ecc.js\n(CLI entry point)"]
    B --> C["install-plan.js\n(resolve components)"]
    C --> D["Read manifests/typescript.json\n(component list)"]
    D --> E["Resolve file paths\nfor each component"]
    E --> F["install-apply.js\n(copy files)"]
    F --> G{"Target directory\nexists?"}
    G -->|"no"| H["Create directory"]
    G -->|"yes"| I["Check conflicts"]
    H --> J["Copy files"]
    I --> J
    J --> K["Update .claude.json\n(hook registration)"]
    K --> L["Write install state\n(installed-state.json)"]
    L --> M["Print success message\nwith next steps"]
```

---

## 7. Cross-Harness Architecture

```mermaid
graph TB
    subgraph Source["ECC Source Repository"]
        core["Core definitions\n(agents/, skills/, hooks/, rules/)"]
    end
    
    subgraph ClaudeCode["Claude Code"]
        cc_agents["agents/\n(Markdown + YAML)"]
        cc_hooks["hooks/hooks.json\n(lifecycle)"]
        cc_skills["skills/\n(SKILL.md)"]
        cc_commands["commands/\n(slash cmds)"]
        cc_rules["rules/\n(always-follow)"]
    end
    
    subgraph Cursor["Cursor IDE"]
        cur_hooks[".cursor/hooks/\n(JS adapters)"]
        cur_rules[".cursor/rules/\n(Markdown)"]
        cur_skills[".cursor/skills/\n(SKILL.md)"]
    end
    
    subgraph OpenCode["OpenCode"]
        oc_prompts[".opencode/prompts/\n(agent prompts .txt)"]
        oc_commands[".opencode/commands/\n(Markdown)"]
        oc_tools[".opencode/tools/\n(TypeScript)"]
    end
    
    subgraph Codex["Codex (OpenAI)"]
        codex_agents[".codex/agents/\n(TOML format)"]
        codex_agents_md["AGENTS.md\n(top-level)"]
    end
    
    core -->|"maps to"| ClaudeCode
    core -->|"adapted to"| Cursor
    core -->|"adapted to"| OpenCode
    core -->|"adapted to"| Codex
```
