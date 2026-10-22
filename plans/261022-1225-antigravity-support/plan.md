---
title: "Nâng cấp Antigravity Session Manager (Hỗ trợ CLI, IDE, VS Code Extension, ACP/Codeg)"
description: "Mở rộng Session Manager để quét đa profile Antigravity, khắc phục hiển thị tool read_file/run_command và gắn nhãn nguồn session"
status: completed
priority: P1
effort: 6h
branch: main
tags: [antigravity, backend, frontend, session-manager, bugfix]
created: 2026-10-22
---

# Kế Hoạch Triển Khai: Nâng Cấp Antigravity Session Manager

> [!CAUTION]
> **RÀNG BUỘC BẮT BUỘC (CRITICAL CONSTRAINT):**
> **TUYỆT ĐỐI KHÔNG CHẠY CARGO** (`cargo build`, `cargo check`, `cargo test`, `cargo clippy`) trong toàn bộ quá trình phát triển và kiểm thử để tránh sinh thư mục `tauri/target/` nặng nhiều GB làm đầy ổ đĩa.
> Kiểm tra cú pháp và logic Rust bằng code inspection, so sánh với pattern hiện có (`proxy_gateway/session_import.rs`), và kiểm thử frontend qua `pnpm test` / `pnpm tsc --noEmit`.

## Tổng quan (Overview)

Nâng cấp phân hệ Antigravity trong AI Toolbox để hỗ trợ đầy đủ toàn bộ hệ sinh thái Google Antigravity:
1. **Quét đa nguồn (Multi-root Scanning):** Tự động phát hiện và tổng hợp các phiên trò chuyện từ **Antigravity CLI** (`~/.gemini/antigravity-cli`), **Extension / App** (`~/.gemini/antigravity`), **Antigravity IDE** (`~/.gemini/antigravity-ide`), và **Antigravity ACP (`codeg`)** (`~/.gemini/antigravity-acp`).
2. **Khắc phục lỗi Tilde Expansion:** Hỗ trợ nhập `~/.gemini/...` khi tùy chỉnh thư mục mà không bắt buộc gõ đường dẫn tuyệt đối.
3. **Sửa lỗi hiển thị Tool Calls:** Hỗ trợ schema PascalCase (`Path`, `CommandLine`, `Content`...) và các tool ACP (`view_file`, `client_create_file`...).
4. **Sửa lỗi dồn dòng Tool Output:** Ghép nối bản ghi kết quả tool (`GENERIC`, `AGENCY_TOOL_CALL`, `RUN_COMMAND`...) vào `tool_call` liền trước để hiển thị trong khối `<pre><code>` có thanh cuộn và giữ nguyên từng dòng.
5. **Gắn nhãn nguồn (Source Badges):** Hiển thị rõ nguồn của từng cuộc hội thoại (`CLI`, `Extension`, `IDE`, `ACP`).

## Các Giai Đoạn (Phases)

| # | Giai đoạn | Trạng thái | Ước tính | Tài liệu chi tiết |
|---|-----------|------------|----------|-------------------|
| 1 | Backend: Quét đa thư mục & Tilde Expansion | Completed | 1.5h | [phase-01](./phase-01-backend-session-manager-multiroot.md) |
| 2 | Backend: Ghép nối Tool Result & Khắc phục dồn dòng | Completed | 1.5h | [phase-02](./phase-02-backend-tool-result-pairing.md) |
| 3 | Frontend: Hiển thị Tool Call (PascalCase & ACP Tools) | Completed | 1.5h | [phase-03](./phase-03-frontend-tool-display.md) |
| 4 | Frontend: Gắn nhãn nguồn & Tinh chỉnh giao diện | Completed | 1.5h | [phase-04](./phase-04-source-tagging-ui.md) |

## Dependencies & Affected Files

- `tauri/src/coding/session_manager/antigravity.rs` (Scan multi-root, deduplicate, parse transcript & tool pairing)
- `tauri/src/coding/session_manager/mod.rs` (Context resolution & multi-root context)
- `tauri/src/coding/runtime_location.rs` (Tilde expansion `~` cho custom path)
- `tauri/src/coding/proxy_gateway/session_import.rs` (Thêm `antigravity-acp` vào danh sách quét usage)
- `web/features/coding/shared/sessionManager/detail/SessionToolExecutionCard.tsx` (PascalCase args & ACP fields)
- `web/features/coding/shared/sessionManager/detail/domain/toolCatalog.ts` (Thêm tool `client_*`, `view_file`, `manage_task`...)
- `web/features/coding/shared/sessionManager/SessionList.tsx` (Badge hiển thị nguồn `CLI` / `VS Code` / `IDE` / `Codeg`)
- `web/i18n/locales/en-US.json` & `zh-CN.json` (i18n nhãn nguồn)

