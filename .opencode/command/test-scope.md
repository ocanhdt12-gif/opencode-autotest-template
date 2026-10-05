# /test-scope — Test đúng phần vừa sửa (luồng 2)

Đọc `.context/test-scope.json` (do template DEV sinh sau bug fix/feature) → test chỉ phạm vi đã đổi.

## Cách dùng
```
/test-scope                    → đọc .context/test-scope.json mặc định
/test-scope <path-to-scope>    → dùng file scope khác
```

## Flow
1. Gọi `scope-planner` → đọc + validate scope, phân loại direct / dependents / regression / acceptance
2. `test-writer` sinh **test mới** cho `impact.direct` + `acceptance`
3. Chạy `dependents` (test lại) — bổ sung nếu thiếu
4. Chạy `impact.regression` — chuyển sang `/regression` nếu cần chuyên sâu
5. `risk: high` → bắt buộc mutation verify
6. Báo cáo: test mới / test lại pass-fail, ghi chú

## Không có scope file?
- Hỏi user: chạy full (`/autotest --full`) hay chỉ module X?
- Không tự đoán rộng

## Rule
- Không bỏ sót direct/dependents (trừ khi ghi lý do)
- Test mới vẫn phải qua quality gate (không test vô hại)