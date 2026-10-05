# /test-scope — Test đúng phần vừa sửa (luồng 2)

Đọc `spec/test-scope/current.json` (do template DEV sinh, có specVersion+scopeVersion) → test chỉ phạm vi đã đổi.

## Cách dùng
```
/test-scope                    → đọc spec/test-scope/current.json
/test-scope <path>             → dùng file scope khác
```

## Flow
1. Gọi `scope-planner` → đọc + validate scope (version khớp spec? đã cover chưa?)
2. `test-writer` sinh **test mới** cho `impact.direct` + `acceptance`
3. Chạy `dependents` (test lại) — bổ sung nếu thiếu
4. Chạy `impact.regression` — chuyển `/regression` nếu cần chuyên sâu
5. `risk: high` → bắt buộc mutation verify
6. Cập nhật `.context/test-status.json` (specVersionCovered / scopeVersionCovered)
7. Báo cáo: test mới / test lại pass-fail, ghi chú

## Không có scope file?
- Hỏi user: chạy full (`/autotest --full`) hay chỉ module X?
- Không tự đoán rộng

## Rule
- Không bỏ sót direct/dependents (trừ khi ghi lý do)
- Test mới vẫn phải qua quality gate (không test vô hại)
- Nếu `scopeVersion` đã cover rồi → báo "đã cover", không test lại