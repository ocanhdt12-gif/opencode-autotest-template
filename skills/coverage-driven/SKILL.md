---
name: coverage-driven
description: "Dùng coverage report để tìm nhánh chưa test rồi bổ sung (không đuổi % mù) — ưu tiên nhánh quan trọng theo spec (core logic, error path, edge) trước. Dùng khi test suite đã có nhưng nghi thiếu sót, hoặc sau characterization bước 2."
---

# Coverage-Driven Test Development (In-house)

## Vì sao dùng

Coverage % cao ≠ test tốt (mutation mới đo chất lượng). Nhưng coverage vẫn hữu ích ở chỗ: **chỉ ra vùng code chưa được chạm** → nơi có thể còn bug chưa test.

## Quy trình

1. Chạy coverage:
   - Python: `pytest --cov=<module> --cov-report=term-missing`
   - TS: `vitest --coverage`
2. **Phân loại vùng chưa phủ theo mức quan trọng** (dựa trên SPEC — không đuổi % mù):
   - 🟥 Core logic / requirement R-xx chính → PHẢI bổ sung test
   - 🟧 Error path, edge case (spec 2D: empty, invalid, concurrent...) → nên bổ sung
   - 🟩 Không quan trọng (boilerplate, thư viện, code sẽ xóa) → bỏ qua, ghi chú
3. Thêm test cho vùng ưu tiên — test phải assert behavior theo spec (quality gate).
4. Chạy lại coverage + mutation → xác nhận tiến bộ thật (không chỉ tăng %).

## Lưu ý
- Coverage bổ sung PHẢI qua test-quality-gate (không thêm test vô hại cho đủ %)
- Ghi chú vùng cố tình không test (lý do) — tránh "con số ảo"
- Kết hợp mutation: vùng phủ cao nhưng mutation fail → test yếu, không phải thiếu phủ

## Output
- Danh sách vùng đã bổ sung + lý do ưu tiên
- Coverage/mutation trước-sau