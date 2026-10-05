---
name: error-memory
description: "Format error-memory CHUNG với template dev — lỗi từ test (test_failure/test_quality/code_bug) ghi vào .context/error-memory.md đúng format, field stack: dev|autotest; lần code sau (cả 2 template) đọc để tránh lặp lỗi. Dùng khi test fail, muốn tránh lỗi đã gặp, hoặc log pattern mới."
---

# Error Memory (format chung với template dev)

> ⚠️ **MATCH BẮT BUỘC với template dev** — để 2 template đọc/ghi chung `.context/error-memory.md`, kỹ thuật và format phải GIỐNG HỆT `.agent/error-analyzer.md` (đã copy từ template dev).

## Vì sao dùng

Query: error bộ test log ra → Error Analyzer (template dev) → khi code sẽ tránh được lỗi này cho lần code sau. Cầu nối: lỗi phát hiện từ TEST phải vào cùng error-memory mà CODE đọc.

## Format (giống template dev + 1 field)

```markdown
# Error Memory

## Entry {N} — {date}

**Task:** {layer-X/task-YY hoặc test case}
**Type:** test_failure | test_quality | code_bug
**Stack:** dev | autotest        ← BỔ SUNG: lỗi phát hiện từ phía nào
**Error:** {concise description}

**Root Cause:**
{1-2 sentences WHY}

**Fix:**
{Specific fix}

**Pattern:**
{Generalizable lesson, e.g., "Always wrap async DB calls in try/catch"}
```

## Quy tắc

1. **Root cause, không symptom** (Iron Law 4 phases — đọc `skills/superpowers/systematic-debugging` nếu có)
2. Chỉ log error MỚI đáng nhớ — không log typo/1-off
3. Cùng error ≥3 lần → promote `common-errors.md`
4. Luôn ghi `Stack:` để biết lỗi từ test hay từ code
5. **Lần code sau:** loop agent đọc error-memory trước khi code — đã là rule của template dev, giờ autotest cũng vậy → 2 chiều tránh lỗi chéo

## Hành vi khi gặp error cũ
- Tra error-memory trước → nếu có pattern → áp fix đã biết (không điều tra lại từ đầu)