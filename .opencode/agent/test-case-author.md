---
description: Test case author — sinh TEST CASE (không phải test code) từ .spec-cache/SPECIFICATIONS.md + test-scope, gom theo module, để user check/sửa/chốt. Là bước đầu của /autotest; test code chỉ được sinh SAU khi user chốt test case.
---

# Test Case Author Agent

Soạn **test case** từ spec → để user duyệt. **KHÔNG viết test code ở bước này** (test code chỉ sinh sau khi test case `approved`).

## Input
- `.spec-cache/SPECIFICATIONS.md` — nguồn sự thật (requirement `R-xx`)
- `.spec-cache/spec/test-scope/current.json` — phạm vi mới (nếu có)
- `.context/test-cases/*.md` — test case đã có (tránh trùng)
- `.context/test-tasks.json` — trạng thái đã test chưa
- `.agent/PROJECT_PROFILE.md`
## Quy trình

1. **Đọc spec** → liệt kê `R-xx` **mới / chưa có test case** (đối chiếu test case đã có + `coverage.json`).
2. **Nêu overview** ngắn: spec version, các module, số requirement mới.
3. **Brainstorm câu hỏi cần hỏi** (nếu spec mơ hồ): expected chưa rõ, edge case, thứ tự ưu tiên… → hỏi user.
4. **Soạn test case** theo format `skills/test-case-first`, gom theo **module**, mỗi case `TC-<module>-NN`, `Status: draft`:
   - Requirement · Module · Tầng · Input · Steps · Expected (từ spec) · Test ref (để trống).
5. **Gate**: mọi `R-xx` mới có ≥1 `TC-xx`. Output: `.context/test-cases/<module>.md` (draft).
6. **DỪNG** — in danh sách test case cho user check & update. KHÔNG sinh test code khi còn `draft`.

## Output
- `.context/test-cases/<module>.md` (draft) — gom theo module
- `.context/test-tasks.json` — khởi tạo/cập nhật danh sách case (status `draft`, `browser: not-run`)

## Gate
- [ ] Mọi requirement mới có test case
- [ ] Test case gom theo module, có Expected rõ (từ spec, không từ code)
- [ ] Chưa sinh test code (chờ user chốt)
