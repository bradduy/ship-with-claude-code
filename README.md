# Claude Code — Agent Infrastructure

This is the monorepo for **Claude Code**, a terminal-based AI coding assistant built in TypeScript/React. It provides a full-featured AI agent runtime: a streaming query engine, a library of 40+ composable tools, a plugin and skill system, MCP (Model Context Protocol) integration, and a custom terminal UI renderer.

---

## What Claude Code Is

Claude Code runs as an interactive REPL in a terminal. You type a request in plain language, and the agent:

1. Builds a **context** (git status, project structure, memory files, system capabilities)
2. Calls the **Claude API** with that context streamed back
3. **Executes tools** in sequence or parallel (file reads, bash commands, web search, sub-agent spawns, etc.)
4. **Yields a response** — text, diffs, summaries, or tool results — rendered in the terminal

The agent is **fully programmatic**: it can be embedded in other tools, controlled via the Bridge API, extended with plugins and MCP servers, and invoked via the SDK from other Node.js processes.

---

## Repository Structure

```
claude-code/
├── main.tsx              # Primary REPL entry point (compiled binary target)
├── setup.ts              # Startup orchestrator (session, git root, memory, hooks)
├── QueryEngine.ts        # Core class: owns a conversation turn, budget, API calls
├── query.ts              # Query loop: streams Claude API, orchestrates tools
├── query/                # Query engine internals (transitions, config, deps)
├── Tool.ts               # Tool interface contract (call, render, permissions, schema)
├── tools.ts              # Tool registry (assembles all tools, dead-code elimination)
├── tools/                # 40+ individual tool implementations
├── Task.ts               # Task types (LocalAgentTask, RemoteAgentTask)
├── tasks/                # Task system (spawn sub-agents, track progress)
├── commands.ts           # Slash command registry (~70 commands)
├── commands/             # One directory per slash command
├── context.ts            # Builds system prompt context (git, files, memory)
├── memdir/               # Memory directory system (.claude/, CLAUDE.md files)
├── history.ts            # Session history and paste tracking
├── cost-tracker.ts       # Token usage and cost tracking
├── hooks/                # 60+ React hooks for UI state
├── components/            # React UI components (REPL, desktop, settings)
├── ink/                  # Custom terminal UI renderer (React + Yoga + Ink reconciler)
├── ink.ts                # Ink renderer entry point
├── state/                # Zustand global state (AppState, store)
├── bootstrap/            # App initialization, session state, startup profiler
├── services/             # Backend services (MCP, API, compact, analytics, voice)
├── bridge/               # IDE Bridge: headless CLI controlled by desktop/web clients
├── coordinator/          # Coordinator mode: multi-agent team orchestration
├── plugins/               # Plugin loader, built-in plugins, bundled skills
├── skills/                # Skill system (bundled skills, skill discovery)
├── server/               # Internal HTTP server (diagnostics, metrics)
├── screens/              # Screen management (IDE screenshots, teleportation)
├── schemas/               # Zod schemas (hooks, plugin manifests)
├── types/                 # Shared TypeScript type definitions
├── native-ts/            # Native bindings (yoga-layout, file-index)
├── utils/                 # 200+ utility modules
├── entrypoints/           # SDK/MCP/CLI entry points
├── voice/                 # Voice mode (STT, wake word, keyterms)
├── migrations/             # Session/data migrations
├── remote/                # Remote session management
├── more/                  # "More right" panel (expanded context view)
└── 20+ root-level .ts     # Top-level utilities and state
```

---

## How the AI Agent Works

### The Query Loop

```
User input
    ↓
processUserInput()         # Slash commands, model selection, allowed tools
    ↓
fetchSystemPromptParts()   # Build context: git status, CLAUDE.md, memory files
    ↓
query()                    # Stream from Claude API
    ↓
Tool execution loop        # runTools() → Tool.call()
    ├→ File system (Bash, Read, Edit, Write, Glob, Grep)
    ├→ Web (WebSearch, WebFetch)
    ├→ Tasks (TaskCreate, TaskStop, Agent)
    ├→ Skills (SkillTool)
    ├→ MCP servers (MCPTool)
    └→ User interaction (AskUserQuestion)
    ↓
Tool result → API stream  # Continue loop with results
    ↓
Yield result               # Text, diffs, summaries → UI render
    ↓
Post-processing            # Hooks (PreToolUse, PostToolUse), analytics, compact
```

