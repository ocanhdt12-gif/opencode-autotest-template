# AGENTS.md — Router (Autotest Template)

Entry point mọi session. Phân loại intent rồi route.

## Intent → Route

| User nói | Route |
|---|---|
| "sinh test cho module X / theo spec" | **`/autotest`** — test-first (Nhánh A) |
| "khóa behavior / test legacy / refactor an toàn" | **`/characterize <path>`** — Nhánh B |
| "case test tay xong, viết auto test" | **`/capture-manual`** — Nhánh C |
| "kiểm tra chất lượng test / verify trước merge" | **`/verify-tests`** |
| "test fail vì sao / lỗi này gặp lại" | test-reflector phân loại → sửa test hoặc báo loop agent |
| "code tiếp feature theo spec" | như template dev: spec → test-writer → code → verify |
| review/check | reviewer (nếu dự án có) |

## Mặc định khi session bắt đầu
1. Đọc `SPECIFICATIONS.md` (nếu có) — nguồn truth
2. Kiểm tra test có sẵn chạy pass không (`npm test` / `pytest`)
3. Kiểm tra test suite hiện tại chạy pass không (`npm test` / `pytest`)
4. Check `.context/manual-cases/` — case manual chưa chuyển auto

## Quy tắc nền
- Mọi yêu cầu test → kiểm tra traceability về spec (`test-validator`)
- Mọi fail → test-reflector phân loại → sửa đúng chỗ
- Không merge nếu chưa `/verify-tests` pass