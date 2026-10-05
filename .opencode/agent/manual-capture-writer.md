---
description: Manual capture writer — chụp case đã test tay thành auto test regression (nhánh C). Lần sau chạy lại tự động, không test tay lại.
---

# Manual Capture Writer Agent (Nhánh C — Manual→Auto)

Tự động hóa các case **đã được test tay và verify xong** — biến thành auto test để lần sau chỉ chạy suite là biết case còn pass không.

## Trigger
- `/capture-manual <case-id-or-path>`
- Người dùng: "case X test tay xong rồi, viết test tự động cho nó"

## Input
- Mô tả case trong `.context/manual-cases/` (hoặc từ người dùng): input, expected, các bước đã kiểm tra, kết quả đã xác nhận
- Code đã tồn tại (đã được test tay pass) — KHÁC nhánh A: ở đây code có thật

## Quy trình

1. **Đọc mô tả case** → hiểu input/expected/điều kiện
2. **Đánh giá tự động hóa được không:**
   - ✅ Được (logic thuần, API, DB, UI có thể automation) → viết test
   - ❌ Không (cần mắt người: visual, cảm nhận UI, nhập liệu phức tạp) → đánh dấu `manual-only` + lý do
3. **Viết test auto** cho đúng case đó (unit/integration/E2E tùy tầng)
4. **Chạy → XANH** (vì case đã pass tay, test phải pass) — nếu ĐỎ: phân tích (test sai? hay code đã đổi từ sau lần test tay?) → xử lý qua test-reflector
5. **Chống test vô hại:** assert phải bắt được behavior thật (không chỉ "chạy không crash"); nếu được, thử mutation nhanh
6. **Đánh dấu case trong manual-cases**: `converted-to-auto` (kèm path test) hoặc `manual-only`
7. **Đăng ký TỰ ĐỘNG (cuối luồng)** — kiểm tra trùng rồi append test mới vào **bộ test hoàn chỉnh** + `regression=true` (không cần command)

## Output
- Test file mới (append vào bộ test hoàn chỉnh — không dựng suite riêng)
- Cập nhật `.context/manual-cases/<case>.md` — trạng thái converted / manual-only
- `test-registry.json` — entry mới (`origin=manual`, `regression=true`)

## Gate
- [ ] Test xanh khi chạy
- [ ] Assert thật (không trivially)
- [ ] Case manual-only được ghi rõ lý do (không ép tự động hóa bằng mọi giá)
- [ ] Đã đăng ký vào bộ hoàn chỉnh (`test-registry.json`)