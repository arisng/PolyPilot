# Squad Management — Steering Agent Teams From Your Phone

> The whole point: orchestrate multi-agent AI squads from anywhere.

## What Are Squads?

A **squad** is a team of AI sessions that work together. Each session uses a different model (or the same model with a different persona). An **orchestrator** coordinates work dispatch, response collection, and quality evaluation.

Squads are defined in `.squad/` directories in your repo's worktree root:

```
.squad/
├── team.md           # Team roster and metadata
├── routing.md        # Orchestrator routing instructions (optional)
├── decisions.md      # Shared context for all workers (optional)
└── agents/
    ├── planner/
    │   └── charter.md    # System prompt for the planner agent
    ├── coder/
    │   └── charter.md    # System prompt for the coder agent
    └── reviewer/
        └── charter.md    # System prompt for the reviewer agent
```

## Orchestration Modes

| Mode | How It Works | Best For |
|------|-------------|----------|
| **Broadcast** | Same prompt to all agents simultaneously | Comparing approaches |
| **Sequential** | One agent at a time, each sees previous output | Chain-of-thought |
| **Orchestrator** | One agent plans + delegates to workers | Task-decomposed work |
| **OrchestratorReflect** | Orchestrator + iterative refinement loop | Serious multi-agent work |

## Creating a Squad

### From Your Desktop

1. **Preset Picker:** Dashboard → click the preset selector → choose from:
   - **📂 From Repo** — squads discovered from `.squad/` in your worktree
   - **⚙️ Built-in** — pre-configured team templates
   - **👤 My Presets** — personally saved configurations

2. **Manual Group:** Dashboard → New Group → add sessions → set orchestration mode

### From Your Phone (Remote)

Everything is synced via the bridge. On your phone you can:

1. **View all squads**: Dashboard shows groups with their orchestration mode
2. **Send prompts to squads**: Type in the group input bar → it routes through the orchestrator
3. **Monitor individual agents**: Tap any session card to see its live chat
4. **See processing status**: Elapsed time + tool round count shown per session

### Squad Discovery

PolyPilot discovers squad definitions from repos:

1. You register a repo/worktree in Settings → Repos
2. PolyPilot scans for `.squad/` (or legacy `.ai-team/`) directories
3. Discovered teams appear under **"📂 From Repo"** in the preset picker
4. Each agent's `charter.md` becomes its system prompt
5. `routing.md` is injected into the orchestrator's planning prompt
6. `decisions.md` provides shared context to all workers

### Squad Write-Back

When you modify a squad and save:

1. PolyPilot writes back to `.squad/` format in the worktree via `SquadWriter`
2. Creates `team.md`, `agents/{name}/charter.md`, and optional files
3. Also saves to `~/.polypilot/presets.json` as a personal backup
4. **Round-trip:** discover → modify → save → share via git

## Steering Squads From Your Phone

### The Workflow

```
┌──────────────┐  prompt   ┌──────────────┐  dispatch   ┌──────────────┐
│    Phone     │──────────►│ Orchestrator │────────────►│   Workers    │
│  (any loc.)  │           │  (desktop)   │             │  (desktop)   │
│              │◄──────────│              │◄────────────│              │
│  live stream │  results  │  synthesize  │  responses  │  execute     │
└──────────────┘           └──────────────┘             └──────────────┘
```

1. **Open the squad** on your phone — see all agents in the group view
2. **Type your prompt** in the orchestrator's input (for Orchestrator/Reflect modes) or the group broadcast bar (for Broadcast/Sequential modes)
3. **Watch agents work** — each agent's card shows live streaming output
4. **Read the synthesis** — the orchestrator collects results and produces a final answer
5. **Iterate** — provide feedback, ask follow-ups, steer direction

### Live Monitoring

All bridge events are streamed to your phone in real-time:

| Event | What You See |
|-------|-------------|
| Content delta | Text appearing character-by-character |
| Tool started | "🔧 Running tool X..." indicator |
| Tool completed | Tool result summary |
| Turn start/end | Processing status changes |
| Reasoning delta | Thinking/reasoning stream |
| Intent changed | Plan update notification |
| Usage info | Token counts per session |

### Per-Session Interaction

Tap any agent's session card to:
- See its full chat history
- Send it a direct prompt (bypassing the orchestrator)
- View its tool calls and results
- Check its model and token usage

## Orchestrator Reflect Mode (The Power Mode)

OrchestratorReflect iterates until goals are met:

1. **Plan** → Orchestrator receives your prompt
2. **Dispatch** → Orchestrator assigns `@worker:name task` to workers
3. **Execute** → Workers run in parallel (10-min timeout each)
4. **Evaluate** → Orchestrator reviews results, checks quality
5. **Iterate** → If not satisfied, dispatches more work
6. **Synthesize** → Final answer when quality gate passes or max iterations hit

The iteration count and status are visible on your phone's processing indicator. Each worker has a 10-minute processing watchdog timeout (the general tool-execution timeout).

## Example: Steering a Code Review Squad

```
You (phone): "Review the auth module for security issues. 
              Focus on OWASP Top 10 vulnerabilities."

Orchestrator: Plans → dispatches to:
  @security-auditor: "Scan auth module for injection, broken auth..."
  @code-reviewer: "Review code quality, error handling..."  
  @test-writer: "Check test coverage for auth edge cases..."

Workers: Execute in parallel on your desktop

Orchestrator: Collects results → synthesizes:
  "Found 3 security issues: 1) SQL injection in login.cs:45..."

You (phone): "Fix issue #1 and add a test"

Orchestrator: Dispatches fix + test tasks...
```

All of this happens with your PC doing the heavy lifting. Your phone is just the steering wheel.

> **Security note:** Anyone who can connect to your bridge can send prompts and steer your agents. See [Security Hardening](security.md) to protect your setup.

---

**Next:** [Security Hardening](security.md) →
