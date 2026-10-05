# AGENT.md — Autotest Generation Pipeline

> Template xử lý bài toán: **sinh + duy trì auto test** cho code mới (test-first), legacy (characterization), và sau manual test (capture). Match với template dev: spec chung + Error Analyzer chung.

## 3 Nhánh core

| Nhánh | Input | Output | Agent chính |
|---|---|---|---|
| A. Test-first | `SPECIFICATIONS.md` + layer/task | Test suite trước code (red → green) | `test-writer` → `test-reflector` |
| B. Characterization | File/module legacy chưa test | Golden/snapshot test khóa behavior | `characterization-writer` → `test-reflector` |
| C. Manual→Auto | Mô tả case đã test tay | Auto test regression (hoặc đánh dấu manual-only) | `manual-capture-writer` → `test-reflector` |

## Pipeline

```
SPECIFICATIONS.md (chung với template dev)
      │  spec-validator PASS
      ▼
TEST PLANNER → test plan traceable (mỗi test → requirement id)
      │
      ├── Nhánh A: test-writer (SPEC → test, ko đọc impl) → chạy ĐỎ
      ├── Nhánh B: characterization-writer (code → golden, scrub unstable) → mutation verify
      └── Nhánh C: manual-capture-writer (case manual → auto test)
      │
      ▼
TEST REFLECTOR — chạy + phân loại fail: bug-test / bug-code
      │  FAIL
      ▼
ERROR ANALYZER (match template dev) — 4 phases Iron Law → `.context/error-memory.md` → lần sau tránh
      │
      ▼
VERIFY-TESTS — mutation testing + quality gate + hidden stash → PASS/FAIL
```

## Gate bắt buộc

1. `spec-validator` PASS trước khi sinh test (match spec-validator của template dev)
2. Test-first: test **ĐỎ đúng cách** trước khi code (không red là test sai)
3. Characterization: **mutation check bắt được** (test vô nghĩa nếu phá code mà test không fail)
4. Manual→Auto: case tự động hóa được phải **xanh khi chạy**; không ép manual-only
5. `verify-tests`: mutation score ≥ ngưỡng + quality gate + hidden stash — trước khi merge
6. Mọi fail đi qua **Error Analyzer** → `error-memory.md` (field `stack: dev|autotest`)

## Conventions

- Test file: `tests/` hoặc cạnh code (theo framework repo); test case gắn `R-xx` từ spec
- Test name phản ánh hành vi, không phải implementation
- Không `try-catch` nuốt lỗi trong test; không assert trivially
- Scrub unstable fields trong golden test (timestamp, id, random)
- Ghi `manual-only` case kèm lý do — không tự động hóa bằng mọi giá
- Mỗi slice 1 commit, review được