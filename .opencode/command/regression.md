# /regression — Retest luồng cũ sau update (luồng 3)

Đảm bảo các luồng cũ **không bị ảnh hưởng** sau thay đổi mới.

## Cách dùng
```
/regression                  → đọc impact.regression + dependents trong .context/test-scope.json
/regression --all            → chạy lại TOÀN BỘ test đã có (an toàn, chậm hơn)
```

## Flow
1. Đọc `test-scope.json` → lấy `impact.regression` + `impact.dependents`
2. Chạy lại các test **đã có** thuộc các luồng đó (không viết mới trừ khi thiếu)
3. Nếu vỡ → `test-reflector` phân loại: bug-code (báo dev) / bug-test (sửa test)
4. Báo cáo: luồng nào còn xanh, luồng nào vỡ + nguyên nhân

## Rule
- Regression = chạy test **đã tồn tại**, không phải sinh mới
- Test suite này là "lưới an toàn" — luôn cập nhật khi thêm test mới (luồng 4, 5 tự add vào đây)