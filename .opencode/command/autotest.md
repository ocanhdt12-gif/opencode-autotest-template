# /autotest — Sinh test tự động từ spec

## Cách dùng
```
/autotest --full          → test TOÀN BỘ spec (luồng 1 — sau khi code lần đầu)
/autotest <module>        → test cho 1 module
/autotest --scope         → alias /test-scope (test phần vừa sửa — luồng 2)
```

## Luồng 1 — Full test lần đầu
1. Đọc `.spec-cache/SPECIFICATIONS.md` + `.agent/PROJECT_PROFILE.md` (framework test)
2. Gọi `test-writer` → viết test cho **mọi** requirement `R-xx` (unit + property-based), traceable
3. Chạy test → **ĐỎ đúng cách** (fail vì chưa có code, không phải fail vì test sai)
4. Báo user: implement code tới khi XANH
5. Sau khi code xong: `/verify-tests`
6. **Đăng ký TỰ ĐỘNG (cuối luồng)** — kiểm tra trùng rồi append test mới vào bộ test hoàn chỉnh (`test-registry.json` + board). Không cần gõ command.

## Rule
- Không đọc implementation khi viết test (file đã tồn tại → báo user, không tự sửa expected)
- Expected từ spec, không từ chạy code
- Test sinh ra **append vào bộ test hoàn chỉnh** (`tests/` + `test-registry.json`) — không dựng suite riêng
- Cuối luồng **tự động kiểm tra trùng** rồi đăng ký — không cần command riêng