### Context Building (`context.ts`)

Every query builds a **system prompt** from multiple sources:

- **Git context**: branch, status, diff summary, worktree configuration
- **Project structure**: file tree, key config files
- **Memory files**: `CLAUDE.md`, `CLAUDE_DEVELOPER.md`, `.claude/` directory files (recursive)
- **Tool descriptions**: all available tools with dynamic descriptions
- **Slash commands**: all registered commands with descriptions
- **Custom instructions**: loaded from `CLAUDE.md` frontmatter hooks

### Budget & Compaction

The query engine tracks **token budgets per turn**. When a budget limit is approached:

1. **Compact** — summarises recent conversation history to free context space
2. **Snip** — removes middle turns of long sessions (HISTORY_SNIP feature)
3. **Reactive compact** — event-driven compaction triggered by hook (`PostCompact`)

---

## Major Subsystems

### Tools (`tools/`, `Tool.ts`, `tools.ts`)

Tools are the **actions** the agent can take. The `Tool` interface defines ~30 methods covering:

| Aspect | Methods |
|---|---|
| **Execution** | `call(args, context, canUseTool)` → `ToolResult` |
| **Schema** | `inputSchema` (Zod), `inputJSONSchema` (raw JSON, for MCP) |
| **Permissions** | `checkPermissions`, `validateInput`, `preparePermissionMatcher` |
| **Safety** | `isConcurrencySafe`, `isReadOnly`, `isDestructive`, `isOpenWorld` |
| **UI rendering** | `renderToolUseMessage`, `renderToolResultMessage`, `renderToolUseProgressMessage`, `renderToolUseErrorMessage`, `renderToolUseRejectedMessage` |
| **Summarisation** | `getToolUseSummary`, `getActivityDescription`, `extractSearchText` |
| **Classification** | `toAutoClassifierInput` (for auto-mode security) |
| **Interruption** | `interruptBehavior()` → `'cancel' \| 'block'` |

Key tools:

| Tool | File | Purpose |
|---|---|---|
| `Bash` | `BashTool/` | Execute shell commands |
| `Read` | `FileReadTool/` | Read file contents |
| `Edit` | `FileEditTool/` | Edit files (diff-based) |
| `Write` | `FileWriteTool/` | Write/overwrite files |
| `Glob` | `GlobTool/` | Glob pattern file search |
| `Grep` | `GrepTool/` | Search file contents |
| `Agent` | `AgentTool/` | Spawn sub-agents (LocalAgentTask / RemoteAgentTask) |
| `Skill` | `SkillTool/` | Invoke skills |
| `MCP` | `MCPTool/` | Call tools on MCP servers |
| `EnterPlanMode` | `EnterPlanModeTool/` | Enter structured planning mode |
| `ExitPlanMode` | `ExitPlanModeV2Tool/` | Exit planning mode |
| `AskUserQuestion` | `AskUserQuestionTool/` | Ask user a question with choices |
| `TaskCreate` | `TaskCreateTool/` | Create subtasks |
| `EnterWorktree` | `EnterWorktreeTool/` | Create git worktree |
| `WebSearch` | `WebSearchTool/` | Web search |
| `WebFetch` | `WebFetchTool/` | Fetch web page content |
| `TodoWrite` | `TodoWriteTool/` | Todo list management |

Tool execution is orchestrated by `services/tools/StreamingToolExecutor.ts` (streaming output) and `services/tools/toolOrchestration.ts` (`runTools()`), which handles parallel tool calls, sequential dependencies, and abort controllers.

### Commands (`commands/`, `commands.ts`)

Slash commands provide **user-facing features** invoked with `/`. There are ~70 commands across categories:

