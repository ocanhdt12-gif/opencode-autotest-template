# /test-register — Ghi test vào bộ test hoàn chỉnh (mọi luồng tự gọi)

Chốt mọi luồng test: đưa test mới/đổi vào **MỘT bộ test hoàn chỉnh** (dùng để retest tính năng cũ + test feature mới). Không tạo suite song song.

> Chi tiết: `skills/complete-test-suite` + `docs/FLOWS.md` (mục "Đăng ký bộ test hoàn chỉnh").

## Cách dùng
```
/test-register                 → ghi tất cả test mới/đổi của luồng vừa chạy
/test-register <origin>        → ghi kèm nhãn nguồn (full|test-scope|regression|manual|from-cases|characterization)
```

## Flow
1. Thu thập test mới/đổi từ luồng vừa chạy (file · test name · `refs` · kết quả)
2. Tra `test-registry.json`: cùng `refs` + cùng behavior → **cập nhật**; chưa có → **append**
3. Test mới **append vào `tests/`** hiện có (theo module) — KHÔNG dựng suite riêng cho luồng
4. Cập nhật `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`)
5. Đối chiếu version: spec > `specVersionCovered` → nhắc `/test-scope` hoặc `/autotest --full`

## Rule
- **Additive** — bộ test chỉ tăng/cập nhật, không ghi đè/xoá của luồng khác
- **Không suite song song** — cấm `tests-full/`, `tests-regression/`... tách khỏi bộ chính
- **Traceable** — mọi entry có `refs` (R-xx hoặc case id)
- Luồng 3 (regression) chỉ cập nhật `status`/`lastRunAt`, không sinh test mới

## Output
- `test-registry.json` + `.context/coverage.json` cập nhật
- Báo: +N test mới · M cập nhật · req còn `pending/untested`
