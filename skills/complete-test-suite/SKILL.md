---
name: complete-test-suite
description: "Nuôi MỘT bộ test hoàn chỉnh — mọi luồng test (full/test-scope/regression/manual/from-cases/characterize) chỉ bổ sung/cập nhật vào cùng bộ (tests/ + test-registry.json), không tạo suite song song; bộ đó để retest tính năng cũ + test feature mới. Dùng khi chạy bất kỳ luồng test nào hoặc khi cần biết/đăng ký test vào bộ hoàn chỉnh."
---

# Một bộ test hoàn chỉnh (nguyên tắc gốc)

Test chỉ có **1 bộ duy nhất** trong repo. Mọi luồng chỉ để **nuôi** bộ này — vừa **retest tính năng cũ**, vừa **test feature mới**. Không bao giờ có suite riêng cho từng luồng.

## Ai ghi gì (bộ hoàn chỉnh)

| File | Vai trò | Ai ghi |
|---|---|---|
| `tests/` (theo PROJECT_PROFILE) | Toàn bộ test thật — **1 bộ duy nhất** | mọi luồng |
| `test-registry.json` (root repo TEST) | Manifest: mọi test + `refs` + `origin` + `status` + `lastRunAt` | `/test-register` sau mọi luồng |
| `.context/coverage.json` | Board độ phủ theo req (req → covered/failing + `testRef`) | `/coverage` + `/test-register` |
| `.context/test-status.json` | Version đã cover (specVersion/scopeVersion) | `/test-scope` + `/spec-link` |

> Registry trả lời "test nào đang giữ phần nào"; board trả lời "req nào đã/chưa test". Cùng một bộ.

## Schema `test-registry.json`

```jsonc
{
  "updatedAt": "2026-10-05T14:30:00+07:00",
  "tests": [
    { "id": "tests/auth.test.ts::login_r01",
      "file": "tests/auth.test.ts",
      "refs": ["R-01"],                       // R-xx (spec) hoặc case id (user/manual)
      "origin": "full | test-scope | regression | manual | from-cases | characterization",
      "tags": ["auth", "smoke"],
      "regression": true,                     // có nằm trong bộ retest luồng cũ không
      "status": "pass | fail | skip",
      "lastRunAt": "2026-10-05T14:35:00+07:00" }
  ]
}
```

## Quy trình `/test-register` (mọi luồng kết thúc ở đây)

1. **Thu thập** test mới/đổi từ luồng vừa chạy (file + test name + `refs` + kết quả chạy).
2. **Tra registry trước khi thêm:** đã có entry cùng `refs` + cùng behavior → **cập nhật** (`status`, `lastRunAt`, `regression`); nếu chưa có → **append** entry mới. Không thêm bản sao.
3. **Ghi/append vào `tests/`** — test mới vào file hiện có theo module, hoặc file mới nhưng thuộc `tests/`; KHÔNG dựng thư mục/suite riêng cho luồng.
4. **Cập nhật board** `.context/coverage.json`: req trong `refs` → `covered`/`failing` + `testRef` + `lastRunAt`; req mới chưa có test → `pending`/`untested`.
5. **Đối chiếu version:** `.spec-cache/SPECIFICATIONS.md` version > `specVersionCovered` (test-status) → còn phần spec mới chưa cover → nhắc chạy `/test-scope` hoặc `/autotest --full`.

## Bất biến

- **Additive:** sau mỗi luồng, bộ test **chỉ tăng/cập nhật**, không bị luồng khác ghi đè/xoá.
- **Không suite song song:** cấm tạo `tests-full/`, `tests-regression/`... tách khỏi bộ chính.
- **Regression = 1 phần của bộ:** luồng 3 chỉ chạy lại entry `regression: true` (hoặc `--all`), cập nhật `status`/`lastRunAt` — không sinh test mới trừ khi `test-reflector` phát hiện thiếu.
- **Traceable:** mọi entry có `refs` — bám `R-xx` (spec) hoặc case id (user/manual).

## Nối vào 5 luồng

| Luồng | Sinh gì | Vào bộ hoàn chỉnh |
|---|---|---|
| 1 `/autotest --full` | test mọi `R-xx` | ✅ append (`origin=full`) |
| 2 `/test-scope` | test cho `direct`/`acceptance` vừa đổi | ✅ append/cập nhật (`origin=test-scope`) |
| 3 `/regression` | retest luồng cũ | ♻️ chỉ cập nhật `status`/`lastRunAt` |
| 4 `/capture-manual` | case tay → auto | ✅ append (`origin=manual`) + `regression=true` |
| 5 `/from-cases` | test theo case user | ✅ append (`origin=from-cases`) + `regression=true` |
| B `/characterize` | golden test legacy | ✅ append (`origin=characterization`) |

## Output
- `test-registry.json` cập nhật
- `.context/coverage.json` cập nhật
- Báo: thêm N test mới · cập nhật M · req còn `pending/untested`
