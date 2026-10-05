# /characterize — Khóa behavior legacy bằng golden test (Nhánh B)

Sinh characterization/golden test cho code cũ chưa test — an toàn để sửa/refactor.

## Cách dùng
```
/characterize <path>      → path file hoặc module cần khóa behavior
```

## Flow
1. Gọi subagent `characterization-writer` → đọc code, snapshot behavior, scrub unstable fields
2. Bổ sung theo coverage (nhánh chưa chạm)
3. **Mutation verify**: cố tình phá code → test PHẢI fail (nếu không, test vô nghĩa)
4. Output `.context/characterization-report.md` + golden tests
5. **`/test-register characterization`** — golden test vào bộ hoàn chỉnh
6. Báo các bug cũ tiềm ẩn phát hiện (không tự sửa)

## Rule
- Chỉ thêm test, KHÔNG đổi behavior code
- Nếu mutation không bị test bắt → thêm test tới khi bắt được
- Golden test **append vào bộ test hoàn chỉnh** — không dựng suite riêng