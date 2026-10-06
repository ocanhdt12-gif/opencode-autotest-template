---
name: test-scope-contract
description: "Hợp đồng bàn giao test-scope (.spec-cache/spec/test-scope/current.json) giữa template DEV (sinh sau khi sửa code) và template AUTOTEST (đọc để biết cần test gì) — có specVersion + scopeVersion để test biết đang cover đến đâu. Dùng khi chạy /autotest hoặc cần biết phạm vi test sau thay đổi."
---

# Test Scope Contract (DEV ⇄ AUTOTEST) — versioned

Hợp đồng để template DEV báo cho template AUTOTEST **cần test cái gì** sau mỗi thay đổi — có version để test biết đang cover đến đâu.

## Ai sinh, ai dùng

| Vai | Ai | Khi nào | Ghi vào |
|---|---|---|---|
| **Producer** | template DEV (`builder`, sau bug fix / feature) | cuối mỗi lần sửa | `.spec-cache/spec/test-scope/current.json` |
| **Consumer** | template AUTOTEST (`scope-planner` + `/autotest`) | giai đoạn 0 của `/autotest` | đọc file trên + ghi `.context/test-status.json` (ở giai đoạn 3) |

## Vị trí & schema

Xem `docs/SPEC_VERSIONING.md` + `docs/FLOWS.md`. Điểm quan trọng:
- File scope: **`.spec-cache/spec/test-scope/current.json`** (KHÔNG phải `.context/test-scope.json`)
- Có **`specVersion`** (bám `.spec-cache/SPECIFICATIONS.md` version) + **`scopeVersion`** (lần sinh thứ mấy)
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
**Quy tắc:** `.spec-cache/SPECIFICATIONS.md` version > `specVersionCovered` → còn phần spec mới chưa cover → chạy `/autotest`.

## Cách AUTOTEST dùng

**`/autotest` (test tính năng MỚI)** — giai đoạn 0 đọc `.spec-cache/spec/test-scope/current.json` để xác định phạm vi ưu tiên:
- `impact.direct` → sinh test mới / cập nhật test cũ
- `impact.dependents` → test lại (bổ sung nếu thiếu)
- `impact.regression` → chạy lại test đã có
- `acceptance` → mỗi tiêu chí ≥1 test

Sau đó chạy ngầm (headless, chỉ lưu log) → autotest browser (headed, giữ browser mở) → cập nhật tiến độ.
