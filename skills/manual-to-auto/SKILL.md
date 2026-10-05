---
name: manual-to-auto
description: "Chụp các case đã test tay (manual) thành auto test regression — mô tả case trong .context/manual-cases/, agent viết test tự động, chạy xanh, đánh dấu converted-to-auto; case cần mắt người thì ghi manual-only + lý do. Dùng khi '/capture-manual' hoặc 'case này test tay xong rồi viết test tự động đi'."
---

# Manual → Auto Test Capture (In-house)

## Vì sao dùng

Test tay tốn công và không lặp lại được. Case đã verify bằng tay → chụp thành auto test → **lần sau chạy 1 lệnh là biết còn pass không** (regression). Giảm dần workload manual theo thời gian.

## Quy trình (verified với manual-capture-writer)

1. **Input — mô tả case** (`.context/manual-cases/<case-id>.md` hoặc từ user):
   - Input cụ thể
   - Expected (đã xác nhận bằng tay)
   - Các bước đã kiểm tra + kết quả tay
2. **Đánh giá tự động hóa được không:**
   - ✅ Logic thuần, API, DB, UI automation được → viết test
   - ❌ Cần cảm nhận người (visual design, UX, nhập liệu phức tạp, thiết bị thật) → `manual-only` + lý do cụ thể — **không ép**
3. **Viết test auto** đúng case (unit/integration/E2E tùy tầng; framework theo PROJECT_PROFILE).
4. **Chạy → XANH** (case đã pass tay; nếu ĐỎ → test-reflector: test sai hay code đã đổi từ sau lần test tay?)
5. **Assert thật** — không chỉ "chạy không crash" (nếu được, mutation nhanh).
6. Ghi trạng thái vào `.context/manual-cases/<case>.md`: `converted-to-auto` (kèm path test) / `manual-only`.

## Format case

```markdown
# Manual Case: <id>
**Tầng:** unit | integration | e2e | ui-visual
**Input:** ...
**Expected:** ...
**Đã test tay:** ngày, kết quả PASS
**Trạng thái:** converted-to-auto | manual-only
**Lý do manual-only (nếu có):** ...
**Test path (nếu converted):** tests/...
```

## Gate
- [ ] Test xanh khi chạy
- [ ] Assert kiểm tra behavior thật
- [ ] Manual-only có lý do rõ (không lười)