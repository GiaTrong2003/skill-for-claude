# Hướng dẫn Claude viết Skill cho dự án

File này là **prompt nền** đưa cho Claude ở công ty. Mở Claude Code trong thư mục dự án, rồi nói: *"Đọc file `HUONG-DAN-VIET-SKILL.md` và làm theo."*

---

## 0. Bối cảnh dự án

> **TODO (user điền trước khi đưa cho Claude):**
> - Tên dự án:
> - Ngôn ngữ / framework chính:
> - Domain nghiệp vụ (fintech / y tế / logistics / ...):
> - Bounded contexts chính (liệt kê 3-7 cái):
> - Link repo / CLAUDE.md nếu có:

---

## 1. Mục tiêu

Xây bộ **skill + rules** cho Claude để:
1. Code sinh ra đúng convention team
2. Hiểu được **nghiệp vụ** của dự án (không phải chỉ cú pháp)
3. Tự phát hiện khi code lệch khỏi rule/nghiệp vụ
4. Onboard member mới + AI agent mới nhanh hơn

---

## 2. Phân loại artifact — chọn đúng loại khi viết

| Loại | Mục đích | Khi nào viết |
|---|---|---|
| **Rule** (`.claude/rules/*.md`) | Luôn áp dụng, không cần quyết định | Stack, naming, lint, test command |
| **Skill** (`.claude/skills/<name>/SKILL.md`) | Kiến thức theo ngữ cảnh, Claude tự kích hoạt | Pattern ngôn ngữ, quy trình, domain knowledge |
| **Agent** (`.claude/agents/<name>.md`) | Subagent chuyên task | Reviewer, builder, test runner |
| **Command** (`.claude/commands/<name>.md`) | User gõ `/x` để chạy | Workflow có trigger rõ ràng |
| **Hook** (`settings.json`) | Tự động theo sự kiện | Pre-commit, post-edit format |

**Nguyên tắc:** Nếu thông tin *"luôn đúng"* → **Rule**. Nếu cần điều kiện *"khi làm X thì dùng"* → **Skill**.

---

## 3. Quy trình 6 bước

### Bước 1 — Quét codebase (Claude tự làm)

Chạy song song:

- Đọc các manifest: `package.json`, `pyproject.toml`, `go.mod`, `pom.xml`, `build.gradle`, `Cargo.toml`
- Đọc config: `tsconfig.json`, `.eslintrc*`, `.prettierrc*`, `pytest.ini`, `Makefile`, `docker-compose*`, `.github/workflows/*`
- Glob top 2 levels thư mục, bỏ `node_modules`, `dist`, `.git`, `vendor`
- Đọc `README.md`, `CONTRIBUTING.md`, `CLAUDE.md` (nếu có)
- `git log --oneline -50` để xem commit style

**Output bước 1:** báo cáo ngắn gồm: tech stack, framework chính, folder layout, commit convention, test/lint command.

### Bước 2 — Viết `rules/<stack>.md`

Template:

```markdown
# <Tên stack> Rules cho <Tên dự án>

## Stack
- Runtime: <e.g., Node 20 / Python 3.11>
- Test runner: <e.g., pytest / vitest>
- Linter / Formatter: <e.g., ruff + black / eslint + prettier>
- Package manager: <e.g., pnpm / poetry>

## File Conventions
- `src/` — <mô tả>
- `tests/` — mirror `src/`, file test `*_test.py` / `*.test.ts`
- Naming: <lowercase-hyphen / camelCase / snake_case>

## Code Style
- <Rule 1 — 1 dòng, actionable>
- <Rule 2>
- Giới hạn độ dài file: <N dòng>
- Import: <relative / absolute, mixed / named>

## Testing Requirements
- Lệnh chạy: `<cmd>`
- Mọi PR phải chạy `<cmd>` trước khi merge
- Coverage tối thiểu: <%>

## Commit / PR
- Convention: <conventional / ticket-prefix>
- Branch naming: <feat/TICKET-123-desc>
```