| Category | Examples |
|---|---|
| **File ops** | `/diff`, `/files`, `/copy`, `/rename` |
| **Git** | `/branch`, `/commit`, `/review`, `/security-review` |
| **Session** | `/session`, `/resume`, `/rewind`, `/compact` |
| **Context** | `/context`, `/ctx_viz`, `/brief` |
| **Tools** | `/doctor`, `/mcp`, `/permissions`, `/env`, `/keybindings` |
| **Output** | `/output-style`, `/theme`, `/color` |
| **Info** | `/help`, `/status`, `/cost`, `/stats`, `/usage` |
| **Teams** | `/team`, `/teammate` |
| **Plugins** | `/plugin`, `/reload-plugins`, `/install-github-app` |
| **Config** | `/config`, `/login`, `/logout` |
| **Workflow** | `/workflows` (feature-gated) |
| **Voice** | `/voice` (feature-gated) |

Each command lives in its own directory under `commands/` with an `index.ts` entry point.

### Hooks System

Hooks are the **extensibility mechanism** for intercepting and modifying agent behavior. There are two hook systems:

**Native hooks** (run by the CLI itself):
| Event | Purpose |
|---|---|
| `SessionStart` | Runs on every REPL startup |
| `SessionStop` | Runs when REPL exits |
| `UserPromptSubmit` | Fires before user input is processed |
| `PreToolUse` | Fires before a tool is called |
| `PostToolUse` | Fires after a tool completes |
| `MessageStart` | Fires when AI message begins |
| `MessageStop` | Fires when AI message completes |
| `AfterMessage` | Post-sampling hook |
| `PostCompact` | Fires after compaction |
| `Embedding` | Runs on file embedding |

**Plugin hooks** (defined in plugin manifests):
- `command` hooks — run shell commands conditionally
- `async` hooks — non-blocking callbacks

Hooks are configured in `hooks/hooks.json` or in project `CLAUDE.md` frontmatter. The hook matcher system supports conditional execution with permission-rule syntax (`if: "Bash(git *)"`).

### Plugin & Skill System

**Plugins** (`plugins/`) provide self-contained extensibility:

- **Built-in plugins** (`{name}@builtin`): Ship with CLI (e.g., `git`, `fast`, `diff`)
- **Marketplace plugins**: Installed from plugin repositories
- Plugin manifest declares: skills, hooks, MCP servers

**Skills** (`skills/`) are named capabilities invoked via `/skill` or the `SkillTool`:

- Bundled skills (ship with CLI): pre-defined prompts + tools
- Custom skills: defined in `.claude/skills/` or `~/.claude/skills/`
- MCP-sourced skills: auto-generated from MCP server tool definitions

Skill structure: `name`, `description`, `prompt` (instructions), optional `tools` array, optional `metadata`.

### MCP (Model Context Protocol)

The MCP system (`services/mcp/`) connects Claude Code to external tools:

- **MCP Client**: connects to any MCP server via stdio, HTTP+SSE, or Streamable HTTP
- **MCP Tool**: wraps MCP tools so they appear as native Claude Code tools
- **OAuth support**: for remote MCP server authentication
- **Tool normalization**: converts MCP tool schemas to Claude Code's format

MCP servers are configured in `settings.json` or via `claude mcp add`.

### UI: Ink Renderer

Claude Code renders its UI in the **terminal** using **Ink** — a custom React renderer:

- React components → terminal ANSI output (no browser)
- **Yoga layout engine**: flexbox layout in the terminal
- Custom **React reconciler**: handles incremental updates
- Supports: mouse tracking, hyperlinks, color, selection, scrollback
- Built-in primitives: `Box`, `Text`, `Button`, `Link`, `ScrollBox`, `RawAnsi`

The UI layer (`components/`) builds on top of Ink primitives:
- `Message.tsx` / `MessageRow.tsx` — chat messages
- `ToolUseLoader.tsx` — tool execution progress
- `StructuredDiff.tsx` — code diff display
- `StatusLine.tsx` — status bar
- `ContextVisualization.tsx` — context inspection overlay
- Settings panels, dialogs, task list views

### Bridge System (`bridge/`)

The Bridge enables Claude Code to run as a **headless CLI process** controlled by a desktop app or web client:

