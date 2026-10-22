# Phase 04: Frontend Source Tagging & Tinh Chỉnh Giao Diện

## Context Links
- Main Plan: [plan.md](./plan.md)
- Target file 1: [`web/features/coding/shared/sessionManager/SessionList.tsx`](file:///Volumes/Media/tools/ai-toolbox/web/features/coding/shared/sessionManager/SessionList.tsx)
- Target file 2: [`web/i18n/locales/en-US.json`](file:///Volumes/Media/tools/ai-toolbox/web/i18n/locales/en-US.json)
- Target file 3: [`web/i18n/locales/zh-CN.json`](file:///Volumes/Media/tools/ai-toolbox/web/i18n/locales/zh-CN.json)
- Target file 4: [`web/features/coding/antigravity/pages/AntigravityPage.tsx`](file:///Volumes/Media/tools/ai-toolbox/web/features/coding/antigravity/pages/AntigravityPage.tsx)

## Overview
- **Priority:** P2
- **Status:** Completed
- **Quyết định đã thống nhất:**
  1. Đổi tên tab trên Sidebar thành **"Antigravity"** (thay vì "Antigravity CLI").
  2. Gắn nhãn badge nguồn cho `antigravity-acp` là **"ACP"** (chuẩn kỹ thuật Agent Client Protocol).

## Chi Tiết Kỹ Thuật (Technical Details)

### 1. Hiển thị Badge nguồn trong `SessionList.tsx`:
* `session.runtimeSource` sẽ nhận các giá trị và hiển thị badge:
  * `"cli"` -> Badge: **CLI**
  * `"extension"` -> Badge: **Extension**
  * `"ide"` -> Badge: **IDE**
  * `"acp"` -> Badge: **ACP**
* Đặt badge này ở vị trí thích hợp cạnh timestamp hoặc project directory trong từng item của Session List.

### 2. Tinh chỉnh Header & Đặt tên tab:
* Cập nhật title trong i18n (`web/i18n/locales/en-US.json` và `zh-CN.json`):
  * Đổi `"Antigravity CLI"` thành **`"Antigravity"`** ở những vị trí đại diện cho toàn bộ module (sidebar tab, tiêu đề trang).
  * Giữ nguyên tính nhất quán trong các chuỗi liên quan đến OAuth CLI.

### 3. Bộ lọc nhanh (Quick Filter - Tùy chọn):
* Cho phép người dùng lọc danh sách hội thoại Antigravity theo nguồn (`All`, `Extension`, `ACP`, `CLI`, `IDE`) nếu cần.

## Todo List
- [x] Cập nhật `SessionList.tsx` để render badge nguồn (`CLI`, `Extension`, `IDE`, `ACP`).
- [x] Thêm styles cho các badge nguồn phù hợp với design token trong `DESIGN.md`.
- [x] Cập nhật nhãn i18n trong `en-US.json` và `zh-CN.json` (đổi tên tab sang "Antigravity", thêm nhãn badge).
- [x] Chạy `pnpm test` và `pnpm tsc --noEmit` để nghiệm thu.

## Validation
- Mở danh sách session Antigravity, kiểm tra các session từ `antigravity-acp` hiển thị badge `ACP`, từ `antigravity` hiển thị badge `Extension`, từ `antigravity-ide` hiển thị badge `IDE`.
