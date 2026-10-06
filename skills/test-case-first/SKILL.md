---
name: test-case-first
description: "Luật trục của template: MỌI test phải follow TEST CASE. Spec → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy. Test case soạn theo module, user duyệt trước khi chạy; test code chỉ hiện thực hoá đúng test case đã chốt; trạng thái từng task lưu lại để không test lại. Dùng khi /autotest, /retest, sinh test case, hoặc khi test lệch test case."
---

# Test-case-first — mọi test đều follow TEST CASE

> **Luật bất biến (anh chốt 10/2026):** test KHÔNG tự do sinh theo code. Chuỗi nguồn sự thật:
> **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy**.

## Vì sao

Test sinh thẳng từ spec/code dễ "tự diễn giải" → lệch ý người kiểm thử. Chèn **test case** làm lớp giữa: user **đọc/sửa/chốt** test case trước, rồi test code chỉ hiện thực hoá **đúng** test case đã chốt. Nhờ vậy test đo đúng cái user muốn, mọi test truy được về 1 test case, và trạng thái "đã test chưa" nằm ở test case (không test lại cái đã test).

## Nơi lưu (gom theo MODULE)

- Test case: `.context/test-cases/<module>.md` — mỗi module 1 file, chứa nhiều `TC-xx`.
- Trạng thái task (đã test chưa): `.context/test-tasks.json` — nguồn để biết case nào **chưa test** (lần sau chỉ chạy phần mới).

## Format test case

```markdown
# Test Cases — <module>

## TC-<module>-01 — <tên ngắn hành vi>
**Requirement:** R-01
**Module:** auth
**Tầng:** unit | integration | e2e | ui
**Input:** email/password hợp lệ
**Steps:** POST /auth/login {email, password}
**Expected:** 200 + token, < 2s
**Status:** draft | approved                  ← user chốt ở bước review
**Test status:** untested | passed | failed | skipped   ← cập nhật SAU khi chạy
**Test ref:** tests/auth.spec.ts::login_r01    ← điền sau khi sinh test code
**Notes:** (user có thể sửa/xoá/thêm)
```

- **TC-xx**: id case, prefix theo module (vd `TC-auth-01`).
- **Requirement**: bám `R-xx` trong spec.
- **Status**: `draft` (máy soạn) → `approved` (user đã check/update) → mới được test.
- **Test status**: `untested` → `passed`/`failed`/`skipped` (cập nhật sau mỗi lần chạy).

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
- [ ] Trạng thái test case + `test-tasks.json` đã cập nhật sau khi chạy
