# /coverage — Xem tiến độ test (CHỈ ĐỂ XEM)

> ⚙️ **Tiến độ tự cập nhật — không cần gõ `/coverage`.** Sau khi **browser test** (`/autotest` giai đoạn 2) chạy xong, agent **tự cập nhật tiến độ** và ghi `.context/coverage.json` (xem `skills/complete-test-suite`). Chạy ngầm (headless) không cập nhật. Command này chỉ để **xem báo cáo**.

**Tiến độ test thuộc template TEST** — chỉ template test biết nó đã chạy gì. **Danh sách req lấy từ spec** (`.spec-cache/SPECIFICATIONS.md`, đọc qua link git — test KHÔNG tạo spec); **trạng thái do bước tự động cuối luồng cập nhật**.

## Cách dùng (chỉ xem)
```
/coverage                 → xem board: req nào covered/pending/untested/failing
/coverage --gaps          → chỉ liệt kê phần CHƯA test (pending/untested/failing)
```
> Cập nhật tiến độ: **tự động** sau khi browser test xong (không gõ). `/coverage --init` cũng đã tự động hoá — board thiếu thì bước cuối luồng tự khởi tạo danh sách req từ spec.

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
1. Board thiếu/chưa có → **khởi tạo danh sách req từ spec** (`.spec-cache/SPECIFICATIONS.md`, mọi `R-xx` = `untested`) — chỉ liệt kê, không tạo nội dung spec
2. So spec version (`.spec-cache`) với board (`.context/coverage.json`):
   - req mới trong spec chưa có trong board → thêm, status `pending`/`untested`
   - req đã bỏ khỏi spec → `n/a`
   - phân loại: `covered` (bỏ qua) · `pending`/`untested` (**cần test**) · `failing` (ưu tiên) · `n/a`
3. Test xong → **tiến độ tự cập nhật** (status `covered`/`failing` + `testRef` + `lastRunAt`)

## Rule
- **Cập nhật tiến độ KHÔNG cần gõ command** — bước tự động sau khi browser test xong ghi; `/coverage` chỉ đọc
- **Spec chỉ đọc từ link** — test KHÔNG tạo/sinh/sửa spec; board KHÔNG ghi vào `.spec-cache/` (read-only)
- Danh sách req lấy từ spec; **trạng thái test do test tự quyết** (không phụ thuộc dev)
- Không "tự nhận" covered khi chưa chạy test thật
- Spec version đổi → req mới/đổi → đánh dấu `pending` để test lại
