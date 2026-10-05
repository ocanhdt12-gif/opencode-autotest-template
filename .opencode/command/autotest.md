# /autotest — Sinh test tự động từ spec (Nhánh A)

Sinh test suite cho code mới từ `SPECIFICATIONS.md`, test-first (test trước code).

## Cách dùng
```
/autotest                 → test cho toàn bộ spec (theo layer/task hiện tại)
/autotest <module>        → test cho 1 module cụ thể
```

## Flow
1. Đọc `SPECIFICATIONS.md` + `.agent/PROJECT_PROFILE.md` (framework test)
2. Gọi subagent `test-writer` → viết test trước (unit + property-based), traceable `R-xx`
3. Chạy test → phải **ĐỎ đúng cách** (fail vì chưa có code, không phải fail vì test sai)
4. Báo user: test đã sẵn sàng, implement code tới khi XANH
5. (Tùy chọn) sau khi code xong: chạy `/verify-tests`

## Rule
- Không đọc implementation khi viết test (nếu file đã tồn tại → báo user, không tự sửa expected)
- Expected từ spec, không từ chạy code