```
Desktop IDE / Web Client
         ↓ (WebSocket or HTTP)
Bridge Server (bridgeMain.ts)
         ↓ (stdio, Unix Domain Socket)
Claude Code CLI (headless)
```

Key capabilities:
- Session spawning and lifecycle management
- JWT-based trusted device authentication
- Remote session management (direct connect, webhooks)
- Polling and event streaming between bridge client and CLI

### Coordinator Mode (`coordinator/`)

Feature-flagged multi-agent orchestration:

- One **coordinator** agent manages multiple **worker** agents
- Workers execute in parallel on sub-tasks
- Permission escalation between coordinator and workers
- Team memory synchronization

### State Management

**Global state** (`state/AppState.tsx`, `state/store.ts`):
- Zustand store for React UI state
- Session, model, messages, tasks, feature flags, MCP clients, team state

**Session state** (`bootstrap/state.ts`):
- Non-React singleton state (project root, session ID, cost, budgets)
- Initialized once at startup, never recreated

**History** (`history.ts`):
- Session transcript (last 100 items)
- Paste tracking with `parseReferences()` for `[Pasted text #N]` refs

---

## How This Project Is Used for AI Agents

Claude Code's architecture makes it a **reusable AI agent runtime**. External systems can use it in several ways:

### 1. Embed via the SDK (`entrypoints/sdk/`)

Import and instantiate the agent from a Node.js process:

```typescript
import { init, initializeTelemetryAfterTrust } from './entrypoints/init.js';
import { fetchBootstrapData } from './services/api/bootstrap.js';
// Initialize session, bootstrap data, then:
// new QueryEngine({ sessionId, projectRoot, messages, tools, ... })
```

The SDK exposes the full agent lifecycle: session management, context building, streaming responses, tool execution, and hook firing.

### 2. Bridge API (`bridge/bridgeApi.ts`)

Control a headless Claude Code session from any HTTP client:

- Start/stop sessions
- Submit prompts and stream responses
- Manage MCP connections
- Access session history and context

### 3. MCP Server Mode (`entrypoints/mcp.ts`)

Run Claude Code as an MCP server (`--claude-in-chrome-mcp`):

```bash
claude --claude-in-chrome-mcp
```

This exposes Claude Code's tools (Bash, Read, Edit, etc.) as MCP tools to any MCP client — enabling Claude Code to be used as a backend agent engine from other AI applications.

### 4. Plugin System

Build plugins that extend Claude Code with:

- New **skills** (prompts + tool bundles)
- New **hooks** (pre/post tool use, session lifecycle)
- New **MCP servers** (connect external tools)
- New **slash commands**

Plugins are published to the plugin marketplace or installed locally.

### 5. Sub-agents (`AgentTool`, `TaskCreate`)

Claude Code can spawn **sub-agents** to handle parallel work:

- `TaskCreate` creates independent tasks
- `AgentTool` spawns a sub-agent (LocalAgentTask or RemoteAgentTask)
- Sub-agents have their own context, tools, and budget
- Coordinator mode coordinates teams of agents

---

## Feature Flags

Claude Code uses a **dead-code elimination** system (`feature()` from `bun:bundle`) to ship different builds. Key flags:

| Flag | Feature |
|---|---|
| `COORDINATOR_MODE` | Multi-agent team orchestration |
| `HISTORY_SNIP` | History snipping for long headless sessions |
| `REACTIVE_COMPACT` | Reactive compaction on `PostCompact` hook |
| `UDS_INBOX` | Unix Domain Socket for IPC |
| `VOICE_MODE` | Voice input mode |
| `WORKFLOW_SCRIPTS` | Workflow automation scripts |
| `CCR_REMOTE_SETUP` | Remote setup command |
| `AGENT_TRIGGERS` | Cron scheduling tools |
| `MONITOR_TOOL` | MCP monitor tool |
| `DUMP_SYSTEM_PROMPT` | Dump system prompt for evals |

---

## Skills System

Skills are **named capabilities** the agent can invoke — they look like slash commands (`/simplify`, `/brainstorming`) but are powered by structured prompts with full tool access. Skills sit between a freeform chat and a built-in command: the model decides when to invoke one based on the conversation, and the skill's prompt steers the agent through a multi-step process.

