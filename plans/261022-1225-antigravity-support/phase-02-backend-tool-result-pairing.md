# Phase 02: Backend Tool Result Pairing & Khắc Phục Dồn Dòng

> [!CAUTION]
> **RÀNG BUỘC (CRITICAL CONSTRAINT):**
> **TUYỆT ĐỐI KHÔNG CHẠY CARGO** (`cargo build`, `cargo check`, `cargo test`, `cargo clippy`).

## Context Links
- Main Plan: [plan.md](./plan.md)
- Target file: [`tauri/src/coding/session_manager/antigravity.rs`](file:///Volumes/Media/tools/ai-toolbox/tauri/src/coding/session_manager/antigravity.rs)

## Overview
- **Priority:** P1
- **Status:** Completed
- **Mục tiêu:** Trong log transcript của Antigravity, kết quả gọi tool nằm ở bản ghi tiếp theo (`type: "GENERIC"`, `"AGENCY_TOOL_CALL"`, `"RUN_COMMAND"`, `"SEARCH_WEB"`, `"READ_URL_CONTENT"`). Hiện tại parser coi bản ghi này là một tin nhắn văn bản thông thường của Assistant, dẫn đến việc Markdown renderer gom các dòng `\n` thành một dòng dài. Giai đoạn này sẽ ghép nối kết quả đó trực tiếp vào `tool_calls` liền trước để tạo thành `tool_execution` có `output`.

## Chi Tiết Kỹ Thuật (Technical Details)

### 1. Cơ chế transcript của Antigravity:
* **Record N** (`type: "PLANNER_RESPONSE"`):
  ```json
  {
    "step_index": 1,
    "source": "MODEL",
    "type": "PLANNER_RESPONSE",
    "tool_calls": [
      {
        "name": "read_file",
        "args": { "Path": "/path/to/file" }
      }
    ]
  }
  ```
* **Record N+1** (kết quả trả về của tool):
  ```json
  {
    "step_index": 2,
    "source": "MODEL",
    "type": "GENERIC",  // hoặc AGENCY_TOOL_CALL, RUN_COMMAND, SEARCH_WEB...
    "content": "Created At: 2026-10-22...\n1: # File content line 1\n2: # File content line 2"
  }
  ```

### 2. Giải pháp Parser trong `session_manager/antigravity.rs`:
* Hiện tại hàm `load_messages` gọi `message_from_record` độc lập cho từng record.
* Nâng cấp `load_messages` thành thuật toán duyệt có trạng thái (stateful pass hoặc lookahead):
  1. Duyệt danh sách các records đã được sort theo `step_index`.
  2. Nếu Record $i$ có `tool_calls`:
     - Kiểm tra Record $i+1$: Nếu Record $i+1$ có `type` thuộc nhóm kết quả tool:
       `["GENERIC", "AGENCY_TOOL_CALL", "RUN_COMMAND", "SEARCH_WEB", "READ_URL_CONTENT"]`
       và chứa `content`:
       * Lấy nội dung `content` này gán làm `output` cho tool call trong Record $i$.
       * Đồng thời đánh dấu đã tiêu thụ Record $i+1$ (consume / skip), không biến Record $i+1$ thành một tin nhắn chat Assistant độc lập nữa.
  3. Khi frontend nhận được `tool_call` đã có sẵn `output`:
     - Component `SessionToolExecutionCard` sẽ render khối `<SessionToolResultBlock output={...} />`.
     - Toàn bộ nội dung kết quả sẽ nằm trong `<pre className={styles.preBlock}>` với thuộc tính CSS `white-space: pre-wrap`.
     - **Kết quả:** Các dòng `1: ...\n2: ...` giữ nguyên từng dòng, có thanh cuộn và số dòng rõ ràng, không bị dồn thành 1 dòng nữa!

### 3. Hỗ trợ nhiều tool call trong 1 step:
* Nếu Record $i$ có nhiều hơn 1 tool call trong mảng `tool_calls`, gán kết quả cho tool call tương ứng (hoặc tool call đầu tiên nếu chỉ có 1 record output).

## Todo List
- [x] Xác định danh sách các type đại diện cho kết quả tool: `GENERIC`, `AGENCY_TOOL_CALL`, `RUN_COMMAND`, `SEARCH_WEB`, `READ_URL_CONTENT`.
- [x] Viết lại vòng lặp duyệt records trong `load_messages` (`session_manager/antigravity.rs`) để ghép nối Record kết quả vào `tool_calls` liền trước.
- [x] Bỏ qua (skip) không tạo tin nhắn text bubble riêng cho các record kết quả đã được ghép vào tool call.
- [x] Đảm bảo nếu record kết quả có chứa lỗi (`error` / status), gán cờ `is_error` cho tool block.

## Validation (Không chạy Cargo)
- Tự rà soát logic bằng mắt: kiểm tra mảng index bounds, check option handling.

