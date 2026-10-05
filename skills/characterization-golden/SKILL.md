---
name: characterization-golden
description: "Sinh characterization/golden test cho legacy code chưa có test — khóa behavior hiện tại (snapshot + scrub unstable fields) để refactor an toàn; mutation verify test bắt được. Dùng khi '/characterize', sửa/refactor code cũ, hoặc nhận code gen (Lovable) chưa test. Nguồn: characterization-test-generator, JetBrains Junie, understandlegacycode."
---

# Characterization / Golden Tests (Curated)

> Nguồn: bkitduy/characterization-test-generator (MIT) · JetBrains Junie "Work with Legacy Code Efficiently" (08/2026) · understandlegacycode.com 3-step recipe.

## Vì sao dùng

Code cũ chưa test: sửa/refactor = mò. Characterization test **khóa behavior HIỆN TẠI** (không phải behavior "đúng") → đổi code mà test fail = behavior đổi = có bug hoặc đổi chủ đích. Không chứng minh code đúng — chứng minh **hành vi không đổi**.

## 3 bước (understandlegacycode — verified)

1. **📸 Snapshot** — ghi lại mọi output thật của code (input điển hình + edge case). Expected = output thật (golden).
2. **✅ Coverage** — chạy coverage → nhánh chưa chạm → thêm input.
3. **👽 Mutation** — cố tình phá code → test PHẢI fail. Test không bắt = vô nghĩa → bổ sung.

## Scrub unstable fields (bắt buộc)

Timestamp, id tự sinh, random, path tuyệt đối, thứ tự dict → normalize/ignore:
- Regex strip: `\d{4}-\d{2}-\d{2}T...` → `[TS]`
- Mock clock/random
- Custom serializer bỏ field unstable
- Sort output trước khi so sánh

## Mẫu (VCR/golden style)

```python
def test_discount_golden():
    # golden: output khóa từ lần chạy đầu (đã scrub timestamp)
    assert normalize(calc_discount(100, "10%")) == normalize(GOLDEN["10pct"])
```

## Prompt agent chuẩn (Junie — verified)

"Generate characterization tests for `calculateDiscount()` that capture its current behavior across the identifiable input cases, including edge cases. Do not change any behavior, only document it."

## Gate
- [ ] Snapshot đủ input + edge case
- [ ] Unstable fields đã scrub
- [ ] Mutation verify: phá code → test fail
- [ ] KHÔNG đổi behavior code (chỉ thêm test)
- [ ] Bug cũ phát hiện → báo để xử lý riêng, không tự sửa