### Three Skill Sources

| Source | Location | Registration | Invocable by |
|---|---|---|---|
| **Bundled** | `skills/bundled/*.ts` | `initBundledSkills()` at startup | Everyone |
| **Disk-based** | `.claude/skills/`, `~/.claude/skills/` | `getSkillDirCommands()` lazily | Project/user |
| **MCP-sourced** | MCP servers | `registerMCPSkillBuilders()` | Remote tools |

### How a Skill Is Loaded

```
User types /<skill-name>
    ↓
findCommand() looks up the name in the command registry
    ↓
SkillTool.call(skillName, args, context)
    ↓
getPromptForCommand(args, toolUseContext)
    — loads markdown content
    — substitutes ${CLAUDE_SKILL_DIR}, ${CLAUDE_SESSION_ID}, argument placeholders
    — executes inline shell blocks (!`…`) if loadedFrom !== 'mcp'
    ↓
Returns ContentBlockParam[] → injected into the conversation as a system prompt
    ↓
Model continues with the skill's guidance
```

### Bundled Skills (`skills/bundled/`)

Each bundled skill is a TypeScript file that calls `registerBundledSkill(definition)`. The definition includes:

```typescript
type BundledSkillDefinition = {
  name: string           // slash command name, e.g. "simplify"
  description: string
  aliases?: string[]
  whenToUse?: string     // guidance for when the model should auto-invoke
  argumentHint?: string
  allowedTools?: string[]     // tool allowlist for this skill
  model?: string              // model override for this skill
  disableModelInvocation?: boolean  // prevent model from invoking itself
  userInvocable?: boolean     // allow /<name> by user
  isEnabled?: () => boolean   // feature flag / runtime check
  hooks?: HooksSettings       // register hooks when this skill runs
  context?: 'inline' | 'fork' // inline = expand into current conversation; fork = run as sub-agent
  agent?: string              // sub-agent type when context: 'fork'
  files?: Record<string, string>  // extract reference files to disk on first invoke
  getPromptForCommand(args, context): Promise<ContentBlockParam[]>
}
```

Bundled skills ship compiled into the binary. Reference files (e.g., prompt templates, code samples) are extracted lazily on first invocation to a secure per-skill directory (`~/.claude/.bundled-skills/<name>/`) using `O_EXCL|O_NOFOLLOW` writes to prevent symlink attacks.

**Example** — `skills/bundled/simplify.ts`:
```typescript
export function registerSimplifySkill(): void {
  registerBundledSkill({
    name: 'simplify',
    description: 'Review changed code for reuse, quality, and efficiency, then fix any issues found.',
    async getPromptForCommand(args) {
      const prompt = SIMPLIFY_PROMPT + (args ? `\n\n## Additional Focus\n\n${args}` : '')
      return [{ type: 'text', text: prompt }]
    },
  })
}
```

The prompt guides the model through three parallel sub-agents (reuse, quality, efficiency review), then aggregates and fixes findings.

### Bundled Skill Registry (`skills/bundled/index.ts`)

`initBundledSkills()` registers all bundled skills at startup. Most are always-on; some are feature-gated:

| Skill | Feature Flag | Purpose |
|---|---|---|
| `update-config` | — | Configure Claude Code settings |
| `keybindings` | — | Keybinding reference |
| `verify` | — | Plan verification |
| `debug` | — | Systematic debugging |
| `lorem-ipsum` | — | Placeholder content |
| `skillify` | — | Convert a conversation to a skill |
| `remember` | — | Extract and save memories |
| `simplify` | — | Code review and cleanup |
| `batch` | — | Batch multiple tasks |
| `stuck` | — | Help when blocked |
| `loop` | `AGENT_TRIGGERS` | Recurring task on interval |
| `schedule-remote-agents` | `AGENT_TRIGGERS_REMOTE` | Schedule agents via cron |
| `claude-api` | `BUILDING_CLAUDE_APPS` | Build with Claude API/SDK |
| `claude-in-chrome` | `shouldAutoEnableClaudeInChrome()` | Chrome extension |
| `dream` | `KAIROS` or `KAIROS_DREAM` | Background reflection |
| `hunter` | `REVIEW_ARTIFACT` | Bughunter feedback |
| `run-skill-generator` | `RUN_SKILL_GENERATOR` | Skill scaffolding |

### Disk-Based Skills

Skills on disk live in **directory format**:

```
.claude/skills/
├── simplify/
│   └── SKILL.md        ← skill content + frontmatter
├── brainstorming/
│   └── SKILLILL.md
```

`SKILL.md` frontmatter controls behavior:

```markdown
---
name: simplify
description: Review changed code for reuse, quality, and efficiency, then fix any issues found.
when_to_use: After editing multiple TSX components, before shipping.
allowed_tools: [Read, Edit, Bash, Glob, Grep]
argument_hint: Optional additional focus areas
context: inline    # or "fork" to run as sub-agent
agent: general-purpose
---
# Simplify

