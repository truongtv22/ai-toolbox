# Phase 01: Backend Session Manager Multi-root & Tilde Expansion

> [!CAUTION]
> **RÀNG BUỘC (CRITICAL CONSTRAINT):**
> **TUYỆT ĐỐI KHÔNG CHẠY CARGO** (`cargo build`, `cargo check`, `cargo test`, `cargo clippy`). Mọi thay đổi trong Rust phải được tự rà soát cẩn thận, đối chiếu kiểu dữ liệu (types), import đầy đủ và kiểm tra cú pháp bằng mắt.

## Context Links
- Main Plan: [plan.md](./plan.md)
- Reference pattern: [`tauri/src/coding/proxy_gateway/session_import.rs`](file:///Volumes/Media/tools/ai-toolbox/tauri/src/coding/proxy_gateway/session_import.rs#L766-L778)
- Target file 1: [`tauri/src/coding/session_manager/antigravity.rs`](file:///Volumes/Media/tools/ai-toolbox/tauri/src/coding/session_manager/antigravity.rs)
- Target file 2: [`tauri/src/coding/session_manager/mod.rs`](file:///Volumes/Media/tools/ai-toolbox/tauri/src/coding/session_manager/mod.rs)
- Target file 3: [`tauri/src/coding/runtime_location.rs`](file:///Volumes/Media/tools/ai-toolbox/tauri/src/coding/runtime_location.rs)

## Overview
- **Priority:** P1
- **Status:** Completed
- **Mục tiêu:** Cho phép `session_manager` tự động quét cả 4 profile Antigravity (`antigravity-cli`, `antigravity`, `antigravity-ide`, `antigravity-acp`), deduplicate theo UUID (giữ bản có `last_modified` mới nhất), đọc `cwd` từ file `.meta` của ACP, gắn nhãn nguồn `runtime_source` (`"cli"`, `"extension"`, `"ide"`, `"acp"`), và mở rộng dấu ngã `~` khi người dùng nhập custom root.

## Yêu Cầu Kỹ Thuật (Requirements)

1. **Tilde Expansion (`runtime_location.rs`):**
   - Khi `get_custom_path_from_record` trả về chuỗi đường dẫn tùy chỉnh (ví dụ `~/.gemini/antigravity`), dùng hàm tiện ích mở rộng dấu `~` thay vì để nguyên chuỗi bắt đầu bằng `~`.
   - Có thể dùng `expand_home_from_user_root(None, &path)` hoặc logic `dirs::home_dir()` có sẵn trong module.

2. **Multi-root Resolution (`session_manager/mod.rs`):**
   - `ToolSessionContext::Antigravity` hiện chỉ chứa `cli_root: PathBuf`.
   - Cập nhật thành:
     ```rust
     ToolSessionContext::Antigravity {
         roots: Vec<PathBuf>,
     }
     ```
   - Trong `resolve_context`, thu thập danh sách `roots`:
     * Thư mục cấu hình chỉ định (nếu có custom root trong DB).
     * Các thư mục mặc định dưới `$HOME/.gemini/`:
       - `antigravity` (VS Code Extension & App)
       - `antigravity-cli` (CLI)
       - `antigravity-ide` (Standalone IDE)
       - `antigravity-acp` (Codeg / ACP server)
     * Loại bỏ các thư mục trùng lặp và chỉ giữ các thư mục thực sự tồn tại (`path.is_dir()`).

3. **Multi-root Scan & Deduplication (`session_manager/antigravity.rs`):**
   - Cập nhật `scan_sessions(roots: &[PathBuf]) -> Vec<SessionMeta>`:
     * Quét tất cả file `conversations/*.db` trong từng root.
     * Sử dụng `HashMap<String, (u64, PathBuf, &str)>` để map `uuid -> (modified_nanos, db_path, source_type)`:
       - Nếu folder là `antigravity-acp` -> `source_type = "acp"`
       - Nếu folder là `antigravity-ide` -> `source_type = "ide"`
       - Nếu folder là `antigravity` -> `source_type = "extension"`
       - Nếu folder là `antigravity-cli` -> `source_type = "cli"`
     * Khi trùng UUID, ưu tiên bản ghi có `last_modified` lớn hơn.
   - Hỗ trợ file `<uuid>.meta`:
     * Nếu cạnh `<uuid>.db` có file `<uuid>.meta`, đọc JSON để lấy `cwd` và gán vào `project_dir`.
   - Gán `runtime_source: Some(source_type.to_string())` vào `SessionMeta`.

4. **Đồng bộ với `proxy_gateway` (`proxy_gateway/session_import.rs`):**
   - Thêm `"antigravity-acp"` vào mảng profile:
     ```rust
     for profile in ["antigravity-cli", "antigravity", "antigravity-ide", "antigravity-acp"]
     ```
   - Giúp Gateway tính toán chính xác lượng token của các session chạy từ `codeg`.

## Todo List
- [x] Bổ sung tilde expansion trong `runtime_location.rs` cho `resolve_antigravity_runtime_location_uncached_async`.
- [x] Cập nhật `ToolSessionContext::Antigravity` trong `session_manager/mod.rs` để hỗ trợ `roots: Vec<PathBuf>`.
- [x] Cập nhật `scan_sessions` và `scan_recent_sessions` trong `session_manager/antigravity.rs` nhận `roots: &[PathBuf]`.
- [x] Bổ sung helper đọc file `<uuid>.meta` để trích xuất `cwd` làm `project_dir`.
- [x] Gán `runtime_source` cho `SessionMeta`.
- [x] Bổ sung `"antigravity-acp"` vào `proxy_gateway/session_import.rs`.

## Validation (Không chạy Cargo)
- Tự rà soát import, syntax, lifetimes trong code Rust.
- Kiểm tra các caller của `scan_sessions` trong `session_manager/mod.rs`.

