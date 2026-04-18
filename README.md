# skill-for-claude

Bộ hướng dẫn giúp Claude (Claude Code) viết **rules + skills** cho dự án lập trình trong doanh nghiệp.

## Nội dung

- [`HUONG-DAN-VIET-SKILL.md`](./HUONG-DAN-VIET-SKILL.md) — Prompt nền đưa cho Claude ở công ty. Hướng dẫn 6 bước: quét codebase → viết rule stack → cài skill ngôn ngữ → sinh skill kiến trúc → viết skill nghiệp vụ → verify skill vs code.

## Cách dùng

1. Copy file `HUONG-DAN-VIET-SKILL.md` vào root dự án của bạn.
2. Mở Claude Code tại thư mục dự án.
3. Điền phần **Bối cảnh dự án** (mục 0) trong file.
4. Gõ prompt khởi động ở mục 7 của file cho Claude.

## Tham khảo

Dựa trên pattern từ [affaan-m/everything-claude-code](https://github.com/affaan-m/everything-claude-code) — 183 skill sản xuất sẵn, đạt giải Anthropic Hackathon.
