---
name: mutation-testing
description: "Đo chất lượng test bằng mutation testing — cố tình phá code (mutant) rồi xem test có bắt được không; mutation score là thước đo test thật, không phải coverage %. Dùng mutmut (Python) / Stryker (JS) hoặc dual-agent AdverTest. Dùng khi verify test suite trước merge hoặc khi nghi test 'vô hại'."
---

# Mutation Testing (Curated)

> Nguồn: mutmut (github.com/boxed/mutmut) · Stryker (stryker-mutator.io, JS/TS) · AdverTest (arXiv 2602.08146: dual-agent Test-writer ↔ Mutant-generator, mutation score làm feedback bidirectional).

## Vì sao dùng

Coverage % nói dối — test phủ nhiều dòng vẫn có thể không bắt bug (assert trivially, expected lấy từ code). **Mutation testing**: sinh hàng loạt "mutant" (code bị cố tình đổi: `>` thành `>=`, bỏ dòng, đổi hằng số...) rồi chạy test. Mutant nào test KHÔNG bắt = **survivor** → chỗ đó test yếu.

**Mutation score = số mutant bị bắt / tổng mutant hợp lệ** — ngưỡng mặc định ≥70% (tuỳ PROJECT_PROFILE).

## Chạy

```bash
# Python
mutmut run            # chạy toàn bộ
mutmut results        # xem survivors: file:line, mutagen
# JS/TS
npx stryker run
```

## Quy trình agent

1. Chạy mutation trên module vừa test.
2. **Survivor** → phân loại:
   - Test thiếu case (thêm test cho behavior đó)
   - Test quá yếu (assert không kiểm tra đúng output — sửa assert)
   - Mutant không đáng bắt (vd đổi tên biến local không ảnh hưởng — cho phép, ghi chú)
3. Lặp tới khi score ≥ ngưỡng hoặc survivors còn lại đều thuộc nhóm "không đáng".
4. Báo vào `/verify-tests` output.

## Dual-agent (nâng cao — AdverTest pattern)

Nếu muốn tối đa: agent thứ 2 chuyên **tạo mutant thoát được test hiện tại** (trong vùng coverage yếu), agent test-writer phải bắt → bidirectional improvement. Optional, không bắt buộc hàng ngày.

## Gate
- [ ] Mutation score ≥ ngưỡng (70% mặc định)
- [ ] Mọi survivor có lý do (thiếu test / test yếu / mutant vô hại)
- [ ] Không hạ ngưỡng để qua gate