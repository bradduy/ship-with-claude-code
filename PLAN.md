# Hands-On Plan: Build Agentic Apps với Claude Code Patterns

> Inspired by [Deep Researcher](https://github.com/jackswl/deep-researcher) — một ví dụ hoàn hảo về cách dùng Claude Code's framework patterns để build agentic system khác ngoài code editing.

---

## 🎯 Mục Tiêu

Áp dụng những production-tested patterns từ Claude Code source để build một agentic application mới. Không copy code — học cách suy nghĩ và thiết kế agent.

**Kết quả cuối:** một application hoàn chỉnh, chạy được, có thể deploy, với documentation đầy đủ.

---

## 📚 Giai Đoạn 1: Nghiên Cứu Patterns từ Claude Code

### 1.1 Đọc Source Code (2-3 giờ)

Đọc những file quan trọng nhất để hiểu core patterns:

```bash
# Core Agent Loop
cat src/query.ts                    # Agentic loop: API call → tool execution → yield

# Tool Interface
cat src/Tool.ts                     # Tool contract: call(), render(), permissions

# Tool Orchestration
cat src/services/tools/toolOrchestration.ts  # Concurrent partitioning

# Token Management
cat src/services/compact/compact.ts        # Auto-compaction khi gần context limit
```

### 1.2 Ghi Chép Patterns Quan Trọng

Điền bảng này khi đọc code:

| Pattern | File | Mô Tả | Khi Nào Dùng |
|---|---|---|---|
| Agentic Loop | `query.ts` | while loop: LLM → tools → results → decide | Khi cần multi-step task |
| Partitioned Execution | `toolOrchestration.ts` | Read tools concurrently, write tools serially | Khi tool calls có thể parallelize |
| Structured Results | `Tool.ts` | `{ text, data }` — human readable + structured | Khi tools cần return data + summary |
| Token Budget | `compact.ts` | Keep context within limit via summarization | Khi conversation dài |
| Permission System | `Tool.ts` | `checkPermissions()` → allow/deny | Khi cần security layer |
| Retry/Recovery | `query.ts` | Exponential backoff, graceful degradation | Production-critical tools |

### 1.3 Architecture Mapping

Deep Researcher đã làm mapping này. Tham khảo trong README của nó:

| Claude Code (TS) | Deep Researcher (Python) |
|---|---|
| `queryLoop()` in `query.ts` | `_search_phase()` in `agent.py` |
| `Tool.ts` with schema + execute | `Tool` class + `execute()` → `ToolResult` |
| `partitionToolCalls()` batching | `execute_partitioned()` via ThreadPoolExecutor |
| `ToolResult<T>` with data + messages | `ToolResult` with text + papers |
| `autoCompact` token-aware compression | `_compact_messages` with token estimation |

---

## 💡 Giai Đoạn 2: Chọn Ý Tưởng

Chọn MỘT trong các ý tưởng dưới, hoặc đề xuất ý tưởng riêng:

### Option A: Code Review Agent (Medium)
> Agent phân tích PR, suggest improvements, run tests, generate review report.

```
User: "Review PR #123"
Agent:
  1. Fetch PR diff (git diff tool)
  2. Fetch file contents (Read tool)
  3. Analyze code quality (LLM reasoning)
  4. Run lint/tests (Bash tool)
  5. Generate review comment (Write tool)
Output: review.md + inline comments
```

**Pattern sử dụng:**
- Agentic loop (multi-step)
- Partitioned execution (read files concurrently)
- Tool result structured data (diff + lint results)

### Option B: Interview Trainer (Medium)
> Agent đóng vai interviewer, generate questions, evaluate answers, provide feedback.

```
User: "Practice system design interview"
Agent:
  1. Generate questions (LLM)
  2. Present question
  3. Receive answer
  4. Evaluate + follow-up (LLM reasoning)
  5. Score + feedback
Output: Interview report với scores
```

**Pattern sử dụng:**
- Multi-turn conversation state
- Token budget management
- Structured tool results

### Option C: Data Pipeline Builder (Hard)
> Agent nhận data source description, generate ETL pipeline code, test it.

```
User: "Build ETL pipeline from CSV to PostgreSQL"
Agent:
  1. Read CSV schema (file tool)
  2. Generate SQL schema (LLM reasoning)
  3. Write Python ETL script (file tool)
  4. Run test (bash tool)
  5. Generate deployment config
Output: etl.py + schema.sql + docker-compose.yml
```

**Pattern sử dụng:**
- Full tool suite (Read, Write, Bash, Glob, Grep)
- Tool orchestration với partitioning
- Retry/recovery on tool failure
- Permission system

### Option D: Security Scanner (Hard)
> Agent scan codebase for vulnerabilities, categorize by severity, suggest fixes.

```
User: "Scan this repo for security issues"
Agent:
  1. List files (Glob tool)
  2. Read sensitive files (Read tool)
  3. Pattern matching (Grep tool)
  4. Analyze với rules (LLM reasoning)
  5. Generate report
Output: security-report.md với severity levels
```

### Option E: Your Idea
> Đề xuất ý tưởng riêng. Hỏi Claude Code để brainstorm.

---

## 🏗️ Giai Đoạn 3: Thiết Kế Architecture

Sau khi chọn ý tưởng, trả lời những câu hỏi sau:

### 3.1 Define Your Agent's Domain

```
Tên agent: _______________
Input: _______________ (user prompt, file, API, etc.)
Output: _______________ (report, file, API response, etc.)
Primary goal: _______________
```

### 3.2 Design Tool Suite

Mỗi tool = một hành động cụ thể. Thiết kế tool suite của bạn:

```
Tools:
├── read_file     → Read file contents
├── search_code   → Grep pattern trong codebase
├── list_files    → Glob pattern matching
├── write_content → Write/overwrite file
├── run_command   → Execute shell command
├── http_request  → Call external API
├── query_db      → Query database
└── send_email   → Send notification
```

### 3.3 Map Claude Code Patterns

| Claude Code Pattern | Your Implementation |
|---|---|
| `queryLoop()` → `Tool.call()` | `agent_loop()` → `tool.execute()` |
| `partitionToolCalls()` | `execute_partitioned()` |
| `ToolResult<T>` | `ToolResult(text, data)` |
| `autoCompact()` | `compact_messages()` |
| `checkPermissions()` | `check_permission()` |
| `renderToolUseMessage()` | `format_tool_call()` |

---

## 🔨 Giai Đoạn 4: Implementation

### 4.1 Project Structure (Template)

```
my-agent/
├── src/
│   ├── __main__.py        # CLI entry point
│   ├── agent.py           # Core agent loop
│   ├── llm.py            # LLM client (OpenAI-compatible)
│   ├── models.py         # Data models (ToolResult, etc.)
│   ├── config.py         # Config loading
│   ├── tools/
│   │   ├── __init__.py
│   │   ├── base.py       # Tool base class
│   │   └── *.py          # Individual tools
│   └── report.py         # Output generation
├── tests/
│   └── test_agent.py
├── pyproject.toml
└── README.md
```

### 4.2 Step-by-Step Implementation

**Step 1: Tool Base Class** (`tools/base.py`)

```python
from dataclasses import dataclass
from typing import Any

@dataclass
class ToolResult:
    text: str           # Human-readable summary
    data: Any = None     # Structured data (optional)

class Tool:
    name: str = ""
    description: str = ""
    parameters: dict = {}
    is_read_only: bool = True

    def execute(self, **kwargs) -> ToolResult:
        raise NotImplementedError

    def to_openai_schema(self) -> dict:
        return {
            "type": "function",
            "function": {
                "name": self.name,
                "description": self.description,
                "parameters": self.parameters,
            }
        }
```

**Step 2: LLM Client** (`llm.py`)

```python
import openai
from dataclasses import dataclass

@dataclass
class Message:
    role: str
    content: str

class LLMClient:
    def __init__(self, model: str, api_key: str, base_url: str):
        self.client = openai.OpenAI(api_key=api_key, base_url=base_url)

    def chat(
        self,
        messages: list[Message],
        tools: list[dict],
    ) -> dict:
        response = self.client.chat.completions.create(
            model=self.model,
            messages=[{"role": m.role, "content": m.content} for m in messages],
            tools=tools,
            tool_choice="auto",
        )
        return response
```

**Step 3: Agent Loop** (`agent.py`)

```python
def agent_loop(user_prompt: str, tools: list[Tool], llm: LLMClient):
    messages = [
        Message(role="system", content="You are a helpful research assistant."),
        Message(role="user", content=user_prompt),
    ]

    while True:
        response = llm.chat(messages, [t.to_openai_schema() for t in tools])

        if response.tool_calls:
            tool_results = execute_partitioned(tools, response.tool_calls)

            for result in tool_results:
                messages.append(Message(
                    role="tool",
                    content=result.text,
                    tool_call_id=result.call_id,
                ))

            # Check if done
            if is_final_response(response):
                break
        else:
            # Direct text response
            print(response.content)
            break
```

**Step 4: Partitioned Execution** (`tools/base.py`)

```python
from concurrent.futures import ThreadPoolExecutor, as_completed

def execute_partitioned(tools: list[Tool], tool_calls: list[dict]) -> list[ToolResult]:
    """Read-only tools: concurrent. Write tools: serial."""

    # Partition
    batches = []
    for tc in tool_calls:
        tool = get_tool_by_name(tools, tc["name"])
        is_safe = tool.is_read_only if tool else False

        if batches and is_safe and batches[-1][0]:
            batches[-1][1].append(tc)
        else:
            batches.append((is_safe, [tc]))

    # Execute
    results = []
    for is_concurrent, batch in batches:
        if is_concurrent and len(batch) > 1:
            results.extend(run_concurrent(tools, batch))
        else:
            for tc in batch:
                tool = get_tool_by_name(tools, tc["name"])
                result = tool.execute(**parse_args(tc["arguments"]))
                results.append(result)

    return results
```

**Step 5: Add Your Domain Tools**

```python
# Ví dụ: Code Review Agent tools

class ReadFileTool(Tool):
    name = "read_file"
    description = "Read contents of a file"
    parameters = {
        "type": "object",
        "properties": {
            "path": {"type": "string", "description": "File path to read"}
        },
        "required": ["path"]
    }
    is_read_only = True

    def execute(self, path: str) -> ToolResult:
        with open(path) as f:
            content = f.read()
        return ToolResult(
            text=f"Read {len(content)} chars from {path}",
            data={"path": path, "content": content}
        )


class RunCommandTool(Tool):
    name = "run_command"
    description = "Run a shell command"
    parameters = {
        "type": "object",
        "properties": {
            "command": {"type": "string", "description": "Command to run"}
        },
        "required": ["command"]
    }
    is_read_only = False  # Write tool: runs serially

    def execute(self, command: str) -> ToolResult:
        result = subprocess.run(command, shell=True, capture_output=True)
        return ToolResult(
            text=f"Exit code: {result.returncode}\n{result.stdout}",
            data={"exit_code": result.returncode, "output": result.stdout}
        )
```

### 4.3 Testing

```python
# tests/test_agent.py
import pytest
from my_agent.agent import agent_loop
from my_agent.llm import LLMClient

def test_agent_responds():
    llm = LLMClient(model="gpt-5.4", api_key="test")
    tools = []

    # Mock LLM response
    result = agent_loop("Hello", tools, llm)
    assert result is not None
```

---

## 📖 Giai Đoạn 5: Documentation

### 5.1 README Structure

```
## What This Does

Một agentic [domain] assistant. Không phải [alternative tools]. Điểm khác biệt:
- [Feature 1]
- [Feature 2]
- [Feature 3]

## Quick Start

```bash
pip install -e .
my-agent "[your input]"
```

## How It Works

```
User Input → Agent Loop → Tool Execution → Output
```

**Patterns used from Claude Code:**
- Agentic loop
- Partitioned concurrent execution
- Structured tool results
- Token-aware context management

## Architecture

```
src/
├── agent.py       # Core loop
├── llm.py        # LLM client
├── tools/         # Tool implementations
└── report.py      # Output generation
```

## Configuration

```json
{
  "model": "...",
  "api_key": "...",
  ...
}
```

## Extending

Thêm tool mới:

```python
from my_agent.tools.base import Tool, ToolResult

class MyTool(Tool):
    name = "my_tool"
    description = "..."
    parameters = {...}
    is_read_only = True

    def execute(self, **kwargs) -> ToolResult:
        return ToolResult(text="result", data={...})
```
```

---

## 🚀 Checklist Hoàn Thành

- [ ] Đọc và ghi chép 6 patterns từ Claude Code source
- [ ] Chọn ý tưởng (A/B/C/D hoặc riêng)
- [ ] Thiết kế architecture trên giấy
- [ ] Implement Tool base class
- [ ] Implement LLM client
- [ ] Implement agent loop
- [ ] Implement partitioned execution
- [ ] Thêm ít nhất 3 domain tools
- [ ] Viết tests
- [ ] Chạy thử end-to-end
- [ ] Viết README đầy đủ
- [ ] Push lên GitHub

---

## 📚 Tài Liệu Tham Khảo

- [Claude Code Source](https://github.com/anthropics/claude-code) — source map exposure, ~512K lines
- [Deep Researcher](https://github.com/jackswl/deep-researcher) — reference implementation
- [Claude Code README](./README.md) — kiến trúc chi tiết của Claude Code
- [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook) — ví dụ về tool calling
- [OpenAI Function Calling](https://platform.openai.com/docs/guides/function-calling) — API reference
