# Claude Code in Action — Tổng hợp kiến thức

> Nguồn: [Anthropic SkillJar — Claude Code in Action](https://anthropic.skilljar.com/claude-code-in-action)

---

## Tổng quan

Khóa học thực hành chuyên sâu về cách tích hợp Claude Code vào quy trình phát triển phần mềm, bao gồm kiến trúc AI coding assistant, kỹ thuật triển khai thực tế, và các chiến lược tích hợp nâng cao.

**Yêu cầu:** Quen với CLI/terminal + hiểu Git cơ bản

---

## Section 1: What is Claude Code?

### Coding assistant là gì?

Claude Code **không phải chatbot** — nó là một **AI agent** có khả năng:
- Đọc, viết, và chỉnh sửa file trực tiếp
- Chạy terminal commands
- Phân tích toàn bộ codebase, không chỉ đoạn code bạn paste vào
- Thực hiện multi-step tasks tự động

### Cơ chế hoạt động (Tool Use Architecture)

Claude Code tương tác với codebase qua **tool system**:

```
User Prompt
    ↓
Claude phân tích → chọn tool phù hợp
    ↓
Tool thực thi (read file, run bash, search code...)
    ↓
Claude đọc kết quả → quyết định bước tiếp theo
    ↓
Lặp lại cho đến khi hoàn thành task
```

**Các tools cốt lõi:**
| Tool | Chức năng |
|------|-----------|
| `Read` | Đọc nội dung file |
| `Write` | Tạo/ghi đè file |
| `Edit` | Chỉnh sửa đoạn cụ thể trong file |
| `Bash` | Chạy lệnh shell |
| `Grep` | Tìm kiếm trong codebase |
| `Glob` | Tìm file theo pattern |
| `WebFetch` | Lấy nội dung từ URL |

---

## Section 2: Getting Hands On

### 2.1 Claude Code Setup

**Cài đặt:**
```bash
npm install -g @anthropic-ai/claude-code
```

**Xác thực:**
```bash
claude  # Lần đầu sẽ yêu cầu đăng nhập
```

**Các chế độ chạy:**
- `claude` — Interactive mode (mặc định)
- `claude -p "prompt"` — One-shot mode, không cần tương tác
- `claude --continue` — Tiếp tục session trước

---

### 2.2 Project Setup

Việc setup project đúng cách giúp Claude hiểu context ngay từ đầu:

**CLAUDE.md — File quan trọng nhất:**
```markdown
# Project Name

## Tech Stack
- Language: TypeScript
- Framework: Next.js
- Database: PostgreSQL

## Conventions
- Dùng camelCase cho variables
- Mỗi component là một file riêng
- Test file đặt cạnh file source

## Commands
- `npm run dev` — Chạy dev server
- `npm test` — Chạy tests
- `npm run build` — Build production
```

Claude **tự động đọc** CLAUDE.md mỗi khi bắt đầu session — đây là "bộ nhớ dài hạn" của project.

---

### 2.3 Adding Context

Claude cần context để làm việc hiệu quả. Cách cung cấp context:

**1. Reference file trực tiếp:**
```
@src/components/Button.tsx hãy refactor component này
```

**2. Reference folder:**
```
@src/api/ giải thích cách các API endpoints hoạt động
```

**3. Reference URL:**
```
@https://docs.example.com/api tích hợp API này vào project
```

**4. Dùng `#` để reference symbol:**
```
#handleUserLogin function này có bug gì không?
```

**Tips:**
- Chỉ cung cấp context liên quan — quá nhiều context làm giảm chất lượng
- CLAUDE.md là nơi lưu context persistent, không cần nhắc lại mỗi lần

---

### 2.4 Making Changes

**Workflow an toàn khi thay đổi code:**

```
1. Dùng Plan Mode trước (Shift+Tab hoặc /plan)
   → Claude lên kế hoạch, chưa thay đổi gì

2. Review kế hoạch
   → Xác nhận approach đúng không

3. Approve → Claude thực thi
   → Theo dõi từng bước

4. Review diff trước khi commit
   → git diff để kiểm tra
```

**Visual inputs:**
- Có thể paste screenshot UI vào Claude
- Claude phân tích hình ảnh và implement theo design
- Hữu ích khi làm frontend hoặc debug UI bugs

---

### 2.5 Controlling Context

**Vấn đề:** Context window có giới hạn — nếu conversation quá dài, Claude mất thông tin quan trọng từ đầu.

**Các lệnh quản lý context:**

| Lệnh | Tác dụng | Khi nào dùng |
|------|----------|--------------|
| `/compact` | Nén lịch sử thành bản tóm tắt | Conversation dài, muốn tiếp tục |
| `/clear` | Xóa toàn bộ, bắt đầu mới | Chuyển sang task hoàn toàn khác |
| `/context` | Xem mức độ sử dụng context | Kiểm tra còn bao nhiêu |

**Best practices:**
- Bắt đầu session mới cho mỗi task độc lập
- Dùng CLAUDE.md để lưu thông tin cần persist
- Tránh paste code dài không cần thiết vào chat

---

### 2.6 Custom Commands

Tạo commands tái sử dụng cho các task lặp đi lặp lại:

**Tạo custom command:**
Tạo file `.claude/commands/review.md`:
```markdown
# Code Review

Hãy review đoạn code sau theo các tiêu chí:
1. Security vulnerabilities
2. Performance issues
3. Code style consistency
4. Missing error handling
5. Test coverage

Code cần review: $ARGUMENTS
```

**Sử dụng:**
```
/review src/auth/login.ts
```

**Ví dụ custom commands hữu ích:**
- `/review` — Review code theo checklist cố định
- `/test` — Generate test cho function
- `/doc` — Generate documentation
- `/refactor` — Refactor theo convention của project
- `/deploy-check` — Kiểm tra trước khi deploy

---

### 2.7 MCP Servers

**Model Context Protocol (MCP)** — Giao thức mở rộng khả năng của Claude bằng external tools.

**Kiến trúc:**
```
Claude Code ←→ MCP Server ←→ External Service
                            (Browser, DB, API, ...)
```

**Cách thêm MCP server vào Claude Code:**
```json
// ~/.claude/settings.json
{
  "mcpServers": {
    "browser": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/mydb"
      }
    }
  }
}
```

**MCP servers phổ biến:**
| Server | Chức năng |
|--------|-----------|
| Playwright/Puppeteer | Browser automation, web scraping |
| PostgreSQL/SQLite | Truy vấn database trực tiếp |
| GitHub | Tương tác với GitHub API |
| Slack | Gửi message, đọc channel |
| Filesystem | Truy cập file ngoài project |

---

### 2.8 GitHub Integration

**Tự động hóa code review với GitHub Actions:**

```yaml
# .github/workflows/claude-review.yml
name: Claude Code Review

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Claude Code Review
        uses: anthropic-ai/claude-code-action@beta
        with:
          anthropic_api_key: ${{ secrets.ANTHROPIC_API_KEY }}
```

**Kết quả:** Claude tự động comment review trên mỗi PR, chỉ ra:
- Potential bugs
- Security issues
- Performance concerns
- Style violations

---

## Section 3: Hooks and the SDK

### 3.1 Hooks là gì?

Hooks là các script chạy tự động khi có **sự kiện cụ thể** xảy ra trong Claude Code session.

**Luồng hoạt động:**
```
Claude chuẩn bị action
        ↓
Hook chạy (pre-action)
        ↓
Hook có thể: cho phép / chặn / modify
        ↓
Action thực thi (nếu được phép)
        ↓
Hook chạy (post-action)
```

---

### 3.2 Định nghĩa Hooks

**Cấu hình trong `~/.claude/settings.json`:**

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Running bash command: ' && cat"
          }
        ]
      }
    ],
    "PostToolUse": [
      {
        "matcher": "Write",
        "hooks": [
          {
            "type": "command",
            "command": "prettier --write $CLAUDE_TOOL_INPUT_PATH"
          }
        ]
      }
    ]
  }
}
```

**Các loại events:**
| Event | Mô tả |
|-------|-------|
| `PreToolUse` | Trước khi Claude dùng tool |
| `PostToolUse` | Sau khi tool chạy xong |
| `Notification` | Khi Claude gửi notification |
| `Stop` | Khi Claude kết thúc response |

---

### 3.3 Triển khai Hooks thực tế

**Hook tự động format code:**
```json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "npx prettier --write \"$CLAUDE_TOOL_INPUT_PATH\" 2>/dev/null || true"
          }
        ]
      }
    ]
  }
}
```

**Hook chặn lệnh nguy hiểm:**
```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash /path/to/safety-check.sh"
          }
        ]
      }
    ]
  }
}
```

**`safety-check.sh`:**
```bash
#!/bin/bash
INPUT=$(cat)
# Chặn các lệnh nguy hiểm
if echo "$INPUT" | grep -qE "rm -rf|DROP TABLE|format C:"; then
  echo "BLOCKED: Dangerous command detected" >&2
  exit 2  # Exit code 2 = block và báo lỗi cho Claude