Your skill prompt content here...
```

`loadSkillsFromSkillsDir()` walks all three skill directories in parallel:

| Directory | Source | Priority |
|---|---|---|
| Managed (policy) | `getManagedFilePath()/.claude/skills/` | Highest |
| User home | `~/.claude/skills/` | Medium |
| Project | `<cwd>/.claude/skills/` (recursive) | Lowest |

Symlinks are resolved via `realpath()` for deduplication. Dynamic discovery also walks up from file paths touched during the session to find nested `.claude/skills/` directories.

### Conditional Skills (Path-Filtered)

Skills with `paths` frontmatter are **conditional** — they only activate when the model touches a matching file:

```markdown
---
paths:
  - "**/*.tsx"
---
# React Best Practices

Check component patterns on TSX files...
```

`activateConditionalSkillsForPaths()` uses the `ignore` library (gitignore-style) to match file paths against patterns. When activated, the skill moves from `conditionalSkills` to `dynamicSkills` and becomes available to the model.

### Skill Invocation Flow

1. `SkillTool.call(name, args, context)` is called by the model
2. `findCommand(name)` looks up the skill in the registry
3. `cmd.getPromptForCommand(args, context)` loads the prompt
   - Resolves `${CLAUDE_SKILL_DIR}` and `${CLAUDE_SESSION_ID}`
   - Substitutes argument placeholders (`$1`, `$2`, named args)
   - Executes inline shell blocks (`!`...``) with allowed tools scoped
   - Prefixes `Base directory for this skill: <path>` so the model can `Read`/`Grep` reference files
4. Returns `ContentBlockParam[]` injected as a user message
5. Model continues conversation with the skill's guidance baked into context

### SkillTool (`tools/SkillTool/`)

The `SkillTool` is the tool the model uses to discover and invoke skills. Its prompt tells the model:

- Skills are listed in `system-reminder` messages each turn
- Invoke with `skill: "<name>", args: "..."`
- Do NOT invoke a skill already running
- Do NOT invoke for built-in CLI commands (`/help`, `/clear`, etc.)

The prompt includes all skill names + truncated descriptions within a **1% context window budget** (≈8,000 chars). Bundled skills' full descriptions are always preserved; non-bundled descriptions are trimmed to fit.

### Hooks on Skills

Skills can register hooks that fire when the skill runs:

```markdown
---
hooks:
  PreToolUse:
    - matcher: "Bash(git *)"
      if: "always"
      hooks:
        - type: "command"
          command: "echo 'running {tool} with {input}'"
```


| File | Role |
|---|---|
| `main.tsx` | REPL entry point |
| `setup.ts` | Startup orchestrator |
| `QueryEngine.ts` | Core query lifecycle class |
| `query.ts` | API streaming + tool orchestration loop |
| `Tool.ts` | Tool interface definition |
| `tools.ts` | Tool registry |
| `commands.ts` | Slash command registry |
| `context.ts` | System prompt builder |
| `state/AppState.tsx` | React global state (Zustand) |
| `bootstrap/state.ts` | Non-React session state |
| `history.ts` | Session transcript |
| `cost-tracker.ts` | Token usage tracking |
| `ink.tsx` | Terminal UI renderer entry |
| `bridge/bridgeMain.ts` | Bridge server for IDE integration |
| `services/mcp/` | MCP client implementation |
| `plugins/` | Plugin loader and built-in plugins |
| `skills/` | Skill system and bundled skills |
| `utils/hooks/` | Native hook system |
| `types/message.ts` | Core message types |
| `types/permissions.ts` | Permission types and rules |

---

## Workflows (Feature-Gated: `WORKFLOW_SCRIPTS`)

Workflows are **scripted multi-step agent procedures** — defined in YAML or TypeScript, they let teams encode a repeatable development process (e.g. "write spec → implement → review → test") as a first-class agent capability. They bridge the gap between a freeform chat and a structured automation pipeline.

### Architecture

```
Feature flag: WORKFLOW_SCRIPTS
    ↓
