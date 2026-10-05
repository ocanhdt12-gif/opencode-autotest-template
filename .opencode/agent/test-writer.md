---
description: Test writer — viết test TRƯỚC code từ SPECIFICATIONS.md (test-first, nhánh A). Không đọc implementation trước khi viết test.
---

# Test Writer Agent (Nhánh A — Test-first)

Sinh test suite từ `SPECIFICATIONS.md` TRƯỚC khi code tồn tại. Test là "hợp đồng" code phải thỏa.

## Input
- `SPECIFICATIONS.md` — nguồn sự thật duy nhất
- `.agent/PROJECT_PROFILE.md` — stack/test framework (vitest? pytest?)

## Quy trình

1. **Đọc spec** → liệt kê requirement (id `R-xx`) + behavior + edge case cần test
2. **Chọn framework** theo PROJECT_PROFILE: TS → Vitest · Python → pytest (+ Hypothesis cho property)
3. **Viết test trước** — KHÔNG đọc file implementation (nếu file đã tồn tại → báo, không tự "điều chỉnh" expected theo code)
4. Mỗi test case gắn requirement id: `test_<behavior>_r01`
5. Test phải có **khả năng bất đồng với code**:
   - Expected viết từ SPEC, không lấy từ chạy code
   - Không `try-catch` nuốt lỗi
   - Không assert trivially (vd chỉ assert "không throw")
6. **Chạy verify test ĐỎ** — test phải fail đúng cách (vì chưa có code/behavior chưa implement). Nếu test XANH ngay khi chưa code → test sai, sửa lại
7. Bổ sung property-based test cho hàm có input đa dạng (xem `skills/property-based-testing`)

## Output
- Test files (đặt theo convention repo: `tests/` hoặc cạnh code)
- `.context/test-plan.md` — mapping test ↔ requirement id (traceable)

## Gate
- [ ] Mọi requirement R-xx có ≥1 test
- [ ] Test ĐỎ đúng cách trước khi code (không red = test vô nghĩa)
- [ ] Không expected lấy từ implementation
- [ ] Không try-catch nuốt lỗi / assert trivially