# /coverage — Xem board độ phủ test (CHỈ ĐỂ XEM)

> ⚙️ **Board tự cập nhật — không cần gõ `/coverage`.** Cuối MỖI luồng test, agent **tự quét độ phủ** và ghi `.context/coverage.json` (xem `skills/complete-test-suite`). Command này chỉ để **xem báo cáo**.

Board độ phủ **thuộc template TEST** — chỉ template test biết nó đã chạy gì. Danh sách req lấy từ spec (qua link git), **trạng thái do bước tự động cuối luồng cập nhật**.

## Cách dùng (chỉ xem)
```
/coverage                 → xem board: req nào covered/pending/untested/failing
/coverage --gaps          → chỉ liệt kê phần CHƯA test (pending/untested/failing)
```
> Cập nhật board: **tự động** sau mỗi luồng test (không gõ). `/coverage --init` cũng đã tự động hoá — board thiếu thì bước cuối luồng tự khởi tạo từ spec.

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

## Flow (tự động — xem `skills/complete-test-suite`)
1. Board thiếu/chưa có → **tự khởi tạo** từ `.spec-cache/SPECIFICATIONS.md` (mọi `R-xx` = `untested`)
2. So spec version (`.spec-cache`) với board (`.context/coverage.json`):
   - req mới trong spec chưa có trong board → thêm, status `pending`/`untested`
   - phân loại: `covered` (bỏ qua) · `pending`/`untested` (**cần test**) · `failing` (ưu tiên) · `n/a`
3. Test xong → **board tự cập nhật** (status `covered`/`failing` + `testRef` + `lastRunAt`)

## Rule
- **Cập nhật board KHÔNG cần gõ command** — bước tự động cuối mỗi luồng test ghi; `/coverage` chỉ đọc
- Board **thuộc template TEST** — lưu ở `.context/coverage.json`, KHÔNG ghi vào `.spec-cache/` (read-only)
- Danh sách req lấy từ spec; **trạng thái test do test tự quyết** (không phụ thuộc dev)
- Không "tự nhận" covered khi chưa chạy test thật
- Spec version đổi → req mới/đổi → đánh dấu `pending` để test lại
