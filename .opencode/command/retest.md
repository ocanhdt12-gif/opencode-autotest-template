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
0. ⚙️ **TỰ ĐỘNG sync spec** — `git -C .spec-cache pull --ff-only` ngay khi bắt đầu (KHÔNG cần gõ `/spec-link --sync`).
1. **ĐỌC TEST CASE** (3 nguồn, theo đúng thứ tự):
   - **(a) Bảng test case** `.context/test-cases/<module>.md` — nguồn liệt kê case (`TC-xx` + `Requirement` + `Steps/Expected`). `--all` → đọc **mọi file**; `<module>` → đọc **1 file**; `<TC-xx>` → tìm **1 dòng**.
   - **(b) `.context/test-tasks.json`** — lọc **CHỈ case `approved`** (case `draft`/chưa duyệt → bỏ qua, báo “case X còn draft — dùng /autotest để duyệt trước”; case mới chưa từng test → thuộc /autotest, không retest).
   - **(c) `test-registry.json`** — map `TC-xx → test file` (`file` + `testCase`): chạy **đúng test code đã có**. Case `approved` mà **chưa có entry trong registry** (chưa từng sinh test code) → KHÔNG retest được, báo dùng `/autotest` để sinh test code trước.
2. **Chọn phạm vi** (từ kết quả bước 1):
   - `--all` → toàn bộ case `approved` có test code
   - `<module|feature>` → case `approved` trong module đó
   - `<TC-xx>` → 1 case cụ thể
3. **Chạy NGẦM (headless)** — chạy lại phạm vi chọn, không bung browser → `.context/test-results/retest-headless-<scope>.json`
4. **Chạy BROWSER (headed, bung hẳn ra)** — E2E chạy lại cho user theo dõi; cuối cùng **lưu kết quả + GIỮ browser mở** → `.context/test-results/retest-browser-<scope>.json`
   - Với `--all`/1 cụm: chạy theo **từng case** như `/autotest` (xoạc xong 1 case chờ user chọn case tiếp nếu user muốn đi từng case).
5. **Cập nhật trạng thái**: `status`/`lastRunAt` trong `test-registry.json` + `Test status` trong `.context/test-cases/<module>.md` + `test-tasks.json` + `coverage.json`.
   - `test-reflector` phân loại fail (`bug-test` / `bug-code`); vỡ do thay đổi chủ đích → cập nhật test code theo test case (không sửa test case cho khớp code).

## Rule
- ⚙️ **Sync spec TỰ ĐỘNG đầu mỗi lần chạy** — không cần gõ `/spec-link --sync`
- **KHÔNG tạo test case/test mới** — chỉ chạy lại test **đã có** (đã approved)
- Muốn sinh test cho spec/tính năng mới → dùng `/autotest`
- Cùng cơ chế chạy: ngầm (headless) trước → browser (headed, giữ mở) sau
- Cuối luồng tự cập nhật trạng thái — không cần command phụ
- Vỡ → `test-reflector` phân loại; bug-code → báo dev, bug-test → sửa test
