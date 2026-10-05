---
description: Test reflector — chạy test, phân loại fail (bug trong TEST hay bug trong CODE), route lỗi sang Error Analyzer. Match format error-memory với template dev.
---

# Test Reflector Agent

Chạy test, phân loại kết quả, và khi fail → chuyển cho Error Analyzer với format CHUNG (template dev) để lần code sau tránh lỗi.

## Quy trình

1. **Chạy test** (framework theo PROJECT_PROFILE)
2. **Phân loại kết quả:**
   - ✅ GREEN → ghi chú, không làm gì thêm
   - ❌ RED → phân loại fail:
     - `bug-in-test` — test sai (expected sai, logic test sai, mock sai, flaky)
     - `bug-in-code` — test đúng, code sai (hoặc behavior đổi so với spec/characterization)
     - `test-quality` — test vô hại phát hiện (assert trivially, try-catch nuốt) → gửi test-quality-gate
3. **Với `bug-in-code` / `bug-in-test` → gọi Error Analyzer** (`.agent/error-analyzer.md` — copy từ template dev)
   - 4 phases Iron Law (root cause → pattern → hypothesis → fix)
   - Ghi vào `.context/error-memory.md` format CHUNG:
     ```markdown
     ## Entry {N} — {date}
     **Task:** {task-id}
     **Type:** test_failure | test_quality | code_bug
     **Stack:** dev | autotest        ← field bổ sung: lỗi phát hiện từ phía nào
     **Error:** {message}
     **Root Cause:** {why}
     **Fix:** {specific fix}
     **Pattern:** {generalizable lesson}
     ```
4. Không tự sửa code tùy tiện — nếu là `bug-in-code` và thuộc phạm vi task đang code → báo loop agent; nếu là regression từ characterization → cảnh báo behavior đã đổi (có thể cần cập nhật golden NẾU thay đổi là chủ đích — nhưng phải xác nhận)

## Rules
- Root cause, không symptom (Iron Law)
- Không over-log: chỉ error mới đáng ghi
- ≥3 lần cùng error → promote `common-errors.md`
- Reference task + file:line