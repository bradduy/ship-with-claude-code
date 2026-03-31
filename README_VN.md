# Claude Code — Cơ Sở Hạ Tầng AI Agent

Đây là monorepo của **Claude Code**, một trợ lý lập trình AI chạy trên terminal, được xây dựng bằng TypeScript/React. Claude Code cung cấp một AI agent runtime đầy đủ tính năng: engine truy vấn streaming, thư viện 40+ công cụ có thể kết hợp, hệ thống plugin và skill, tích hợp MCP (Model Context Protocol), và một UI renderer tùy chỉnh cho terminal.

---

## Claude Code Là Gì

Claude Code chạy như một REPL tương tác trên terminal. Bạn nhập yêu cầu bằng ngôn ngữ tự nhiên, và agent sẽ:

1. **Xây dựng context** — git status, cấu trúc project, file memory, system capabilities
2. **Gọi Claude API** với context và nhận stream phản hồi
3. **Thực thi tools** theo thứ tự hoặc song song (đọc file, chạy bash, web search, spawn sub-agent, v.v.)
4. **Trả về kết quả** — text, diffs, summaries, hoặc tool results — được render trên terminal

Agent hoàn toàn có thể **lập trình được**: nhúng vào tool khác, điều khiển qua Bridge API, mở rộng bằng plugins và MCP servers, và gọi qua SDK từ bất kỳ Node.js process nào.

---

## Cấu Trúc Repository

```
claude-code/
├── main.tsx              # REPL entry point (compiled binary)
├── setup.ts              # Startup orchestrator (session, git root, memory, hooks)
├── QueryEngine.ts        # Core class: quản lý conversation turn, budget, API calls
├── query.ts              # Query loop: stream từ Claude API, orchestrate tools
├── query/                # Query engine internals (transitions, config, deps)
├── Tool.ts               # Tool interface contract (call, render, permissions, schema)
├── tools.ts              # Tool registry (assemble all tools, dead-code elimination)
├── tools/                # 40+ individual tool implementations
├── Task.ts               # Task types (LocalAgentTask, RemoteAgentTask)
├── tasks/                # Task system (spawn sub-agents, track progress)
├── commands.ts           # Slash command registry (~70 commands)
├── commands/            # Mỗi directory = một slash command
├── context.ts            # Builds system prompt context (git, files, memory)
├── memdir/               # Memory directory system (.claude/, CLAUDE.md files)
├── history.ts            # Session history và paste tracking
├── cost-tracker.ts      # Token usage và cost tracking
├── hooks/                # 60+ React hooks cho UI state
├── components/            # React UI components (REPL, desktop, settings)
├── ink/                  # Custom terminal UI renderer (React + Yoga + Ink reconciler)
├── ink.ts                # Ink renderer entry point
├── state/                # Zustand global state (AppState, store)
├── bootstrap/            # App initialization, session state, startup profiler
├── services/             # Backend services (MCP, API, compact, analytics, voice)
├── bridge/               # IDE Bridge: headless CLI điều khiển bởi desktop/web clients
├── coordinator/          # Coordinator mode: multi-agent team orchestration
├── plugins/              # Plugin loader, built-in plugins, bundled skills
├── skills/                # Skill system (bundled skills, skill discovery)
├── server/               # Internal HTTP server (diagnostics, metrics)
├── schemas/               # Zod schemas (hooks, plugin manifests)
├── types/                 # Shared TypeScript type definitions
├── native-ts/            # Native bindings (yoga-layout, file-index)
├── utils/                 # 200+ utility modules
├── entrypoints/           # SDK/MCP/CLI entry points
├── voice/                 # Voice mode (STT, wake word, keyterms)
├── migrations/             # Session/data migrations
└── remote/                # Remote session management
```

---

## Cách AI Agent Hoạt Động

### Query Loop

```
User input
    ↓
processUserInput()         # Slash commands, model selection, allowed tools
    ↓
fetchSystemPromptParts()   # Build context: git status, CLAUDE.md, memory files
    ↓
query()                    # Stream từ Claude API
    ↓
Tool execution loop        # runTools() → Tool.call()
    ├→ File system (Bash, Read, Edit, Write, Glob, Grep)
    ├→ Web (WebSearch, WebFetch)
    ├→ Tasks (TaskCreate, TaskStop, Agent)
    ├→ Skills (SkillTool)
    ├→ MCP servers (MCPTool)
    └→ User interaction (AskUserQuestion)
    ↓
Tool result → API stream  # Continue loop với results
    ↓
Yield result               # Text, diffs, summaries → UI render
    ↓
Post-processing            # Hooks (PreToolUse, PostToolUse), analytics, compact
```

