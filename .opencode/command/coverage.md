# /coverage — Xem + báo cáo độ phủ test theo requirement

Đọc board độ phủ `spec/coverage.json` (từ dev, qua `.spec-cache/`) → biết req nào đã/chưa test → test phần còn thiếu → báo cáo về.

## Cách dùng
```
/coverage                 → xem board: req nào covered/pending/untested/failing
/coverage --gaps          → chỉ liệt kê phần CHƯA test (pending/untested/failing)
/coverage --report        → xuất .context/coverage-report.json (báo về dev)
```

## Flow
1. Đọc `.spec-cache/spec/coverage.json` (board của dev)
2. Phân loại:
   - `covered` → đã test, bỏ qua
   - `pending`/`untested` → **cần test** (gợi ý `/test-scope` nếu có scope, hoặc `/autotest` theo req)
   - `failing` → test đang fail → ưu tiên xử lý
   - `n/a` → bỏ qua (có lý do)
3. Sau khi chạy test → xuất `.context/coverage-report.json`:
   ```jsonc
   { "specVersion": "1.5.0", "generatedAt": "...",
     "requirements": [ { "id": "R-01", "status": "covered", "testRef": "tests/auth.test.ts", "lastRunAt": "..." } ] }
   ```
4. Báo user: % đã cover, danh sách còn thiếu, đề xuất bước tiếp

## Rule
- Board `spec/coverage.json` **thuộc dev** — chỉ đọc, không sửa trong `.spec-cache/`
- Test báo về bằng `coverage-report.json` (dev cập nhật board theo report)
- Không "tự nhận" covered khi chưa chạy test thật