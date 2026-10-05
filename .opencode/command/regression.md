# /regression — Retest luồng cũ sau update (luồng 3)

Đảm bảo các luồng cũ **không bị ảnh hưởng** sau thay đổi mới.

## Cách dùng
```
/regression                  → đọc impact.regression + dependents trong .spec-cache/spec/test-scope/current.json
/regression --all            → chạy lại TOÀN BỘ test đã có (an toàn, chậm hơn)
```

## Flow
1. Đọc `.spec-cache/spec/test-scope/current.json` → lấy `impact.regression` + `impact.dependents`
2. Chạy lại các test **đã có** thuộc các luồng đó (không viết mới trừ khi thiếu)
3. Nếu vỡ → `test-reflector` phân loại: bug-code (báo dev) / bug-test (sửa test)
4. Cập nhật `.context/test-status.json`
5. Báo cáo: luồng nào còn xanh, luồng nào vỡ + nguyên nhân

## Rule
- Regression = chạy test **đã tồn tại**, không phải sinh mới
- Bộ regression tự lớn lên: luồng 4 (manual→auto) + luồng 5 (from-cases) tự add test mới vào đây