---
name: complete-test-suite
description: "Nuôi MỘT bộ test hoàn chỉnh — mọi luồng test (full/test-scope/regression/manual/from-cases/characterize) TỰ ĐỘNG bổ sung/cập nhật vào cùng bộ (tests/ + test-registry.json) khi chạy xong, không cần gõ command; bộ đó để retest tính năng cũ + test feature mới. Dùng khi chạy bất kỳ luồng test nào hoặc khi cần biết/đăng ký test vào bộ hoàn chỉnh."
---

# Một bộ test hoàn chỉnh (nguyên tắc gốc)

Test chỉ có **1 bộ duy nhất** trong repo. Mọi luồng chỉ để **nuôi** bộ này — vừa **retest tính năng cũ**, vừa **test feature mới**. Không bao giờ có suite riêng cho từng luồng.

> ⚙️ **TỰ ĐỘNG — không có command riêng.** Cuối MỌI luồng test, agent **tự chạy** bước đăng ký dưới đây. Người dùng KHÔNG gõ gì thêm. Đây là 1 bước ẩn của luồng, không phải thao tác tay.

## Ai ghi gì (bộ hoàn chỉnh)

| File | Vai trò | Ai ghi |
|---|---|---|
| `tests/` (theo PROJECT_PROFILE) | Toàn bộ test thật — **1 bộ duy nhất** | mọi luồng (tự động) |
| `test-registry.json` (root repo TEST) | Manifest: mọi test + `refs` + `origin` + `status` + `lastRunAt` | bước đăng ký tự động (cuối mọi luồng) |
| `.context/coverage.json` | Board độ phủ theo req (req → covered/failing + `testRef`) | bước đăng ký tự động |
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

## Cơ chế tự động (chạy cuối MỌI luồng)

1. **Thu thập** test mới/đổi từ luồng vừa chạy (file + test name + `refs` + kết quả).
2. **Kiểm tra trùng** (BẮT BUỘC, trước khi ghi) — tra `test-registry.json` + `tests/`:
   - Entry đã có **cùng `refs` VÀ cùng behavior** (cùng test name/logic) → **cập nhật** (`status`, `lastRunAt`, `regression`, `origin` giữ nguyên gốc).
   - Entry chưa có → **append**.
   - Test mới phủ requirement đã có test khác phủ → ghi chú, ưu tiên **bổ sung** vào test hiện có thay vì tạo bản sao.
   - **Không** bao giờ thêm 2 entry trùng `id`/`refs`+behavior.
3. **Ghi/append vào `tests/`** — test mới vào file hiện có theo module, hoặc file mới nhưng thuộc `tests/`; KHÔNG dựng thư mục/suite riêng cho luồng.
4. **Cập nhật board** `.context/coverage.json`: req trong `refs` → `covered`/`failing` + `testRef` + `lastRunAt`; req mới chưa có test → `pending`/`untested`.
5. **Đối chiếu version:** `.spec-cache/SPECIFICATIONS.md` version > `specVersionCovered` (test-status) → còn phần spec mới chưa cover → nhắc chạy `/test-scope` hoặc `/autotest --full`.

## Bất biến

- **Tự động:** không cần người dùng gõ command; mỗi luồng tự kết thúc bằng bước này.
- **Additive:** sau mỗi luồng, bộ test **chỉ tăng/cập nhật**, không bị luồng khác ghi đè/xoá.
- **Không trùng:** kiểm tra trùng trước khi ghi; cùng `refs`+behavior → cập nhật, không thêm bản sao.
- **Không suite song song:** cấm tạo `tests-full/`, `tests-regression/`... tách khỏi bộ chính.
- **Regression = 1 phần của bộ:** luồng 3 chỉ chạy lại entry `regression: true` (hoặc `--all`), cập nhật `status`/`lastRunAt` — không sinh test mới trừ khi `test-reflector` phát hiện thiếu.
- **Traceable:** mọi entry có `refs` — bám `R-xx` (spec) hoặc case id (user/manual).

## Nối vào các luồng (tự động, không command)

| Luồng | Sinh gì | Vào bộ hoàn chỉnh (tự động) |
|---|---|---|
| 1 `/autotest --full` | test mọi `R-xx` | ✅ append (`origin=full`) |
| 2 `/test-scope` | test cho `direct`/`acceptance` vừa đổi | ✅ append/cập nhật (`origin=test-scope`) |
| 3 `/regression` | retest luồng cũ | ♻️ chỉ cập nhật `status`/`lastRunAt` |
| 4 `/capture-manual` | case tay → auto | ✅ append (`origin=manual`) + `regression=true` |
| 5 `/from-cases` | test theo case user | ✅ append (`origin=from-cases`) + `regression=true` |
| B `/characterize` | golden test legacy | ✅ append (`origin=characterization`) |

## Output (mỗi luồng tự báo)
- `test-registry.json` cập nhật (kèm kết quả kiểm tra trùng)
- `.context/coverage.json` cập nhật
- Báo: thêm N test mới · cập nhật M · trùng bỏ qua K · req còn `pending/untested`
