---
name: test-scope-contract
description: "Hợp đồng bàn giao test-scope.json giữa template DEV (sinh sau khi sửa code) và template AUTOTEST (đọc để biết cần test gì). Định nghĩa format + producer/consumer. Dùng khi /test-scope, /regression, hoặc khi cần biết phạm vi test sau một thay đổi."
---

# Test Scope Contract (DEV ⇄ AUTOTEST)

Hợp đồng để template DEV báo cho template AUTOTEST **cần test cái gì** sau mỗi thay đổi — anh không phải tự mò.

## Ai sinh, ai dùng

| Vai | Ai | Khi nào |
|---|---|---|
| **Producer** | template DEV (`builder`, sau bug fix / feature) | cuối mỗi lần sửa code → ghi `.context/test-scope.json` |
| **Consumer** | template AUTOTEST (`scope-planner` + commands) | `/test-scope`, `/regression` |

## Format (bắt buộc — 2 template thống nhất)

Xem `docs/FLOWS.md`. Tóm tắt field quan trọng:
- `trigger`: `initial-build | bug-fix | feature-update`
- `specRefs`: requirement R-xx liên quan (traceability về spec)
- `changed.files` / `changed.modules`
- `impact.direct` — hành vi đổi trực tiếp → **test mới/sửa**
- `impact.dependents` — phụ thuộc có thể vỡ → **test lại**
- `impact.regression` — luồng cũ cần retest
- `acceptance` — tiêu chí nghiệm thu (test phải cover)
- `risk` — quyết định độ sâu test

## Cách AUTOTEST dùng

1. `/test-scope`: test `direct` + `dependents` + `acceptance` (không test cả repo)
2. `/regression`: chạy lại test đã có cho `regression` + `dependents`
3. `risk: high` → bắt buộc mutation verify + mở rộng phạm vi; `low` → scope hẹp

## Nếu thiếu test-scope.json
- Không có file → hỏi: "chạy full hay chỉ ranh giới module X?" (không tự đoán rộng)
- File cũ (quá 1 workItem) → cảnh báo stale, đề nghị dev sinh lại

## Validate
- `specRefs` phải tồn tại trong `SPECIFICATIONS.md` (nếu không → cảnh báo lệch spec)
- `changed.files` phải là file thật trong repo
- Thiếu `acceptance` → nhắc dev bổ sung (tiêu chí test)