# FLOWS — Luồng chạy test (chuẩn hoá)

> Luồng chạy test **DUY NHẤT**: `/autotest` = **chạy ngầm (headless) trước → autotest browser (headed) sau → cập nhật tiến độ**. Hợp đồng bàn giao `test-scope` giữa template DEV (viết code) và template AUTOTEST (viết/chạy test).

## ⭐ Một bộ test hoàn chỉnh (nguyên tắc gốc)

**Mọi test đều phục vụ MỘT đích: nuôi bộ test hoàn chỉnh.** Chỉ có **1 bộ duy nhất** trong repo — vừa **retest tính năng cũ**, vừa **test feature mới**. Không có suite riêng cho từng luồng.

- Đăng ký trung tâm: `tests/` (hoặc theo PROJECT_PROFILE) + `test-registry.json` (manifest mọi test + nguồn gốc).
- Test mới **append/cập nhật** vào bộ này, ghi `origin` — không bao giờ tạo suite song song.
- Retest = chạy lại đúng bộ này theo `impact.regression`; feature mới cũng vào cùng bộ.
- `test-registry.json` + `.context/coverage.json` là 2 mặt của cùng bộ: cái nào tồn tại/đã chạy (board) · test nào đang giữ phần đó (`testRef`).

## Hợp đồng bàn giao: `.spec-cache/spec/test-scope/current.json` (versioned)

**Template DEV sinh ra** sau mỗi bug fix / feature update. **Template AUTOTEST đọc** để biết cần test cái gì — không phải tự mò.

> 📌 Vị trí + version scheme: xem `docs/SPEC_VERSIONING.md`. Scope nằm trong `spec/test-scope/` (cạnh spec), **có `specVersion` + `scopeVersion`** để test biết đang cover đến đâu.

```jsonc
{
  "specVersion": "1.2.0",       // scope này bám spec version nào
  "scopeVersion": 3,             // lần sinh thứ mấy (tăng mỗi lần)
  "generatedAt": "2026-10-05T13:32:00+07:00",
  "trigger": "initial-build | bug-fix | feature-update",
  "workItem": "bug-login-timeout | feature-stripe-selfserve",
  "specRefs": ["R-01", "R-05"],              // requirement liên quan
  "changed": {
    "files": ["src/auth/login.ts", "src/api/session.ts"],
    "modules": ["auth", "session"]
  },
  "impact": {
    "direct": ["auth.login", "auth.refresh"],     // hành vi đổi TRỰC TIẾP
    "dependents": ["portal.session"],             // module phụ thuộc → có thể vỡ
    "regression": ["payment.checkout"]            // luồng cũ cần retest lại
  },
  "risk": "low | medium | high",
  "acceptance": ["login thành công < 2s", "token refresh không mất session"],
  "notes": "đổi timeout 30s → 10s, ảnh hưởng mọi call auth"
}
```

**Producer:** template DEV — bắt buộc, không bỏ.
**Consumer:** template AUTOTEST (`scope-planner` + `/autotest`) — đọc để chọn test, ghi `.context/test-status.json` để theo dõi version đã cover.

---

## Luồng chạy DUY NHẤT — `/autotest` (3 giai đoạn)

> Cơ bản: **chạy test ngầm 1 lượt trước** (headless, không bung browser, cho nhanh), **rồi mới autotest với browser** (headed, bung hẳn ra cho user theo dõi).

### Giai đoạn 0 — Chuẩn bị
- `/spec-link --sync` — pull spec mới nhất; chưa link → hỏi link
- Đọc `.spec-cache/spec/test-scope/current.json` → xác định **phạm vi ưu tiên**: `impact.direct` + `dependents` + `regression` + `acceptance`
- `spec-validator` PASS trước khi sinh/chạy test

### Giai đoạn 1 — Chạy NGẦM (headless, KHÔNG bung browser)
- **Chạy test suite như bình thường**, chế độ headless cho nhanh:
  - unit/integration: `npm test` / `npx vitest run` / `pytest`
  - E2E (nếu muốn soi trước): `npx playwright test` (mặc định headless)
- Mục đích: kết quả **nhanh** + phát hiện lỗi cơ bản trước khi mở browser (chậm hơn)
- **LƯU kết quả** vào `.context/test-results/headless-run.json` — **CHỈ LƯU LOG, KHÔNG cập nhật tiến độ**
- Fail → `test-reflector` phân loại (`bug-in-test` / `bug-in-code`)

### Giai đoạn 2 — AUTOTEST với browser (HEADED — bung hẳn ra)
- Chạy E2E Playwright chế độ **headed** (`headless: false`) → **browser mở thật** để user theo dõi từng thao tác
  - `npx playwright test --headed` + `use: { headless: false }`; tuỳ chọn `slowMo` cho user kịp nhìn
