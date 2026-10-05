---
name: test-scope-contract
description: "Hợp đồng bàn giao test-scope (spec/test-scope/current.json) giữa template DEV (sinh sau khi sửa code) và template AUTOTEST (đọc để biết cần test gì) — có specVersion + scopeVersion để test biết đang cover đến đâu. Dùng khi /test-scope, /regression, hoặc cần biết phạm vi test sau thay đổi."
---

# Test Scope Contract (DEV ⇄ AUTOTEST) — versioned

Hợp đồng để template DEV báo cho template AUTOTEST **cần test cái gì** sau mỗi thay đổi — có version để test biết đang cover đến đâu.

## Ai sinh, ai dùng

| Vai | Ai | Khi nào | Ghi vào |
|---|---|---|---|
| **Producer** | template DEV (`builder`, sau bug fix / feature) | cuối mỗi lần sửa | `spec/test-scope/current.json` |
| **Consumer** | template AUTOTEST (`scope-planner` + commands) | `/test-scope`, `/regression` | đọc file trên + ghi `.context/test-status.json` |

## Vị trí & schema

Xem `docs/SPEC_VERSIONING.md` + `docs/FLOWS.md`. Điểm quan trọng:
- File scope: **`spec/test-scope/current.json`** (KHÔNG phải `.context/test-scope.json`)
- Có **`specVersion`** (bám `SPECIFICATIONS.md` version) + **`scopeVersion`** (lần sinh thứ mấy)
- Archive: `spec/test-scope/archive/test-scope-<specVersion>-<scopeVersion>.json`

## Test biết "cần test đến đâu"

Template TEST ghi `.context/test-status.json`:
```jsonc
{
  "specVersionCovered": "1.2.0",
  "scopeVersionCovered": 3,
  "lastRun": "...",
  "pendingSpecVersion": null
}
```
**Quy tắc:** `SPECIFICATIONS.md` version > `specVersionCovered` → còn phần spec mới chưa cover → chạy `/test-scope` (nếu có scope) hoặc `/autotest --full` (mốc lớn).

## Cách AUTOTEST dùng

1. `/test-scope`: đọc `spec/test-scope/current.json` → test `direct` + `dependents` + `acceptance`
2. `/regression`: chạy lại test đã có cho `regression` + `dependents`
3. `risk: high` → bắt buộc mutation verify + mở rộng phạm vi; `low` → scope hẹp
4. Sau khi xong → cập nhật `.context/test-status.json`

## Nếu thiếu / stale
- Không có file → hỏi: "chạy full hay chỉ module X?" (không tự đoán rộng)
- `scopeVersion` đã cover (trong test-status) → không test lại, báo đã cover
- File cũ hơn spec version → cảnh báo stale, đề nghị dev sinh lại

## Validate
- `specRefs` phải tồn tại trong `SPECIFICATIONS.md`
- `changed.files` phải là file thật trong repo
- Thiếu `acceptance` → nhắc dev bổ sung