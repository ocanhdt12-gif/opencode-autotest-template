# AGENTS.md — Router (Autotest Template)

Entry point mọi session. Phân loại intent rồi route.

## 5 luồng test (xem `docs/FLOWS.md`)

| User nói | Route |
|---|---|
| "code xong rồi, test hết đi" | **`/autotest --full`** — luồng 1 (toàn bộ spec) |
| "vừa fix/update, test phần này" | **`/test-scope`** — luồng 2 (đọc test-scope.json từ dev) |
| "test lại luồng cũ / có vỡ không" | **`/regression`** — luồng 3 |
| "case này test tay xong rồi" | **`/capture-manual`** — luồng 4 |
| "test theo case anh/khách đưa" | **`/from-cases`** — luồng 5 |
| "khóa behavior code cũ / refactor an toàn" | **`/characterize <path>`** — nhánh B |
| "kiểm tra chất lượng test trước merge" | **`/verify-tests`** |
| "đã test đến đâu / còn gì chưa test" | **`/coverage`** — board độ phủ `.context/coverage.json` (test tự lưu) |
| "test fail vì sao" | test-reflector phân loại → sửa test hoặc báo loop agent |
| review/check | reviewer (nếu dự án có) |

## Mặc định khi session bắt đầu
1. `/spec-link --sync` — pull spec mới nhất về `.spec-cache/` (nếu chưa link → hỏi link)
2. Đọc `.spec-cache/SPECIFICATIONS.md` — nguồn truth
3. Đọc `.spec-cache/SPECIFICATIONS.md` + `.spec-cache/spec/test-scope/current.json` — spec + scope mới từ dev; đối chiếu board `.context/coverage.json` (req nào chưa test)
4. Chạy test suite hiện tại xem pass không (`npm test` / `pytest`)
5. Check `.context/manual-cases/` — case manual chưa chuyển auto

## Quy tắc nền
- Spec = link git tới `.spec-cache/` (KHÔNG lưu bản riêng — tránh lệch)
- Mọi test → traceability về spec (`test-validator`) hoặc case user/manual
- Test-scope là hợp đồng từ dev; thiếu → hỏi, không tự đoán rộng
- Mọi fail → test-reflector phân loại → sửa đúng chỗ
- Không merge nếu chưa `/verify-tests` pass