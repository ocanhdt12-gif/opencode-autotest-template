---
name: test-case-first
description: "Luật trục của template: MỌI test phải follow TEST CASE. Spec → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy. Test case viết DƯỚI DẠNG BẢNG theo module (mỗi dòng 1 case) cho dễ nhìn; task mới cùng module → UPDATE file cũ (thêm dòng), module mới → tạo file mới. User duyệt trước khi chạy; test code chỉ hiện thực hoá đúng test case đã chốt; trạng thái từng case lưu lại để không test lại. Dùng khi /autotest, /retest, sinh test case, hoặc khi test lệch test case."
---

# Test-case-first — mọi test đều follow TEST CASE

> **Luật bất biến (anh chốt 10/2026):** test KHÔNG tự do sinh theo code. Chuỗi nguồn sự thật:
> **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy**.

## Vì sao

Test sinh thẳng từ spec/code dễ "tự diễn giải" → lệch ý người kiểm thử. Chèn **test case** làm lớp giữa: user **đọc/sửa/chốt** test case trước, rồi test code chỉ hiện thực hoá **đúng** test case đã chốt. Nhờ vậy test đo đúng cái user muốn, mọi test truy được về 1 test case, và trạng thái "đã test chưa" nằm ở test case (không test lại cái đã test).

## Nơi lưu (gom theo MODULE, dạng BẢNG)

- Test case: `.context/test-cases/<module>.md` — mỗi module 1 file, bên trong là **1 bảng**, **mỗi dòng = 1 test case** (`TC-xx`).
- Trạng thái task (đã test chưa): `.context/test-tasks.json` — nguồn để biết case nào **chưa test** (lần sau chỉ chạy phần mới).

## Format test case — DÙNG BẢNG (bắt buộc)

```markdown
# Test Cases — auth

| ID | Requirement | Tầng | Input | Steps | Expected | Status | Test status | Test ref | Notes |
|----|-------------|------|-------|-------|----------|--------|-------------|----------|-------|
| TC-auth-01 | R-01 | integration \| e2e | email hợp lệ + password ≥ 6 ký tự | POST /auth/login {email, password} | 200 + token | draft | untested | | |
| TC-auth-02 | R-01 | integration | email sai format / password < 6 | POST /auth/login {email:"bad"} | 401 invalid_credentials | draft | untested | | |
```

- **TC-xx**: id case, prefix theo module (`TC-<module>-NN`).
- **Requirement**: bám `R-xx` trong spec.
- **Status**: `draft` (máy soạn) → `approved` (user đã check/update) → mới được test.
- **Test status**: `untested` → `passed`/`failed`/`skipped` (cập nhật sau mỗi lần chạy).
- **Test ref**: path test code hiện thực hoá case (điền sau khi sinh test).

Schema `.context/test-tasks.json`:
```jsonc
{
  "updatedAt": "2026-10-06T12:00:00+07:00",
  "specVersion": "1.2.0",
  "modules": [
    { "module": "auth",
      "cases": [
        { "id": "TC-auth-01", "requirement": "R-01", "title": "đăng nhập thành công",
          "status": "approved", "headless": "pass", "browser": "not-run",
          "lastRunAt": null, "bugs": [] }
      ] }
  ]
}
```

## 📌 FILE MỚI hay UPDATE FILE CŨ khi có task mới (anh chốt 10/2026)

Khi có **task mới cần test** (feature mới / requirement mới / spec mới sinh thêm case):

| Tình huống | Làm gì |
|---|---|
| Task mới **thuộc module ĐÃ CÓ** (vd module `auth` đã có `auth.md`, thêm case login-GOOGLE) | **UPDATE FILE CŨ** — thêm **dòng mới vào bảng** của `auth.md`, đánh số `TC` tiếp theo (`TC-auth-03`…). **KHÔNG tạo file mới.** |
| Task mới **thuộc module MỚI** (chưa có file) | **TẠO FILE MỚI** `<module>.md` với bảng mới, đánh số từ `TC-<module>-01`. |
| Test case cũ đổi behavior (do spec đổi) | **UPDATE DÒNG cũ** trong bảng (sửa Input/Steps/Expected + `Status: draft` lại để user duyệt lại), không thêm dòng trùng. |

- Đồng thời cập nhật `.context/test-tasks.json` cho khớp (thêm case mới / sửa case cũ trong module tương ứng).
- Nguyên tắc: **1 module = 1 file** — không rải test case của cùng module ra nhiều file, không tạo file trùng (`auth.md` + `auth-2.md`…). Tra `.context/test-cases/` trước khi tạo mới.

## Luật

1. **Mọi test phải có test case tương ứng.** Test code không gắn `TC-xx` → **không hợp lệ**.
2. **Test case phải `approved` mới chạy.** Soạn xong (draft) → **DỪNG cho user check & update** → user chốt → mới sinh test code + chạy.
3. **Test code hiện thực hoá ĐÚNG test case** — expected/kịch bản lấy từ test case, không từ chạy code. Không test gì ngoài test case.
4. **Test case đổi → test code cập nhật theo** (không sửa test case cho khớp code).
5. **Traceability 2 lớp**: `R-xx` (spec) ↔ `TC-xx` (test case) ↔ test code (`test-registry.json` field `testCase`).
6. **Không test lại cái đã test**: sau khi chạy, cập nhật `Test status`/`browser` trong test case + `test-tasks.json`; lần sau chỉ lấy case `untested`/`failed`.
7. Test case là artifact **được commit** (`.context/test-cases/`), user sửa trực tiếp.

## Gate
- [ ] Mọi test code có `TC-xx` tương ứng
- [ ] Test case đã `approved` bởi user trước khi chạy
- [ ] Test code bám đúng test case (không bịa thêm case ngoài)
- [ ] Test case viết dạng **bảng**, 1 module = 1 file
- [ ] Task mới: cùng module → **update file cũ** (thêm dòng); module mới → **tạo file mới**
- [ ] Trạng thái test case + `test-tasks.json` đã cập nhật sau khi chạy