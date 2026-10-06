# .context/test-cases/ — TEST CASE (gom theo module)

Mỗi **module** 1 file: `<module>.md`, chứa nhiều test case `TC-<module>-NN`.

**Luật:** mọi test phải follow test case (`skills/test-case-first`) — SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.

Format mỗi case:

```markdown
## TC-<module>-01 — <tên ngắn hành vi>
**Requirement:** R-01
**Module:** <module>
**Tầng:** unit | integration | e2e | ui
**Input:** ...
**Steps:** ...
**Expected:** ...
**Status:** draft | approved
**Test status:** untested | passed | failed | skipped
**Test ref:** tests/...::<test-name>
**Notes:** ...
```

- `Status`: `draft` (máy soạn) → user sửa/chốt → `approved` (mới được sinh test code + chạy).
- `Test status`: cập nhật SAU khi chạy để lần sau không test lại cái đã test.
