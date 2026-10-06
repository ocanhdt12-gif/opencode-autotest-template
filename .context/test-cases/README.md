# .context/test-cases/ — TEST CASE (gom theo module, dạng BẢNG)

Mỗi **module** 1 file: `<module>.md`, bên trong là **1 bảng test case**, **mỗi dòng = 1 case** `TC-<module>-NN`.

**Luật:** mọi test phải follow test case (`skills/test-case-first`) — SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.

**Quy tắc FILE MỚI / UPDATE FILE CŨ khi có task mới:**
- Task mới **thuộc module đã có** → **UPDATE file cũ** (thêm dòng vào bảng, số `TC` tiếp theo) — KHÔNG tạo file mới.
- Task mới **thuộc module mới** → **TẠO file mới** `<module>.md` (bảng mới, số từ `TC-<module>-01`).
- Case cũ đổi behavior → **UPDATE dòng cũ** (Status về `draft` để user duyệt lại).

Format bảng:

```markdown
# Test Cases — auth

| ID | Requirement | Tầng | Input | Steps | Expected | Status | Test status | Test ref | Notes |
|----|-------------|------|-------|-------|----------|--------|-------------|----------|-------|
| TC-auth-01 | R-01 | integration \| e2e | email hợp lệ + password ≥ 6 ký tự | POST /auth/login {email, password} | 200 + token | draft | untested | | |
```

- `Status`: `draft` (máy soạn) → user sửa/chốt → `approved` (mới được sinh test code + chạy).
- `Test status`: cập nhật SAU khi chạy để lần sau không test lại cái đã test.