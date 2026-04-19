# Claude Code 101 — Ghi chú khóa học

> Nguồn: [Anthropic SkillJar — Claude Code 101](https://anthropic.skilljar.com/claude-code-101/469793)

---

## Tổng quan khóa học

Claude Code 101 được thiết kế cho hai đối tượng:
- **Developers mới** đang bắt đầu với lập trình phần mềm
- **Developers có kinh nghiệm** chưa khám phá AI coding agents

Khóa học dạy cách sử dụng Claude Code hiệu quả trong quy trình phát triển phần mềm hàng ngày.

---

## Mục tiêu học tập

Sau khi hoàn thành khóa học, học viên sẽ có thể:

1. **Hiểu AI coding agents** — Phân biệt Claude Code với các công cụ AI chat thông thường
2. **Nắm vững cơ chế hoạt động** — Agentic loops, context windows, tools, và permissions
3. **Cài đặt Claude Code** — Trên terminal, VS Code, JetBrains, Claude Desktop, và web
4. **Thành thạo viết prompt** — Sử dụng approval mode, auto-accept, và Plan Mode
5. **Thực thi workflow** — Quy trình Explore → Plan → Code → Commit
6. **Quản lý context** — Dùng `/compact`, `/clear`, và `/context` commands
7. **Tạo và duy trì CLAUDE.md** — File persistent memory cho project
8. **Xây dựng custom subagents** — Phân công task cho subagents
9. **Tích hợp MCP servers** — Kết nối external tools
10. **Triển khai hooks** — Kiểm soát formatting và commands

---

## Điều kiện tiên quyết

- Quen thuộc cơ bản với code editors và command line
- Tài khoản Claude (Pro, Max, Enterprise) hoặc API key

---

## Cấu trúc chương trình

### Module 1: Claude Code là gì?

| Bài học | Nội dung |
|---------|---------|
| What is Claude Code? | Định nghĩa, so sánh với AI chat thông thường |
| How Claude Code works | Agentic loop, context window, tools, permissions |
| Your first prompt | Thực hành prompt đầu tiên |

**Điểm cốt lõi:**
- Claude Code là một **AI coding agent**, không phải chatbot
- Hoạt động theo vòng lặp: nhận task → lập kế hoạch → thực thi → kiểm tra → lặp lại
- Có khả năng truy cập file system, chạy terminal commands, và dùng tools

---

### Module 2: Cài đặt Claude Code

| Bài học | Nội dung |
|---------|---------|
| Installation guidance | Hướng dẫn cài đặt trên các platform |
| Initial prompt exercises | Bài tập prompt ban đầu |

**Các nền tảng hỗ trợ:**
- Terminal (CLI)
- VS Code extension
- JetBrains plugin
- Claude Desktop app
- Web app

---

### Module 3: Quy trình làm việc hàng ngày

| Bài học | Nội dung |
|---------|---------|
| Explore → Plan → Code → Commit | Core workflow |
| Context management | Quản lý context window |
| Code review practices | Quy trình review code |

**Workflow chính — Explore → Plan → Code → Commit:**

```
1. EXPLORE   — Khám phá codebase, hiểu cấu trúc hiện tại
2. PLAN      — Lập kế hoạch thực hiện (dùng Plan Mode)
3. CODE      — Viết/sửa code với sự hỗ trợ của Claude
4. COMMIT    — Review và commit thay đổi
```

**Quản lý context:**
- `/compact` — Nén context để tiết kiệm không gian
- `/clear` — Xóa toàn bộ context, bắt đầu mới
- `/context` — Xem và quản lý context hiện tại

---

### Module 4: Tùy chỉnh nâng cao

| Bài học | Nội dung |
|---------|---------|
| CLAUDE.md | File cấu hình persistent memory |
| Subagents | Phân công task chuyên biệt |
| Skills | Kỹ năng tùy chỉnh |
| MCP integration | Tích hợp Model Context Protocol |
| Hooks | Tự động hóa với hooks |
| Final quiz | Kiểm tra cuối khóa |

**CLAUDE.md — Project Memory:**
- File đặc biệt nằm ở gốc project
- Claude tự động đọc mỗi khi bắt đầu session
- Dùng để lưu: quy ước code, context project, hướng dẫn cho Claude

**Subagents:**
- Agents chuyên biệt cho từng loại task
- Có thể chạy song song để tăng hiệu suất
- Ví dụ: agent cho testing, agent cho documentation

**MCP Servers:**
- Mở rộng khả năng của Claude với external tools
- Ví dụ: kết nối database, API, file systems
- Cấu hình trong settings của Claude Code

**Hooks:**
- Script chạy tự động khi có sự kiện nhất định
- Dùng cho: auto-formatting, linting, validation
- Kiểm soát commands được phép chạy

---

## Tóm tắt kiến thức quan trọng

### 3 điều khác biệt giữa Claude Code và AI chat thông thường

| Tiêu chí | AI Chat | Claude Code |
|----------|---------|-------------|
| Khả năng | Trả lời, giải thích | Thực thi, viết file, chạy lệnh |
| Context | Một cuộc trò chuyện | Toàn bộ codebase |
| Workflow | Q&A | Agentic loop |

### Best practices

- Sử dụng **Plan Mode** trước khi thực thi task lớn
- Viết **CLAUDE.md** chi tiết để Claude hiểu context project
- Dùng **subagents** cho task phức tạp hoặc song song
- Quản lý **context window** để tránh mất thông tin quan trọng
- Tích hợp **hooks** để tự động hóa quy trình

---

*Ghi chú được tổng hợp từ khóa học Claude Code 101 — Anthropic SkillJar*
