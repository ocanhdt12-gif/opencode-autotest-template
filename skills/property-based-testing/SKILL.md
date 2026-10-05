---
name: property-based-testing
description: "Sinh property-based test cho code — agent suy property từ signature/type/docs rồi dùng Hypothesis (Python) hoặc fast-check (TS) fuzz input để bắt bug thật. Dùng khi viết test cho hàm có input đa dạng, logic phức tạp, hoặc muốn test mạnh hơn unit thường."
---

# Property-Based Testing (Curated)

> Nguồn: Hypothesis docs (hypothesis.readthedocs.io) · fast-check (github.com/fast-check) · Anthropic "Finding bugs with Claude and property-based testing" (01/2026 — agent viết property test bắt được bug trong NumPy/SciPy).

## Vì sao dùng

Unit test với input cố định bỏ sót input bất thường. Property-based test khai báo **một tính chất luôn đúng** → framework tự sinh hàng trăm/nghìn input tìm phản ví dụ → bắt được bug mà không cần đoán input.

## Quy trình agent (theo Anthropic — verified)

1. **Đọc và hiểu target**: đọc code, type annotations, docstring, tên hàm, comment, cách nó liên quan codebase.
2. **Đề xuất properties**: tính chất từ spec (R-xx) hoặc từ ngữ nghĩa (vd: `sorted(x)` luôn là list đã sắp; round-trip encode/decode = gốc; idempotent; range giữ nguyên...).
3. **Viết property test**:
   - Python: Hypothesis `@given(st.integers(), ...)`
   - TS/JS: fast-check `fc.property(fc.integer(), ...)`
4. **Chạy + phản ánh**:
   - Fail → bug THẬT trong code, hay property sai (test cần sửa)?
   - Pass → test có đang test gì đáng giá không, hay trivially pass (vd wrapped trong try-catch)?
5. **Bug thật** → báo qua test-reflector → phân loại (bug-code → loop agent fix).

## Mẫu

Python:
```python
from hypothesis import given, strategies as st

@given(st.lists(st.integers()))
def test_sort_is_idempotent(xs):
    once = sorted(xs)
    assert sorted(once) == once  # round-trip / idempotent

@given(st.text())
def test_slugify_roundtrip(s):
    slug = slugify(s)
    assert slugify(slug) == slug
```

TS (fast-check):
```ts
import { test } from 'vitest';
import fc from 'fast-check';
test('sort is idempotent', () => {
  fc.assert(fc.property(fc.array(fc.integer()), xs => {
    const once = [...xs].sort((a,b)=>a-b);
    return [...once].sort((a,b)=>a-b).join() === once.join();
  }));
});
```

## Gate
- [ ] Property thật sự phản ánh spec (không phải "giữ nguyên dạng" tầm thường)
- [ ] Không wrap cả test trong try-catch (nuốt fail là mất ý nghĩa)
- [ ] Seed/replay để deterministic trong CI (Hypothesis auto DB replay)