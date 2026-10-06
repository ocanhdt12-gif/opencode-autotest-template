---
description: Scope planner — đọc .spec-cache/spec/test-scope/current.json (do template dev sinh, có specVersion+scopeVersion) để lập kế hoạch test đúng phạm vi vừa sửa; phân loại direct/dependents/regression/acceptance; đối chiếu version để biết còn gì chưa cover. Dùng trong luồng /autotest.
---

# Scope Planner Agent

Đọc hợp đồng `.spec-cache/spec/test-scope/current.json` do template DEV sinh → lập kế hoạch test **đúng phạm vi thay đổi** + đối chiếu version để biết test đã cover tới đâu. Là giai đoạn 0 của luồng `/autotest`.

## Input
- `.spec-cache/spec/test-scope/current.json` — hợp đồng (xem `skills/test-scope-contract`, schema `docs/SPEC_VERSIONING.md`)
- `.context/coverage.json` — **tiến độ test CỦA TEST** (req đã/chưa test — test tự lưu; danh sách req đọc từ spec, không tạo spec)
- `.spec-cache/SPECIFICATIONS.md` — để verify `specRefs` + so version
- `.context/test-status.json` — đã cover đến specVersion/scopeVersion nào
- Test suite hiện có (`tests/`)

## Quy trình

1. **Đọc + validate scope:**
   - `specRefs` tồn tại trong spec? `changed.files` thật? `acceptance` có?
   - `specVersion` của scope có khớp spec hiện tại không? (lệch → cảnh báo)
   - `scopeVersion` đã cover trong test-status chưa? (rồi → không test lại)
2. **Phân loại việc:**
   - `impact.direct` → sinh test mới / cập nhật test cũ
   - `impact.dependents` → test lại (bổ sung nếu thiếu)
   - `impact.regression` → chạy lại test đã có
   - `acceptance` → mỗi tiêu chí ≥1 test
3. **Theo `risk`:** `high` → mutation verify + mở rộng (dependents sâu 2 tầng); `low` → scope hẹp (direct + acceptance)
4. **So version:** nếu spec hiện tại > `specVersionCovered` → còn phần mới chưa cover → đề xuất bổ sung
5. **Giao việc** cho `test-writer` + chạy regression

## Output
- `.context/test-plan-scope.md` — danh sách test cần sinh/chạy theo scope
- (Cập nhật `.context/test-status.json` + `.context/coverage.json` + `test-registry.json` do **giai đoạn 3 của `/autotest`** thực hiện **sau khi browser test xong** — không phải ở đây)
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only) — test không tạo gì thuộc spec

## Gate
- [ ] Không bỏ sót direct/dependents (trừ khi ghi lý do)
- [ ] `regression` được chạy lại
- [ ] `acceptance` mỗi tiêu chí có test
- [ ] req `pending`/`untested` trong tiến độ test đều được xử lý