fi
exit 0   # Cho phép
```

**Exit codes của hooks:**
- `0` — Cho phép tiếp tục
- `1` — Có lỗi nhưng vẫn tiếp tục
- `2` — Chặn action, báo lỗi cho Claude

---

### 3.4 Gotchas (Lưu ý quan trọng)

- Hook **timeout sau 60 giây** — tránh script chạy quá lâu
- Hook chạy **synchronously** — Claude đợi hook xong mới tiếp tục
- Biến môi trường có sẵn: `$CLAUDE_TOOL_NAME`, `$CLAUDE_TOOL_INPUT_*`
- Hook viết ra **stderr** để gửi message cho Claude
- Hook viết ra **stdout** để log (không gửi cho Claude)

---

### 3.5 Hooks hữu ích

| Hook | Mục đích |
|------|----------|
| Auto-format | Prettier/ESLint sau mỗi lần write |
| Type-check | `tsc --noEmit` sau khi sửa TypeScript |
| Test runner | Chạy relevant tests sau khi thay đổi |
| Git auto-stage | `git add` file vừa được sửa |
| Notification | Desktop notification khi Claude hoàn thành task dài |
| Safety guard | Chặn lệnh destructive |

---

### 3.6 Claude Code SDK

SDK cho phép tích hợp Claude Code vào ứng dụng hoặc script của bạn.

**Cài đặt:**
```bash
npm install @anthropic-ai/claude-code
```

**Sử dụng cơ bản:**
```typescript
import { query } from "@anthropic-ai/claude-code";