WorkflowTool (tools/WorkflowTool/WorkflowTool.ts)     # The Tool the agent calls
    ↓
createWorkflowCommand (tools/WorkflowTool/createWorkflowCommand.ts)  # Loads workflow scripts
    ↓
Bundled + custom workflow scripts  (tools/WorkflowTool/bundled/)   # The actual scripts
    ↓
workflow commands  (kind: 'workflow')   # Registered as slash commands via commands.ts
```

### How Workflows Work

1. **Workflow scripts** live in `.claude/workflows/` (project) or `~/.claude/workflows/` (user)
2. Each script defines a named **workflow** (e.g. `spec`, `review`, `test`) with steps
3. `createWorkflowCommand()` scans those directories and registers each workflow as a `PromptCommand` with `kind: 'workflow'`
4. The agent sees them as slash commands (`/spec`, `/review`) and can invoke them
5. `WorkflowTool` (`WORKFLOW_TOOL_NAME = 'Workflow'`) is the tool the agent calls when executing a workflow step
6. Bundled workflows are initialized via `initBundledWorkflows()` at startup

### Key Integration Points

| File | Role |
|---|---|
| `commands.ts` | Feature-gated import of `createWorkflowCommand`; registers workflow commands in `loadAllCommands()` |
| `tools.ts` | Feature-gated import of `WorkflowTool`; adds it to the tool registry |
| `constants/tools.ts` | `WORKFLOW_TOOL_NAME` constant; `WorkflowTool` added to `ALL_AGENT_DISALLOWED_TOOLS` (prevents recursive workflow execution in sub-agents) |
| `types/command.ts` | `kind?: 'workflow'` field on `CommandBase` — used to badge workflows in autocomplete |
| `components/WorkflowMultiselectDialog.tsx` | UI for selecting multiple workflows at once (Space to toggle, Enter to confirm) |
| `components/tasks/BackgroundTask.tsx` | Renders workflow tasks in the background task list (handles `task.type === 'local_workflow'`) |
| `entrypoints/sdk/coreSchemas.ts` | SDK schema for `task_started` events with `workflow_name` field |
| `constants/prompts.ts` | Skill/discover guidance referencing workflow discovery |

### Workflow Commands in the Command System

```typescript
// commands.ts
const workflowsCmd = feature('WORKFLOW_SCRIPTS')
  ? (await import('./commands/workflows/index.js')).default
  : null
// ...
...(workflowsCmd ? [workflowsCmd] : []),  // registered as slash commands

// loadAllCommands()
const workflowCommands = await getWorkflowCommands(cwd)
return [...workflowCommands, ...]  // merged with skills, plugins, built-in commands
```

The `formatDescriptionWithSource()` function renders `kind === 'workflow'` commands as `"{description} (workflow)"` in autocomplete to distinguish them from regular skills.

### Bundled Workflows

`tools/WorkflowTool/bundled/` contains the built-in workflow scripts that ship with Claude Code. These are initialized at startup via `initBundledWorkflows()` in `tools.ts`.

### Custom Workflows

Teams can add their own workflows by placing scripts in:
- `.claude/workflows/` — project-scoped (committed to repo)
- `~/.claude/workflows/` — user-scoped (personal to each developer)

### Task Integration

Workflow steps run as `BackgroundTask` objects (`task.type === 'local_workflow'`). The SDK emits `task_started` events with `workflow_name` so external dashboards can track which workflow a task belongs to.
