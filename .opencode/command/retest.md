# /retest — Chạy LẠI test đã có (retest, KHÔNG tạo test mới)

> Lệnh **độc lập** với `/autotest`. Dùng khi đã có test trong bộ, chỉ muốn **chạy lại** để xác nhận còn xanh.
> Chạy được **toàn bộ** hoặc **1 chức năng/module**. **KHÔNG** sinh test mới (việc đó là của `/autotest`).

## Cách dùng
```
/retest --full              → retest TOÀN BỘ test đã có (tests/ + test-registry.json)
/retest <feature|module>    → retest 1 chức năng/module (lọc theo tag/module trong test-registry.json)
/retest --no-browser        → chỉ chạy ngầm (kết quả nhanh, không cần xem browser)
```

## Flow
1. **Chọn phạm vi**:
   - `--full` → lấy **toàn bộ** test trong `tests/` (theo `test-registry.json`)
   - `<feature|module>` → lọc test theo `tags` / module trong `test-registry.json` (khớp `refs`/`requirement`)
2. **Chạy NGẦM (headless)** — chạy lại bộ test đã chọn, không bung browser → lưu `.context/test-results/retest-headless-<scope>.json`
3. **Chạy BROWSER (headed, bung hẳn ra)** — E2E chạy lại để user theo dõi; cuối cùng **lưu kết quả + GIỮ browser mở** → `.context/test-results/retest-browser-<scope>.json`
4. **Cập nhật `status` / `lastRunAt`** trong `test-registry.json` + tiến độ `.context/coverage.json`
   - `test-reflector` phân loại fail (`bug-test` / `bug-code`); nếu vỡ do thay đổi chủ đích → cảnh báo

## Rule
- **KHÔNG tạo/sinh test mới** — retest chỉ chạy test **đã có** trong bộ hoàn chỉnh
- Muốn sinh test cho spec/tính năng mới → dùng `/autotest`
- Cùng cơ chế 2 bước: chạy ngầm (headless) trước → browser (headed, giữ mở) sau
- Cuối luồng tự cập nhật `lastRunAt`/`status` — không cần command phụ
- Vỡ → `test-reflector` phân loại; bug-code → báo dev, bug-test → sửa test
