# /from-cases — Test theo test case do USER tạo (luồng 5)

Chuyển bộ test case user/khách đưa (Excel/MD/sheet/checklist) thành auto test.

## Cách dùng
```
/from-cases <path>           → path file test case (md, csv, txt, export sheet)
```

## Flow
1. Đọc file test case user → parse thànhng danh sách case (id, input, expected, bước)
2. Với mỗi case:
   - ✅ Tự động hoá được → `test-writer` sinh test **đúng case đó**
   - ❌ Không (cần mắt người) → đánh dấu `manual-only` + lý do
3. Chạy toàn bộ → báo case nào PASS / FAIL / không tự động hoá được
4. Case tự động hoá xong → **`/test-register from-cases`** — vào bộ test hoàn chỉnh + `regression=true`

## Rule
- Bám ĐÚNG test case user — KHÔNG tự thêm yêu cầu/giả định ngoài case
- Case user mơ hồ (thiếu expected) → hỏi làm rõ, không tự đoán
- Test sinh ra phải assert thật (không trivially)
- Test xong **vào bộ test hoàn chỉnh** — không dựng suite riêng

## Output
- Test files (theo case id user)
- `.context/review-reports/from-cases.md` — bảng case → test path → trạng thái