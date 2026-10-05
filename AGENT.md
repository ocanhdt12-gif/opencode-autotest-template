# AGENT.md — Autotest Generation Pipeline

> Template xử lý bài toán: **sinh + duy trì auto test**. Match với template dev: **chỉ spec là cái chung** (`SPECIFICATIONS.md`).

## 5 luồng test (xem chi tiết `docs/FLOWS.md`)

| # | Luồng | Command | Đọc gì | Test gì |
|---|---|---|---|---|
| 1 | Full lần đầu (code mới từ template dev) | `/autotest --full` | SPECIFICATIONS.md | toàn bộ R-xx |
| 2 | Phần vừa sửa (bug/feature) 🔑 | `/test-scope` | `spec/test-scope/current.json` | direct + dependents + acceptance |
| 3 | Regression (retest luồng cũ) | `/regression` | test-scope + suite hiện có | regression + dependents |
| 4 | Manual→Auto (case test tay) | `/capture-manual` | manual-cases/ | case tay → auto, add regression |
| 5 | Theo test case user tạo | `/from-cases` | file case user | đúng case user |

## Nhánh sinh test (bổ trợ)

| Nhánh | Input | Output | Agent |
|---|---|---|---|
| A. Test-first | `SPECIFICATIONS.md` + task | test trước code (red → green) | `test-writer` → `test-reflector` |
| B. Characterization | Legacy chưa test | golden test khóa behavior | `characterization-writer` → `test-reflector` |
| C. Manual→Auto | Case đã test tay | auto test + add regression | `manual-capture-writer` → `test-reflector` |

## Hợp đồng bàn giao

Template DEV sinh `spec/test-scope/current.json` (có `specVersion`+`scopeVersion`) sau mỗi sửa (xem `skills/test-scope-contract` + `docs/SPEC_VERSIONING.md`) → template AUTOTEST (`scope-planner`) đọc để biết cần test gì, ghi `.context/test-status.json` để theo dõi version đã cover.

## Pipeline

```
SPECIFICATIONS.md (chung với template dev)
      │  spec-validator PASS
      ▼
Luồng 1 (full) / Luồng 2 (test-scope.json từ dev) / Luồng 3 (regression)
      │
      ├── Nhánh A: test-writer → chạy ĐỎ
      ├── Nhánh B: characterization-writer → mutation verify
      ├── Nhánh C: manual-capture-writer (luồng 4) · from-cases (luồng 5)
      │
      ▼
TEST REFLECTOR — phân loại fail: bug-test / bug-code → sửa đúng chỗ
      │
      ▼
VERIFY-TESTS — mutation + quality gate + hidden stash → PASS/FAIL
```

## Gate bắt buộc

1. `spec-validator` PASS trước khi sinh test
2. Test-first: test **ĐỎ đúng cách** trước khi code
3. Characterization: **mutation check bắt được**
4. Manual→Auto / from-cases: test xanh, assert thật, không ép tự động hóa case cần người
5. `verify-tests`: mutation score ≥ ngưỡng + quality gate + hidden stash
6. Mọi fail → test-reflector phân loại → sửa đúng chỗ

## Conventions

- Test file trong `tests/` hoặc cạnh code; test gắn `R-xx` (spec) hoặc case id (user/manual)
- Test name phản ánh hành vi, không implementation
- Không try-catch nuốt lỗi; không assert trivially
- Scrub unstable fields trong golden test
- `manual-only` case ghi rõ lý do
- Luồng 4/5: test tự động hoá xong **phải add vào regression suite** (luồng 3)
- Mỗi slice 1 commit