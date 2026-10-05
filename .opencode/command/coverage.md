# /coverage — Board độ phủ test (do TEST tự lưu)

Board độ phủ **thuộc template TEST** — chỉ template test biết nó đã chạy gì. Danh sách req lấy từ spec (qua link git), **trạng thái do test tự cập nhật**.

## Cách dùng
```
/coverage                 → xem board: req nào covered/pending/untested/failing
/coverage --gaps          → chỉ liệt kê phần CHƯA test (pending/untested/failing)
/coverage --init          → khởi tạo board từ spec (đọc .spec-cache/SPECIFICATIONS.md → mọi R-xx = untested)
```

## Nơi lưu: `.context/coverage.json` (repo TEST — KHÔNG phải trong .spec-cache)

```jsonc
{
  "specVersion": "1.5.0",          // bám spec version hiện tại
  "updatedAt": "2026-10-05T14:00:00+07:00",
  "requirements": [
    { "id": "R-01", "title": "đăng ký tài khoản", "status": "covered",
      "testRef": "tests/auth.test.ts", "lastRunAt": "2026-10-05T14:05:00+07:00", "notes": "" },
    { "id": "R-05", "title": "thanh toán", "status": "pending", "testRef": null, "lastRunAt": null }
  ]
}
```

## Flow
1. `/coverage --init` (lần đầu): đọc `.spec-cache/SPECIFICATIONS.md` → liệt kê mọi `R-xx` → status `untested`
2. `/coverage`: so spec version (`.spec-cache`) với board (`.context/coverage.json`):
   - req mới trong spec chưa có trong board → thêm, status `pending`/`untested`
   - phân loại: `covered` (bỏ qua) · `pending`/`untested` (**cần test**) · `failing` (ưu tiên) · `n/a`
3. Test xong → **tự cập nhật board** (status `covered`/`failing` + `testRef` + `lastRunAt`)

## Rule
- Board **thuộc template TEST** — lưu ở `.context/coverage.json`, KHÔNG ghi vào `.spec-cache/` (read-only)
- Danh sách req lấy từ spec; **trạng thái test do test tự quyết** (không phụ thuộc dev)
- Không "tự nhận" covered khi chưa chạy test thật
- Spec version đổi → req mới/đổi → đánh dấu `pending` để test lại