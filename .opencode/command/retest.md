# /retest — Chạy lại test đã có (toàn hệ thống / cụm chức năng / 1 test case)

> Lệnh **độc lập** với `/autotest`. Chỉ **chạy lại** test đã test rồi — **KHÔNG** tạo test case/test mới.
> ⭐ Mọi test đều follow TEST CASE — retest chạy đúng test case `approved` đã có (`skills/test-case-first`).

## Cách dùng
```
/retest --all                → retest LẠI TOÀN BỘ hệ thống (mọi test case đã approved)
/retest <module|feature>     → retest 1 cụm chức năng (theo module)
/retest <TC-xx>              → retest 1 test case
/retest ... --no-browser      → chỉ chạy ngầm (headless)
```

## Flow
1. **Chọn phạm vi**:
   - `--all` → toàn bộ test case đã có (theo `.context/test-tasks.json` + `test-registry.json`)
   - `<module|feature>` → lọc theo module/cụm chức năng
   - `<TC-xx>` → 1 test case cụ thể
2. **Chạy NGẦM (headless)** — chạy lại phạm vi chọn, không bung browser → `.context/test-results/retest-headless-<scope>.json`
3. **Chạy BROWSER (headed, bung hẳn ra)** — E2E chạy lại cho user theo dõi; cuối cùng **lưu kết quả + GIỮ browser mở** → `.context/test-results/retest-browser-<scope>.json`
   - Với `--all`/1 cụm: chạy theo **từng case** như `/autotest` (xoạc xong 1 case chờ user chọn case tiếp nếu user muốn đi từng case).
4. **Cập nhật trạng thái**: `status`/`lastRunAt` trong `test-registry.json` + `Test status` trong `.context/test-cases/<module>.md` + `test-tasks.json` + `coverage.json`.
   - `test-reflector` phân loại fail (`bug-test` / `bug-code`); vỡ do thay đổi chủ đích → cập nhật test code theo test case (không sửa test case cho khớp code).

## Rule
- **KHÔNG tạo test case/test mới** — chỉ chạy lại test **đã có** (đã approved)
- Muốn sinh test cho spec/tính năng mới → dùng `/autotest`
- Cùng cơ chế chạy: ngầm (headless) trước → browser (headed, giữ mở) sau
- Cuối luồng tự cập nhật trạng thái — không cần command phụ
- Vỡ → `test-reflector` phân loại; bug-code → báo dev, bug-test → sửa test
