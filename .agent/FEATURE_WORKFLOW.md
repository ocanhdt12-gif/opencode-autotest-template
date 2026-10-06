# FEATURE_WORKFLOW — Autotest Template (test-case-first)

> ⚠️ **Maintenance-mode override:** state dùng `features[]`/`bugs[]`; **cấm push thẳng `forbidden_branch`** (mặc định `main`); branch/push model: staging-direct (cấu hình trong `PROJECT_PROFILE.md`).

## Luật trục — TEST-CASE-FIRST
**SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.** Mọi test phải follow test case (`skills/test-case-first`). Test case gom theo module ở `.context/test-cases/`; trạng thái task ở `.context/test-tasks.json`.

## Workflow chính — `/autotest` (test tính năng MỚI)

**Lần đầu:**
1. **Config spec** — chưa link → `/spec-link <git-url>`
2. **Config web URL** — chưa có `web_app_url` (`.agent/PROJECT_PROFILE.md`) → hỏi **link deploy web** (baseURL browser) + điền
3. **Đọc spec + overview** — spec version, modules, `R-xx`, phần chưa test
4. **Brainstorm câu hỏi** — spec mơ hồ → hỏi user
4. **Tạo test case** (draft, gom theo module) — `test-case-author`
5. ⛔ **User check & update** → chốt `approved` (HUMAN CHECKPOINT, không được bỏ)
6. **Sinh test code theo test case** → chạy **NGẦM (headless)** → log bug ra file
7. **Chạy BROWSER TỪNG CASE** (headed, bung hẳn, giữ mở) — gom theo module, xong 1 case chờ user chọn case tiếp
8. **Cập nhật trạng thái task** → không test lại cái đã test

**Các lần sau:** đọc spec → lấy task **chưa test** → tạo test case → chạy bước 4–8.

## Workflow `/retest` (chạy LẠI test đã có)
`/retest --all` (toàn hệ thống) · `/retest <module|feature>` (1 cụm) · `/retest <TC-xx>` (1 case) — chạy ngầm → browser (headed, giữ mở) → cập nhật trạng thái. **Không tạo test mới.**

## Workflow characterization (legacy chưa test)
1. `/characterize <path>` → characterization-writer: snapshot + scrub unstable + coverage bổ sung
2. **Mutation verify** bắt buộc (phá code → test fail)
3. Refactor/sửa an toàn trên nền golden

## Workflow manual capture
1. Case test tay xong → mô tả vào `.context/manual-cases/<id>.md`
2. `/capture-manual <id>` → viết test auto → XANH → đánh dấu `converted-to-auto`
3. Không tự động hóa được → `manual-only` + lý do

## Gates (bắt buộc)

| Gate | Khi nào | Fail khi |
|---|---|---|
| spec-validator | trước khi sinh test case | requirement thiếu/mâu thuẫn |
| test case `approved` | trước khi sinh test code/chạy | còn `draft` mà đã test |
| mọi test gắn `TC-xx` | luôn | test ngoài test case |
| test ĐỎ đúng cách | trước khi code | test xanh khi chưa code |
| mutation verify | nhánh B | test không bắt mutation |
| mutation score ≥ floor | `/verify-tests` | survivors không giải thích được |
| quality gate | `/verify-tests` | expected từ code, try-catch nuốt, assert trivially |
| hidden stash | `/verify-tests` + CI | test ẩn fail |
| browser headed + từng case + giữ mở | `/autotest` bước D | chạy headless / tự đóng browser |
| cập nhật trạng thái sau khi chạy | bước E | test lại cái đã test |

## Lỗi test (xử lý nội bộ)
Test fail → test-reflector phân loại: `bug-in-test` (test sai → sửa test) · `bug-in-code` (test đúng code sai → báo loop agent fix theo spec). Ghi chú ngắn vào `.context/test-notes.md` nếu đáng nhớ.