**Quy tắc khi viết rule:**
- Mỗi rule 1 dòng, dạng *"Do X"* hoặc *"Don't Y"*
- Không viết *"X quan trọng"* — phải actionable
- Nếu rule cần giải thích dài → tách sang skill

### Bước 3 — Cài skill ngôn ngữ có sẵn

Trước khi viết mới, check repo `everything-claude-code` có skill sẵn không:
- `python-patterns`, `golang-patterns`, `rust-patterns`
- `springboot-patterns`, `nestjs-patterns`, `laravel-patterns`, `nuxt4-patterns`
- `swiftui-patterns`, `kotlin-patterns`, `perl-patterns`

Copy vào `.claude/skills/`, **chỉ sửa phần dự án mình khác đi**. Không viết lại từ đầu.

### Bước 4 — Sinh skill codebase từ code có sẵn

Claude đọc code và sinh **draft skill mô tả "code đang làm gì"**:

Template `skills/<ten-du-an>-architecture/SKILL.md`:

```markdown
---
name: <ten-du-an>-architecture
description: Kiến trúc và layout của <tên dự án>. Dùng khi thêm feature mới, refactor, hoặc onboard member mới.
origin: auto-generated
version: "0.1.0"
---

# <Tên dự án> — Architecture

## When to Use
- Thêm feature mới cần biết đặt file ở đâu
- Refactor cross-module
- Review PR đụng nhiều layer

## Tech Stack
<từ bước 1>

## Kiến trúc tổng thể
<monolith / microservice / modular monolith>
<vẽ sơ đồ ASCII đơn giản nếu cần>

## Bounded Contexts
| Context | Thư mục | Trách nhiệm |
|---|---|---|
| <e.g. billing> | `src/billing/` | <mô tả> |
| <e.g. auth> | `src/auth/` | <mô tả> |

## Data Flow (1 request)
<trace 1 API call từ route → service → repo → DB → response>

## Entry Points quan trọng
- `<path>` — <mô tả>
- `<path>` — <mô tả>

## Related Skills
- <domain>-business-rules
- <stack>-patterns
```

### Bước 5 — User viết skill NGHIỆP VỤ (Claude KHÔNG tự viết)

**Cực kỳ quan trọng:** code kể được WHAT, không kể được WHY. Claude **không được bịa** nghiệp vụ. Nếu thiếu thông tin → hỏi user.

Mỗi bounded context = 1 file. Template `skills/<context>-business-rules/SKILL.md`:

```markdown
---
name: <context>-business-rules
description: Quy tắc nghiệp vụ <context> của <tên dự án>. Dùng khi code/review feature liên quan <context>.
origin: hand-written
version: "1.0.0"
---

# <Context> — Business Rules

## When to Use
- Sửa file trong `src/<context>/`
- Thiết kế API mới liên quan <context>
- Review feature đụng <context>

## Ngôn ngữ nghiệp vụ (Glossary)
| Thuật ngữ | Nghĩa |
|---|---|
| <Customer> | <định nghĩa chính xác theo team> |
| <Order> | |
| <Invoice> | |

## Business Rules
### BR-001: <Tên rule>
- **Điều kiện:** <khi nào áp dụng>
- **Quy tắc:** <luật cụ thể>
- **Edge case:** <trường hợp ngoại lệ>
- **Liên quan code:** `src/...`

### BR-002: ...

## Invariants (luôn đúng)
- Một <Customer> chỉ có tối đa 1 <active subscription>
- <Order status> chỉ đi theo chiều: draft → submitted → paid → completed
- ...

## Lịch sử / Legacy
- Trước <ngày>, <rule cũ A>. Sau <ngày>, <rule mới B>. Code xử lý ở `<path>`.

## Anti-patterns
- ❌ Không gọi trực tiếp repo từ controller — phải qua service layer
- ❌ Không hardcode <magic value>
```