### Xây Dựng Context (`context.ts`)

Mỗi query xây dựng một **system prompt** từ nhiều nguồn:

- **Git context**: branch, status, diff summary, worktree configuration
- **Project structure**: file tree, key config files
- **Memory files**: `CLAUDE.md`, `CLAUDE_DEVELOPER.md`, `.claude/` directory files (recursive)
- **Tool descriptions**: tất cả available tools với dynamic descriptions
- **Slash commands**: tất cả registered commands với descriptions
- **Custom instructions**: loaded từ `CLAUDE.md` frontmatter hooks

### Budget & Compaction

Query engine theo dõi **token budgets per turn**. Khi gần đến giới hạn:

1. **Compact** — summarises recent conversation để giải phóng context space
2. **Snip** — removes middle turns của long sessions (HISTORY_SNIP feature)
3. **Reactive compact** — event-driven compaction triggered by hook (`PostCompact`)

---

## Hệ Thống Tools (`tools/`, `Tool.ts`, `tools.ts`)

Tools là **hành động** mà agent có thể thực hiện. `Tool` interface định nghĩa ~30 methods bao gồm:

| Aspect | Methods |
|---|---|
| **Execution** | `call(args, context, canUseTool)` → `ToolResult` |
| **Schema** | `inputSchema` (Zod), `inputJSONSchema` (raw JSON, for MCP) |
| **Permissions** | `checkPermissions`, `validateInput`, `preparePermissionMatcher` |
| **Safety** | `isConcurrencySafe`, `isReadOnly`, `isDestructive`, `isOpenWorld` |
| **UI rendering** | `renderToolUseMessage`, `renderToolResultMessage`, `renderToolUseProgressMessage`, `renderToolUseErrorMessage`, `renderToolUseRejectedMessage` |
| **Summarisation** | `getToolUseSummary`, `getActivityDescription`, `extractSearchText` |
| **Classification** | `toAutoClassifierInput` (cho auto-mode security) |
| **Interruption** | `interruptBehavior()` → `'cancel' \| 'block'` |

Các tools chính:

| Tool | File | Mục đích |
|---|---|---|
| `Bash` | `BashTool/` | Thực thi shell commands |
| `Read` | `FileReadTool/` | Đọc nội dung file |
| `Edit` | `FileEditTool/` | Sửa file (diff-based) |
| `Write` | `FileWriteTool/` | Ghi/overwrite file |
| `Glob` | `GlobTool/` | Tìm file theo glob pattern |
| `Grep` | `GrepTool/` | Tìm kiếm nội dung file |
| `Agent` | `AgentTool/` | Spawn sub-agents (LocalAgentTask / RemoteAgentTask) |
| `Skill` | `SkillTool/` | Gọi skills |
| `MCP` | `MCPTool/` | Gọi tools trên MCP servers |
| `EnterPlanMode` | `EnterPlanModeTool/` | Vào structured planning mode |
| `ExitPlanMode` | `ExitPlanModeV2Tool/` | Thoát planning mode |
| `AskUserQuestion` | `AskUserQuestionTool/` | Hỏi user với choices |
| `TaskCreate` | `TaskCreateTool/` | Tạo subtasks |
| `EnterWorktree` | `EnterWorktreeTool/` | Tạo git worktree |
| `WebSearch` | `WebSearchTool/` | Tìm kiếm web |
| `WebFetch` | `WebFetchTool/` | Fetch web page content |
| `TodoWrite` | `TodoWriteTool/` | Quản lý todo list |

Tool execution được orchestrate bởi `services/tools/StreamingToolExecutor.ts` (streaming output) và `services/tools/toolOrchestration.ts` (`runTools()`), xử lý parallel tool calls, sequential dependencies, và abort controllers.

---

## Hệ Thống Commands (`commands/`, `commands.ts`)

Slash commands cung cấp **user-facing features** được gọi bằng `/`. Có ~70 commands trong các categories:

| Category | Ví dụ |
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

Mỗi command nằm trong directory riêng dưới `commands/` với `index.ts` entry point.

---

## Hệ Thống Skills

Skills là **các khả năng được đặt tên** mà agent có thể gọi — chúng trông như slash commands (`/simplify`, `/brainstorming`) nhưng được cung cấp bởi structured prompts với full tool access. Skills nằm giữa freeform chat và built-in command: model quyết định khi nào gọi một skill dựa trên conversation, và skill's prompt hướng dẫn agent qua một multi-step process.

