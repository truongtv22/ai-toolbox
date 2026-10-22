# Phase 03: Frontend Tool Execution Display (PascalCase & ACP Tools)

## Context Links
- Main Plan: [plan.md](./plan.md)
- Target file 1: [`web/features/coding/shared/sessionManager/detail/SessionToolExecutionCard.tsx`](file:///Volumes/Media/tools/ai-toolbox/web/features/coding/shared/sessionManager/detail/SessionToolExecutionCard.tsx)
- Target file 2: [`web/features/coding/shared/sessionManager/detail/domain/toolCatalog.ts`](file:///Volumes/Media/tools/ai-toolbox/web/features/coding/shared/sessionManager/detail/domain/toolCatalog.ts)

## Overview
- **Priority:** P1
- **Status:** Completed
- **Mục tiêu:** Bổ sung hỗ trợ cho các tham số dạng PascalCase của Google Antigravity (`Path`, `CommandLine`, `Content`, `Target`, `Replacement`) và các tool đặc thù của `antigravity-acp` (`view_file`, `client_create_file`, `client_view_file`, `client_edit_file`, `read_url_content`, `manage_task`, `schedule`), đồng thời hiển thị `toolAction` / `toolSummary`.

## Chi Tiết Kỹ Thuật (Technical Details)

### 1. Chuẩn hóa tên tool trong `toolCatalog.ts`:
* Thêm mapping cho các tool của Antigravity & ACP:
  * `read`: Thêm `['view_file', 'client_view_file']`
  * `write`: Thêm `['client_create_file']`
  * `edit`: Thêm `['client_edit_file']`
  * `web_fetch`: Thêm `['read_url_content']`
  * `task`: Thêm `['manage_task', 'schedule']`

### 2. Cập nhật trích xuất tham số trong `SessionToolExecutionCard.tsx`:
* **Trường Path:**
  ```tsx
  stringField(input, ['Path', 'absolute_path', 'AbsolutePath', 'target_file', 'file_path', 'filePath', 'path'])
  ```
  * Xử lý trường hợp chuỗi bị bọc trong dấu ngoặc kép lặp lại (`"\"/path/...\""`): dùng regex hoặc `.replace(/^"|"$/g, '')` để làm sạch.
* **Trường Command trong `BashBody`:**
  ```tsx
  stringField(input, ['CommandLine', 'command', 'cmd'])
  ```
* **Trường Content trong `WriteBody` & `EditBody`:**
  ```tsx
  stringField(input, ['Content', 'code_content', 'content'])
  ```
* **Trường Target / Replacement trong `EditBody`:**
  * Nếu input có `Target` và `Replacement`: hiển thị diff block hoặc code panels tương ứng.
* **Trường Mô tả (Description / Action Summary):**
  * Hiển thị mô tả ngắn từ các trường của `antigravity-acp`:
    ```tsx
    stringField(input, ['toolAction', 'toolSummary', 'description'])
    ```
  * Thẻ tool sẽ hiển thị tiêu đề thân thiện: ví dụ *"Checking git status"* hoặc *"View code-review skill"*.

## Todo List
- [x] Bổ sung tên tool Antigravity/ACP vào `getNormalizedToolName` trong `toolCatalog.ts`.
- [x] Cập nhật `stringField` và các component con (`ReadBody`, `WriteBody`, `EditBody`, `BashBody`, `GrepBody`, `GlobBody`) trong `SessionToolExecutionCard.tsx`.
- [x] Làm sạch chuỗi quotes thừa nếu tham số bị bọc trong `\"...\"`.
- [x] Thêm hiển thị `toolAction` / `toolSummary` trên header hoặc description của thẻ tool.
- [x] Chạy `pnpm test` để kiểm tra không làm hỏng các test suite frontend hiện có.

## Validation
- Chạy `pnpm tsc --noEmit` để đảm bảo type checking hoàn toàn hợp lệ.
- Chạy `pnpm test` trên các unit test frontend.