**Claude: nếu user chưa viết phần này, KHÔNG bịa. Hỏi:**

> "Để viết skill `<context>-business-rules`, tôi cần bạn xác nhận:
> 1. Các thuật ngữ nghiệp vụ quan trọng của <context>?
> 2. 3-5 rule nghiệp vụ bắt buộc?
> 3. Invariant nào luôn đúng?
> Tôi có thể *đoán* từ code, nhưng cần bạn xác nhận đúng/sai trước khi lưu."

### Bước 6 — Verify skill vs code (vòng lặp)

Sau khi có skill nghiệp vụ, Claude đọc skill + code và báo cáo:

```
## Skill-Code Compliance Report

### ✅ Khớp
- BR-001 — `src/order/service.py:42` thực thi đúng

### ⚠️  Lệch (cần quyết định)
- BR-003 nói "<X>" nhưng `src/billing.py:87` đang làm "<Y>"
  → Sửa code? Sửa skill? Bỏ rule?

### ❓ Không tìm thấy code thực thi
- BR-005 — không có code nào match. Rule có còn active không?
```

User quyết định từng mục → cập nhật skill hoặc code.

---

## 4. Checklist trước khi commit skill

- [ ] Frontmatter có `name`, `description` cụ thể (không chung chung)
- [ ] `description` đủ gợi ý để Claude tự activate đúng lúc
- [ ] Section `When to Use` có ít nhất 3 ví dụ cụ thể
- [ ] Có ít nhất 1 code example thật từ dự án
- [ ] Không trùng với rule đã có
- [ ] Không chứa secret / data khách hàng
- [ ] Đã chạy markdownlint

---

## 5. Cấu trúc thư mục cuối cùng

```
<project-root>/
├── CLAUDE.md                      # Entry point cho Claude
├── .claude/
│   ├── rules/
│   │   ├── <stack>.md             # Bước 2
│   │   └── common.md              # Rule chung (git, security)
│   ├── skills/
│   │   ├── <project>-architecture/SKILL.md    # Bước 4
│   │   ├── billing-business-rules/SKILL.md    # Bước 5
│   │   ├── auth-business-rules/SKILL.md       # Bước 5
│   │   └── <context>-business-rules/SKILL.md
│   ├── agents/
│   │   └── <project>-reviewer.md
│   └── commands/
│       └── <project>-feature.md
```

---

## 6. Quy tắc dành riêng cho Claude khi viết skill

1. **Không bịa nghiệp vụ.** Nếu không có thông tin từ user hoặc doc → hỏi, hoặc ghi `TODO: cần user xác nhận`.
2. **Ưu tiên ngắn gọn.** Skill quá dài → tách thành nhiều skill nhỏ theo context.
3. **Dùng file path thật** của dự án trong ví dụ, không dùng placeholder.
4. **Check trùng lặp** trước khi tạo skill mới — đọc hết tên skill đang có trong `.claude/skills/`.
5. **Viết tiếng Việt** cho phần nghiệp vụ (để team đọc), giữ **tiếng Anh** cho tên field, code example, thuật ngữ kỹ thuật.
6. **Mỗi lần cập nhật skill** → bump `version` trong frontmatter.
7. **Khi user nói "viết skill X"** — làm theo thứ tự: (a) check đã có chưa, (b) quét code liên quan, (c) hỏi phần nghiệp vụ, (d) viết draft, (e) verify vs code, (f) chốt với user.

---

## 7. Lệnh khởi động (user copy-paste cho Claude)

```
Đọc file HUONG-DAN-VIET-SKILL.md.
Điền phần "Bối cảnh dự án" ở mục 0 bằng cách quét codebase (Bước 1).
Sau đó đề xuất danh sách rule + skill nên viết cho dự án này,
sắp xếp theo độ ưu tiên. KHÔNG viết skill ngay —
chờ tôi duyệt danh sách trước.
```