### Ba Nguồn Skills

| Nguồn | Location | Đăng ký | Gọi bởi |
|---|---|---|---|
| **Bundled** | `skills/bundled/*.ts` | `initBundledSkills()` at startup | Mọi người |
| **Disk-based** | `.claude/skills/`, `~/.claude/skills/` | `getSkillDirCommands()` lazily | Project/user |
| **MCP-sourced** | MCP servers | `registerMCPSkillBuilders()` | Remote tools |

### Cách Load Một Skill

```
User nhập /<skill-name>
    ↓
findCommand() tra cứu trong command registry
    ↓
SkillTool.call(skillName, args, context)
    ↓
getPromptForCommand(args, toolUseContext)
    — load markdown content
    — substitute ${CLAUDE_SKILL_DIR}, ${CLAUDE_SESSION_ID}, argument placeholders
    — execute inline shell blocks (!`…`) nếu loadedFrom !== 'mcp'
    ↓
Returns ContentBlockParam[] → injected vào conversation như system prompt
    ↓
Model tiếp tục với skill's guidance
```

### Bundled Skills (`skills/bundled/`)

Mỗi bundled skill là một TypeScript file gọi `registerBundledSkill(definition)`.

| Skill | Feature Flag | Mục đích |
|---|---|---|
| `update-config` | — | Configure Claude Code settings |
| `keybindings` | — | Keybinding reference |
| `verify` | — | Plan verification |
| `debug` | — | Systematic debugging |
| `simplify` | — | Code review: reuse + quality + efficiency |
| `stuck` | — | Help khi bị stuck |
| `batch` | — | Batch multiple tasks |
| `brainstorming` | — | Khám phá ý tưởng, thiết kế |
| `loop` | `AGENT_TRIGGERS` | Recurring task on interval |
| `schedule-remote-agents` | `AGENT_TRIGGERS_REMOTE` | Schedule agents via cron |
| `dream` | `KAIROS` | Background reflection |
| `claude-api` | `BUILDING_CLAUDE_APPS` | Build with Claude API/SDK |

### Skills Trên Disk

Skills trên disk sử dụng **directory format**:

```
.claude/skills/
├── simplify/
│   └── SKILL.md        ← skill content + frontmatter
├── brainstorm/
│   └── SKILL.md
└── sprint-planning/
    └── SKILL.md
```

**SKILL.md frontmatter** kiểm soát behavior:

```markdown
---
name: sprint-planning
description: Lên kế hoạch sprint từ backlog
when_to_use: Khi bắt đầu sprint mới hoặc sprint planning meeting.
allowed_tools: [Bash, Read, Glob]
argument_hint: Tên sprint (VD: "Sprint 42")
context: inline    # hoặc "fork" để chạy như sub-agent
---
# Sprint Planning

1. Đọc backlog từ Linear/Jira
2. Estimate points cho từng story
3. Assign stories cho team members
4. Output sprint plan
```

### Conditional Skills (Path-Filtered)

Skills với `paths` frontmatter là **conditional** — chúng chỉ activate khi model touch matching files:

```markdown
---
paths:
  - "**/*.tsx"
---
# React Best Practices

Check component patterns on TSX files...
```

### Hooks Trên Skills

Skills có thể đăng ký hooks để fire khi skill chạy:

```markdown
---
hooks:
  PreToolUse:
    - matcher: "Bash(git *)"
      hooks:
        - type: "command"
          command: "echo 'running {tool} with {input}'"
---
```

---

## Hệ Thống Hooks

Hooks là **cơ chế mở rộng** để intercept và modify agent behavior. Có hai hệ thống hooks:

### Native Hooks (chạy bởi CLI):

| Event | Mục đích |
|---|---|
| `SessionStart` | Chạy mỗi khi REPL khởi động |
| `SessionStop` | Chạy khi REPL thoát |
| `UserPromptSubmit` | Fire trước khi user input được xử lý |
| `PreToolUse` | Fire trước khi tool được gọi |
| `PostToolUse` | Fire sau khi tool hoàn thành |
| `MessageStart` | Fire khi AI message bắt đầu |
| `MessageStop` | Fire khi AI message hoàn thành |
| `AfterMessage` | Post-sampling hook |
| `PostCompact` | Fire sau compaction |
| `Embedding` | Chạy trên file embedding |

### Plugin Hooks (định nghĩa trong plugin manifests):

