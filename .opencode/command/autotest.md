# /autotest — Sinh test case từ spec → user chốt → chạy test (không retest)

> Luồng test tính năng **MỚI/spec mới**. Đọc `skills/test-case-first` — **mọi test phải follow TEST CASE**.
> Kết thúc ở: chạy ngầm (headless) log bug → **chạy browser từng case cho user kiểm tra** → cập nhật trạng thái task.

## Cách dùng
```
/autotest                  → luồng đầy đủ (lần đầu: hỏi config spec trước)
/autotest <module>         → giới hạn 1 module
/autotest --no-browser      → chỉ chạy ngầm (không mở browser)
```

## LẦN ĐẦU (chưa link spec)
1. **Hỏi config spec**: chưa có link → yêu cầu `/spec-link <git-url>` (hỏi, không tự bịa spec).
2. **Đọc spec + nêu overview**: spec version, các module, danh sách `R-xx`, phần chưa có test.
3. **Brainstorm câu hỏi cần hỏi**: chỗ spec mơ hồ (expected, edge case, ưu tiên) → hỏi user.
4. **Tạo test case** (xem bước A bên dưới).

## CÁC LẦN SAU
- Vào lệnh → đọc `.spec-cache/SPECIFICATIONS.md` + `test-scope` → lấy các **task/case CHƯA test** (đối chiếu `.context/test-tasks.json`) → tạo test case cho phần mới → chạy luồng y hệt bên dưới.

---

## A. Tạo test case (draft) — gom theo module
- `test-case-author` soạn **TEST CASE** (không phải test code) cho `R-xx` mới → `.context/test-cases/<module>.md`, mỗi case `TC-<module>-NN`, **`Status: draft`**.
- Khởi tạo/cập nhật `.context/test-tasks.json` (module → cases, `browser: not-run`).
- **Gate**: mọi requirement mới có ≥1 test case.

## B. ⛔ HUMAN CHECKPOINT — user sửa & update test case
- **DỪNG**. In danh sách test case theo module cho user.
- User **check + update** (sửa/xoá/thêm case) trực tiếp trong `.context/test-cases/<module>.md`.
- Chỉ tiếp tục khi user **chốt** → đánh dấu `Status: approved`.
- ⚠️ **KHÔNG sinh test code / chạy test khi test case còn `draft`.**

## C. Sinh test code THEO test case approved + chạy NGẦM (headless)
- `test-writer` sinh test code hiện thực hoá **đúng** các case `approved` — mỗi test gắn `TC-xx`; expected lấy từ test case; điền `Test ref` ngược vào test case.
  - **Không test gì ngoài test case.**
- Chạy **ngầm (headless)**, không bung browser, cho nhanh: `npm test` / `npx vitest run` / `pytest` / `npx playwright test`
- **LƯU kết quả + BUG ra file**: `.context/test-results/headless-run.json` (+ `.context/test-results/bugs.md` nếu có fail). Ghi rõ test nào fail → gắn `TC-xx`.
- ⚠️ **CHỈ LƯU LOG — chưa cập nhật "đã test" chính thức** (đối chiếu ở bước E).

## D. Chạy BROWSER cho user kiểm tra — TỪNG CASE, gom theo module
- Từ test case `approved`, tạo **list task chạy browser**, **gom theo module** cho dễ theo dõi (dựa `.context/test-tasks.json`).
- **Đi từng case 1** trong test case, **chạy browser HEADED (bung hẳn ra)** để user theo dõi thao tác:
  - Mở browser thật (`headless: false`), chạy 1 case → **giữ browser mở, không đóng** → để user xem màn hình kết quả.
  - **Sau khi 1 case xong → CHỜ user chọn case tiếp theo** (in danh sách case còn lại trong module + các module khác).
  - User có thể yêu cầu chạy lại case, chuyển module, hoặc dừng.
- Không tự chạy hết hàng loạt — **user điều khiển từng case**.

## E. Cập nhật trạng thái task (để không test lại)
- Sau mỗi case chạy xong: cập nhật `Test status` trong `.context/test-cases/<module>.md` + `headless`/`browser`/`lastRunAt` trong `.context/test-tasks.json`.
- Ghi `test-registry.json` (mỗi entry có `testCase: "TC-xx"`), cập nhật `.context/coverage.json` + `.context/test-status.json`.
- Lần sau `/autotest` chỉ lấy case **chưa test / fail** → không test lại cái đã test.

## Rule
- ⭐ **MỌI test follow test case** — không có test code ngoài test case `approved`
- **Test case phải được user check/update & chốt trước khi test** (checkpoint B)
- **Browser chạy TỪNG CASE 1**, gom theo module; xong 1 case **chờ user chọn case tiếp** — không tự chạy hết
- Browser **LUÔN headed**; cuối mỗi case **GIỮ browser mở** (không `browser.close()`)
- Chạy ngầm trước (headless, log bug) → browser sau (headed, user kiểm tra)
- **Cập nhật trạng thái task sau khi chạy** → không test lại cái đã test
- **KHÔNG retest toàn hệ thống** ở đây — việc đó thuộc `/retest`
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only)