- **Đến bước cuối cùng:**
  1. **LƯU kết quả** (screenshot + report + json) vào `.context/test-results/browser-run.json`
  2. **KHÔNG đóng browser** — để lại màn hình kết quả:
     - `await page.pause()` ở cuối test (Playwright Inspector, treo browser), hoặc
     - fixture `keepBrowserOpen` chạy khi env `LEAVE_BROWSER_OPEN=1` (auto `page.pause()` cho mọi page khi không headless)
- `test-reflector` phân loại fail (`bug-in-test` / `bug-in-code`)

### Giai đoạn 3 — Cập nhật tiến độ (TỰ ĐỘNG — CHỈ sau Giai đoạn 2)
1. **Kiểm tra trùng** rồi ghi/cập nhật `test-registry.json`
2. Cập nhật tiến độ `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`)
3. Cập nhật `.context/test-status.json` (`specVersionCovered` / `scopeVersionCovered`)
- Không gõ command nào. **Chạy ngầm KHÔNG cập nhật tiến độ.**

### Biến thể
- `/autotest <module>` — giới hạn 1 module
- `/autotest --no-browser` — **chỉ chạy ngầm** (khi chỉ cần kết quả nhanh, không cần xem browser)

---

## Bảng tóm tắt

| Giai đoạn | Command/hành động | Đọc/ghi gì | Cập nhật tiến độ? |
|---|---|---|---|
| 0 Chuẩn bị | `/spec-link --sync` | `.spec-cache/spec/test-scope/current.json` | — |
| 1 Chạy ngầm (headless) | suite test thường | ghi `.context/test-results/headless-run.json` | ❌ chỉ lưu log |
| 2 Browser (headed) | `npx playwright test --headed` | ghi `.context/test-results/browser-run.json` | ❌ |
| 3 Cập nhật tiến độ | tự động | `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` | ✅ |

## Vòng lặp thực tế

```
[DEV] code lần đầu / fix bug / feature ──► sinh .spec-cache/spec/test-scope/current.json
                                                        │
                                                        ▼
[TEST] /autotest ──► (1) chạy NGẦM headless (chỉ lưu log)
                       (2) autotest BROWSER headed (bung hẳn, giữ browser mở)
                       (3) CẬP NHẬT TIẾN ĐỘ (sau khi browser xong)
                                                        │
[TEST TAY] case pass ──► /capture-manual ──► auto test ─┤
[USER] đưa test case ──► /from-cases ──────► auto test ─┤
[LEGACY] ─────────────► /characterize ─────► golden test ┘
                                                        │
                                                        ▼
                            📦 BỘ TEST HOÀN CHỈNH (tests/ + test-registry.json)
                                    └──► retest luồng cũ + test feature mới
```

---

## Đăng ký bộ test + cập nhật tiến độ test (TỰ ĐỘNG, chỉ sau browser)

Sau khi **browser test xong** (Giai đoạn 3), agent **tự chạy** các việc sau (giữ 1 bộ duy nhất, không tạo suite song song). Người dùng KHÔNG gõ command:

1. **Ghi manifest** `test-registry.json` (root repo TEST) — mỗi test 1 entry:
   ```jsonc
   {
     "updatedAt": "2026-10-05T14:30:00+07:00",
     "tests": [
       { "id": "tests/auth.test.ts::login_r01", "file": "tests/auth.test.ts",
         "refs": ["R-01"], "requirement": "R-01",
         "origin": "autotest | manual | from-cases | characterization",
         "tags": ["auth", "smoke"], "regression": true,
         "status": "pass", "lastRunAt": "2026-10-05T14:35:00+07:00" }
     ]
   }
   ```
2. **Không tạo file/suite test mới tách rời** — test mới append vào file hiện có theo module, hoặc thêm file nhưng cùng `tests/` và cùng được liệt kê trong registry.
3. **Cập nhật tiến độ test**: danh sách req lấy từ spec (`.spec-cache/SPECIFICATIONS.md` — đọc, không ghi); board thiếu → khởi tạo danh sách req từ spec; req trong `refs` vừa chạy → `covered`/`failing` + `testRef` + `lastRunAt`; req mới trong spec → `pending`/`untested`; req bỏ khỏi spec → `n/a`. Ghi `.context/coverage.json` + `.context/test-status.json`. **Test KHÔNG tạo gì thuộc spec.**

**Rule chống trùng:** trước khi thêm test, tra `test-registry.json` — đã có test phủ cùng `requirement` + cùng behavior → **cập nhật** thay vì thêm bản sao.

**Rule tiến độ:** `/coverage` chỉ để **xem báo cáo** — tiến độ do bước tự động này cập nhật. **Spec chỉ đọc từ link** (`.spec-cache/`, read-only) — test không tạo/sinh/sửa spec.
