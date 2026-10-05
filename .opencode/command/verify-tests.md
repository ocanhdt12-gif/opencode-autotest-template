# /verify-tests — Kiểm tra chất lượng test suite (gate bắt buộc)

Chạy toàn bộ test + mutation + quality gate trước khi merge.

## Flow
1. Chạy full test suite (framework theo PROJECT_PROFILE)
2. **Mutation testing**: `mutmut` (Python) / `stryker` (JS) → mutation score
   - Score < ngưỡng (mặc định 70%? — theo PROJECT_PROFILE) → FAIL, cần thêm test
3. **Quality gate** (skills/test-quality-gate):
   - Expected KHÔNG lấy từ chạy code
   - Không try-catch nuốt lỗi
   - Không assert trivially
   - Hidden stash test (không nằm trong prompt agent) chạy + pass
4. Output `.context/review-reports/test-quality.md` + verdict

## Verdict
- PASS → sẵn sàng merge
- FAIL → liệt kê: mutation survivors, test vô hại, hidden stash fail → loop sửa test (KHÔNG sửa code để làm test pass trừ khi bug thật)