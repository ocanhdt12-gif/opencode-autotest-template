---
name: test-quality-gate
description: "Gate chặn test 'vô hại' — expected lấy từ chạy code, try-catch nuốt lỗi, assert trivially, cached code cũ; kèm hidden stash test ẩn chống agent overfit. Dùng khi /verify-tests, review test do AI sinh, hoặc nghi test không bắt được bug."
---

# Test Quality Gate (In-house)

> Từ bài học bottleneck vibe coding (ACM/GitClear/Exadel 2026) + OpenReplay "Testing in the AI Era" (09/2026): test agent thường "xác nhận code chứ không kiểm tra code" — expected lấy từ chạy code → **lock bug vào test**; test không có cách bất đồng với code.

## Các dạng test VÔ HẠI (phải chặn)

| Dạng | Dấu hiệu | Xử lý |
|---|---|---|
| Expected lấy từ chạy code | Test được viết SAU khi code pass, expected = output code hiện tại | Expected phải từ SPEC (nhánh A) hoặc từ golden đã scrub (nhánh B). Nếu chỉ để "confirm what code does" → FAIL |
| Try-catch nuốt lỗi | `try { ... } catch { assert(true) }` | FAIL — nuốt fail là mất ý nghĩa |
| Assert trivially | Chỉ assert "không throw", "trả về object", type check | FAIL — phải assert behavior cụ thể |
| Test không bao giờ red | Test contain code đã pass mà không có input bất thường | Soi kỹ: đổi behavior nhỏ → test có đổi không? (mutation) |
| Cache code cũ | Agent "nhớ" expected cũ (đã sửa) không cập nhật | Đối chiếu spec hiện tại |

## Hidden Stash (chống overfit — verified từ nghiên cứu)

Agent overfit test thấy được trong prompt. Giữ **1 bộ test ẩn**:
- Nằm trong `tests/hidden/` — KHÔNG đưa vào prompt agent lúc viết code
- Chạy trong CI (`verify-tests`) — agent không biết nội dung
- Test các case khó / regression quan trọng đã từng vỡ

## Kiểm tra nhanh 1 test

> Đổi 1 dòng behavior của code (vd `>` thành `>=`) → test có fail không? Không → test yếu.

## Gate (trong /verify-tests)

- [ ] Không test "xác nhận bug" (expected từ chạy code khi code đã sai)
- [ ] Không try-catch nuốt lỗi
- [ ] Không assert trivially
- [ ] Hidden stash chạy + pass
- [ ] Mutation score ≥ ngưỡng