async function runClaudeCode() {
  const messages = [];

  for await (const message of query({
    prompt: "Tìm tất cả TODO comments trong project và tạo issue list",
    options: {
      maxTurns: 10,
      cwd: "/path/to/project",
    },
  })) {
    messages.push(message);
    console.log(message);
  }
}
```

**Use cases của SDK:**
- Chạy Claude Code trong CI/CD pipeline
- Xây dựng custom coding agents
- Tích hợp vào internal tools của team
- Tự động hóa repetitive coding tasks theo schedule

---

## Section 4: Thinking & Planning Modes

### Khi nào dùng chế độ nào?

| Chế độ | Kích hoạt | Dùng khi |
|--------|-----------|----------|
| Normal | Mặc định | Task đơn giản, rõ ràng |
| Plan Mode | `Shift+Tab` hoặc `/plan` | Task phức tạp, nhiều file |
| Extended Thinking | `--thinking` flag | Bài toán logic, architecture |
| Auto-accept | `--dangerously-skip-permissions` | Script/automation |

**Plan Mode workflow:**
1. Nhập task → Claude phân tích
2. Claude trình bày kế hoạch chi tiết (chưa làm gì)
3. Bạn review, approve hoặc điều chỉnh
4. Claude thực thi theo kế hoạch đã duyệt

---

## Tóm tắt Best Practices

### Viết prompt hiệu quả
- **Cụ thể** về output mong muốn
- **Cung cấp context** liên quan với `@file` hoặc `@folder`
- **Chia nhỏ** task phức tạp thành các bước
- **Dùng Plan Mode** cho task lớn

### Quản lý project
- **CLAUDE.md** cho mọi project — lưu conventions, commands, context
- **Custom commands** cho tasks lặp lại
- **MCP servers** để mở rộng khả năng

### An toàn và kiểm soát
- **Review** trước khi approve destructive actions
- **Hooks** để enforce standards tự động
- **Git commit** thường xuyên để có rollback point
- Dùng **approval mode** (mặc định) thay vì auto-accept với code production

---

*Tổng hợp từ khóa học Claude Code in Action — Anthropic SkillJar*
