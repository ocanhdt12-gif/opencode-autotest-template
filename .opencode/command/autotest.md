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
   - Sau khi link xong, các lần chạy sau **tự sync** — KHÔNG cần gõ `/spec-link --sync` thủ công.
2. **Hỏi link deploy web**: chưa có `web_app_url` trong `.agent/PROJECT_PROFILE.md` → hỏi user link deploy
   (vd `https://staging.example.com`) → điền vào `web_app_url`. Đây là **baseURL** cho test browser/E2E.
   - ⚠️ **Không bắt buộc để bắt đầu.** Thiếu `web_app_url` thì **vẫn chạy được bước C (headless)** cho test tầng `unit`/`integration`; CHỈ **bước D (browser/E2E)** bị hoãn. Nếu test case có tầng `e2e`/`browser` mà chưa có url → hỏi url trước khi tới D (không chặn A→C).
3. **Đọc spec + nêu overview**: spec version, các module, danh sách `R-xx`, phần chưa có test.
4. **Brainstorm câu hỏi cần hỏi**: chỗ spec mơ hồ (expected, edge case, ưu tiên) → hỏi user.
5. ⛔ **HUMAN CHECKPOINT — DUYỆT SPEC trước khi tạo test case** (xem bước **A0** bên dưới): nêu lại spec vừa hiểu (bằng lời của agent) → **DỪNG cho user duyệt** → user OK mới sang tạo test case.
6. **Tạo test case** (xem bước A bên dưới).

## CÁC LẦN SAU
- ⚙️ **TỰ ĐỘNG sync spec trước khi test** — chạy `git -C .spec-cache pull --ff-only` ngay khi vào lệnh (KHÔNG cần gõ `/spec-link --sync`). Cache hỏng/mất mạng → dùng cache cũ + cảnh báo "spec có thể cũ".
- Đọc `.spec-cache/SPECIFICATIONS.md` + `test-scope` → lấy các **task/case CHƯA test** (đối chiếu `.context/test-tasks.json`) → tạo test case cho phần mới → chạy luồng y hệt bên dưới.
- 🔁 **PHÁT HIỆN TEST CASE STALE do spec đổi** (quan trọng — tránh test cũ chạy theo behavior sai):
  - Đọc `.spec-cache/spec/test-scope/current.json`: nếu `trigger` = `feature-update`/`bug-fix`, lấy `specRefs` (các `R-xx` bị đổi) + `changed.modules`.
  - Đối chiếu với test case **đã `approved`**: case nào map tới `R-xx` vừa đổi → **đánh dấu `Status: draft` lại** (behavior có thể đã khác) theo rule `skills/test-case-first` (UPDATE dòng cũ, không thêm trùng).
  - In cho user danh sách case **nghi stale** → yêu cầu check/update expected rồi chốt `approved` lại trước khi chạy. KHÔNG tự chạy test case nghi stale với expected cũ.

---

## A0. ⛔ HUMAN CHECKPOINT — DUYỆT SPEC (trước khi tạo test case)
- Sau khi đọc spec, **nêu lại cho user bằng lời của agent** (KHÔNG chép nguyên văn spec) để xác nhận hiểu đúng:
  - **Phạm vi lần này**: module/tính năng nào sẽ test, các `R-xx` liên quan.
  - **Hiểu của agent về từng requirement**: mỗi `R-xx` làm gì + expected behavior chính (1-2 dòng/req).
  - **Giả định & điểm mơ hồ**: chỗ spec không rõ agent đang hiểu/giả định thế nào (để user sửa sớm).
  - **Ngoài phạm vi**: phần spec agent CỐ Ý chưa test lần này (nếu có) + lý do.
- **DỪNG — chờ user duyệt.** User xác nhận "đúng/OK" hoặc chỉnh lại cách hiểu.
- ⚠️ **Chưa được tạo test case khi user chưa duyệt spec.** Nếu user sửa cách hiểu → cập nhật rồi nêu lại, duyệt xong mới sang A.
- Mục đích: bắt sai sót **ở tầng hiểu spec** trước, rẻ hơn nhiều so với phát hiện khi đã soạn cả bộ test case.

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
- 🔀 **Phân loại fail NGAY (gọi `test-reflector`) trước khi sang D:**
  - `bug-in-test` (test sai) → sửa test theo test case rồi chạy lại headless; KHÔNG sang D với test sai.
  - `bug-in-code` (app sai, test đúng) → ghi `bugs.md` + **báo user/dev**; case đó **đánh dấu `failing`**, KHÔNG chạy browser (D) cho tới khi code fix (browser chỉ để user xác nhận hành vi ĐÚNG — chạy case đang có bug code là vô nghĩa).
  - `GREEN` → đủ điều kiện sang D.
  - ⛔ Nếu TẤT CẢ case headless đều `bug-in-code` (app hỏng) → dừng, báo user "code đang lỗi, cần fix trước khi kiểm browser", không sang D.

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
- Browser **LUÔN headed** khi chạy bước D; cuối mỗi case **GIỮ browser mở** (không `browser.close()`).
  - ⚠️ **Ngoại lệ có chủ đích:** cờ `--no-browser` **bỏ qua bước D** (chỉ chạy C headless — dùng cho CI / chạy nhanh / chưa có `web_app_url`). Khi đã vào D thì mới áp rule "luôn headed".
- Chạy ngầm trước (headless, log bug) → browser sau (headed, user kiểm tra)
- **Cập nhật trạng thái task sau khi chạy** → không test lại cái đã test
- **KHÔNG retest toàn hệ thống** ở đây — việc đó thuộc `/retest`
- ⚙️ **Sync spec TỰ ĐỘNG đầu mỗi lần chạy** — chạy `git -C .spec-cache pull --ff-only` (không cần gõ `/spec-link --sync`; chỉ lần đầu chưa link mới cần `/spec-link <git-url>`)
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only)
