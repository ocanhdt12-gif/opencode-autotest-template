# /capture-manual — Chụp case đã test tay thành auto test (Nhánh C)

Tự động hóa các case **đã verify bằng tay** → lần sau chỉ chạy suite là biết còn pass không.

## Cách dùng
```
/capture-manual <case-id>   → case mô tả trong .context/manual-cases/<case-id>.md
/capture-manual <path>      → viết mô tả case rồi tự động hóa
```

## Flow
1. Đọc mô tả case (input, expected, các bước đã test tay pass)
2. Đánh giá tự động hóa được không:
   - ✅ Được → viết test auto (unit/integration/E2E tùy tầng)
   - ❌ Không (cần mắt người) → đánh dấu `manual-only` + lý do
3. Chạy test → **XANH** (case đã pass tay). Nếu ĐỎ → test-reflector phân tích (test sai? code đã đổi?)
4. Cập nhật `.context/manual-cases/<case>.md`: `converted-to-auto` / `manual-only`
5. **Đăng ký TỰ ĐỘNG (cuối luồng)** — kiểm tra trùng rồi append test mới vào bộ hoàn chỉnh + `regression=true` (lần sau chạy chung suite). Không cần gõ command.

## Rule
- Không ép tự động hóa case thật sự cần người (visual, cảm nhận) — ghi rõ lý do
- Test phải assert thật, không chỉ "chạy không crash"
- Test tự động hoá xong **add vào bộ test hoàn chỉnh** (không dựng suite riêng) — cuối luồng tự đăng ký, không cần command