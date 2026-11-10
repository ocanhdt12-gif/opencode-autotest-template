# .context/manual-cases/ — CASE TEST TAY (chưa chuyển auto)

Nơi chứa các **case đã test TAY** nhưng **chưa** chuyển thành auto test. Lệnh `/capture-manual <case-id>` đọc `.context/manual-cases/<case-id>.md` → sinh auto test → chạy XANH → đánh dấu `regression=true`.

**Khi nào có file ở đây:**
- User test tay 1 case xong (vd kiểm thủ công trên staging) và muốn Eve chuyển nó thành auto test về sau.
- Session mới `/autotest` sẽ **check thư mục này** (xem có case tay nào chưa auto hoá không).

**Format 1 file `<case-id>.md`:**

```markdown
# Manual Case — <case-id>

- **Requirement:** R-xx (nếu map được)
- **Module:** <module>
- **Môi trường:** staging / local / ...
- **Input:** <dữ liệu / tiền điều kiện>
- **Steps:** (các bước đã làm tay)
  1. ...
  2. ...
- **Expected:** <kết quả mong đợi>
- **Actual (manual):** <kết quả khi test tay — pass/fail>
- **Status:** captured   # captured → (sau /capture-manual) → automated
```

> Sau khi `/capture-manual` chuyển thành auto test XANH, case được đăng ký vào bộ test hoàn chỉnh (`test-registry.json`) với `regression=true`; file ở đây giữ lại làm nguồn gốc (traceability).
