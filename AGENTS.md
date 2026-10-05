# AGENTS.md — Router (Autotest Template)

Entry point mọi session. Phân loại intent rồi route.

## Intent → Route

| User nói | Route |
|---|---|
| "sinh test cho module X / theo spec" | **`/autotest`** — test-first (Nhánh A) |
| "khóa behavior / test legacy / refactor an toàn" | **`/characterize <path>`** — Nhánh B |
| "case test tay xong, viết auto test" | **`/capture-manual`** — Nhánh C |
| "kiểm tra chất lượng test / verify trước merge" | **`/verify-tests`** |
| "test fail vì sao / lỗi này gặp lại" | error-analyzer → `.context/error-memory.md` |
| "code tiếp feature theo spec" | như template dev: spec → test-writer → code → verify |
| review/check | reviewer (nếu dự án có) |

## Mặc định khi session bắt đầu
1. Đọc `SPECIFICATIONS.md` (nếu có) — nguồn truth
2. Kiểm tra test có sẵn chạy pass không (`npm test` / `pytest`)
3. Check `.context/error-memory.md` — lỗi đã gặp để tránh lặp
4. Check `.context/manual-cases/` — case manual chưa chuyển auto

## Quy tắc nền
- Mọi yêu cầu test → kiểm tra traceability về spec (`test-validator`)
- Mọi fail → Error Analyzer → error-memory (match template dev)
- Không merge nếu chưa `/verify-tests` pass