- `command` hooks — chạy shell commands có điều kiện
- `async` hooks — non-blocking callbacks

Hooks được configure trong `hooks/hooks.json` hoặc trong project `CLAUDE.md` frontmatter.

---

## Workflows (Feature-Gated: `WORKFLOW_SCRIPTS`)

Workflows là **scripted multi-step agent procedures** — được định nghĩa trong YAML hoặc TypeScript, chúng cho phép teams encode một repeatable development process (ví dụ: "write spec → implement → review → test") như một first-class agent capability.

### Kiến Trúc

```
Feature flag: WORKFLOW_SCRIPTS
    ↓
WorkflowTool (tools/WorkflowTool/WorkflowTool.ts)     # Tool agent gọi
    ↓
createWorkflowCommand (tools/WorkflowTool/createWorkflowCommand.ts)  # Load workflow scripts
    ↓
Bundled + custom workflow scripts  (tools/WorkflowTool/bundled/)   # Scripts thực tế
    ↓
workflow commands  (kind: 'workflow')   # Registered như slash commands
```

### Custom Workflows

Teams có thể thêm workflows riêng bằng cách đặt scripts trong:

- `.claude/workflows/` — project-scoped (commit vào repo)
- `~/.claude/workflows/` — user-scoped (riêng cho mỗi developer)

---

## MCP (Model Context Protocol)

Hệ thống MCP (`services/mcp/`) kết nối Claude Code với external tools:

- **MCP Client**: kết nối đến bất kỳ MCP server nào qua stdio, HTTP+SSE, hoặc Streamable HTTP
- **MCP Tool**: wrap MCP tools để chúng xuất hiện như native Claude Code tools
- **OAuth support**: cho remote MCP server authentication
- **Tool normalization**: chuyển đổi MCP tool schemas sang Claude Code's format

MCP servers được configure trong `settings.json` hoặc qua `claude mcp add`.

---

## UI: Ink Renderer

Claude Code render UI trong **terminal** sử dụng **Ink** — một custom React renderer:

- React components → terminal ANSI output (không có browser)
- **Yoga layout engine**: flexbox layout trong terminal
- Custom **React reconciler**: xử lý incremental updates
- Hỗ trợ: mouse tracking, hyperlinks, color, selection, scrollback

UI layer (`components/`) xây dựng trên Ink primitives:
- `Message.tsx` / `MessageRow.tsx` — chat messages
- `ToolUseLoader.tsx` — tool execution progress
- `StructuredDiff.tsx` — code diff display
- `StatusLine.tsx` — status bar
- `ContextVisualization.tsx` — context inspection overlay

---

## SDLC: Software Development Lifecycle

Dựa trên kiến trúc Claude Code, đây là những gì bạn có thể làm cho dự án:

### 1. Khởi Tạo Dự Án

```
/init              → bootstrap project, install deps, setup config
/init-verifiers    → thêm verification tooling
```

### 2. Thiết Kế & Brainstorm

```
/superpowers:brainstorming  → khám phá ý tưởng, đặt câu hỏi, trình bày design
                            → ghi spec ra docs/superpowers/specs/
```

### 3. Lập Kế Hoạch

```
/enter-plan-mode  → tạo todo list, phân tích task
                  → tự động tạo TaskCreate cho từng bước
```

### 4. Triển Khai Code

```
# Code trực tiếp
Claude Code Agent
    ├── Read / Glob / Grep    → đọc codebase
    ├── Edit / Write          → sửa / tạo file
    ├── Bash                  → chạy build, test, lint
    └── Agent                 → spawn sub-agent cho task song song

# Code quality
/simplify          → 3 parallel agents: reuse + quality + efficiency review
/debug             → systematic debugging methodology
/stuck             → help khi bị stuck
```

### 5. Review Code

```
/review            → code review
/security-review   → bảo mật
/git diff          → xem thay đổi
```

### 6. Git Operations

```
/commit            → commit với conventional commit format
/branch            → tạo branch
/pr_comments       → review PR comments
/commit-push-pr    → commit + push + tạo PR
```

### 7. Testing

```
BashTool           → chạy npm test, pytest, etc.
TaskCreateTool     → tạo test task song song
AgentTool          → spawn test-writing agent
```

### 8. Deployment

```
Bash               → chạy deploy scripts
Vercel / GitHub Actions / Docker...
```

---

## Đặc Biệt Mạnh Cho SDLC

### Custom Skills — Dự Án Của Bạn

Tạo `.claude/skills/` trong project của bạn:

```
.claude/skills/
├── sprint-planning/
│   └── SKILL.md           → "Lên kế hoạch sprint từ backlog"
├── api-documentation/
│   └── SKILL.md           → "Generate API docs từ code"
├── on-call-runbook/
│   └── SKILL.md           → "Debug production issue"
└── release-checklist/
    └── SKILL.md           → "Chạy release checklist"
```

### Hooks — Tự Động Hóa Không Cần Gọi Skill

```
PreToolUse Hook    → trước mỗi tool: log, validate, require confirmation
PostToolUse Hook   → sau mỗi tool: auto-format, generate docs
SessionStart Hook  → mỗi lần khởi động: pull env vars, check deps
```

**Ví dụ** — tự động chạy lint sau mỗi Edit:

```json
// hooks/hooks.json
{
  "PostToolUse": [{
    "matcher": "Edit",
    "hooks": [{
      "type": "command",
      "command": "npm run lint -- {file}"
    }]
  }]
}
```

### Workflows — Multi-Step Repeatable Processes

```
.spec workflow     → brainstorm → write spec → review → implement
.test workflow    → write test → run → fix
.ci workflow      → lint → test → build → deploy
```

### Memory Files — Claude Code Nhớ Context Dài

```
CLAUDE.md          → project-wide instructions (team conventions, architecture)
CLAUDE_DEVELOPER.md → development guidelines
.claude/           → team memory, runbooks, decisions
```

### Sub-Agents — Parallelism Cho SDLC

```
TaskCreate + AgentTool
    ├── Agent: viết feature A
    ├── Agent: viết feature B
    ├── Agent: viết tests
    └── Agent: viết docs
    → chạy song song, aggregate kết quả
```

---

## Bảng Mapping SDLC Phase → Claude Code

| SDLC Phase | Claude Code Capability |
|---|---|
| **Requirements** | `/brainstorming` skill → design docs |
| **Design** | Memory files + context building |
| **Implementation** | Full tool suite + skills |
| **Code Review** | `/review`, `/simplify`, `/security-review` |
| **Testing** | Bash + Agent spawn parallel tests |
| **Deployment** | Bash + Vercel CLI + workflows |
| **Maintenance** | `/debug`, `/stuck`, on-call runbook skills |
| **Onboarding** | Memory files (`CLAUDE.md`) cho team mới |
| **Continuous Improvement** | `/loop` skill cho recurring tasks |

---

## Quick Start Cho Dự Án Của Bạn

```bash
# 1. Tạo CLAUDE.md cho project conventions
cat > CLAUDE.md << 'EOF'
# Project Conventions

## Architecture
- Monorepo với Turborepo
- Next.js cho frontend, Node.js backend

## Code Style
- TypeScript strict mode
- ESLint + Prettier

## Git Workflow
- Conventional commits
- PR requires 1 approval
EOF

# 2. Thêm skills cho dự án
mkdir -p .claude/skills/sprint
cat > .claude/skills/sprint/SKILL.md << 'EOF'
---
name: sprint
description: Lên kế hoạch sprint từ backlog
---
# Sprint Planning

1. Đọc backlog từ Linear/Jira
2. Estimate points
3. Assign stories
4. Output sprint plan
EOF

# 3. Setup hooks cho automation
cat > hooks/hooks.json << 'EOF'
{
  "PostToolUse": [{
    "matcher": "Bash(npm test*)",
    "hooks": [{ "type": "command", "command": "open coverage/index.html" }]
  }]
}
EOF
```

---

## Key Files Reference

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
| `skills/bundledSkills.ts` | `registerBundledSkill()` + bundled skill extraction |
| `skills/loadSkillsDir.ts` | Load skills từ disk + conditional skills |
| `skills/bundled/index.ts` | `initBundledSkills()` — tất cả bundled skills |
| `tools/SkillTool/` | SkillTool cho model discover và invoke skills |
| `state/AppState.tsx` | React global state (Zustand) |
| `bootstrap/state.ts` | Non-React session state |
| `history.ts` | Session transcript |
| `cost-tracker.ts` | Token usage tracking |
| `ink.tsx` | Terminal UI renderer entry |
| `bridge/bridgeMain.ts` | Bridge server cho IDE integration |
| `services/mcp/` | MCP client implementation |
| `plugins/` | Plugin loader và built-in plugins |
| `utils/hooks/` | Native hook system |
| `types/message.ts` | Core message types |
| `types/permissions.ts` | Permission types và rules |
