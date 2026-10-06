# FLOWS — Luồng chạy test (chuẩn hoá)

> **2 lệnh độc lập:** **`/autotest`** = test tính năng **MỚI** (check spec → **tạo test case cho spec mới** → chạy ngầm → browser → cập nhật tiến độ); **`/retest`** = **chạy lại** test đã có (`--full` hoặc 1 chức năng, KHÔNG tạo test mới). Cả 2 dùng chung cơ chế: **chạy ngầm (headless) trước → browser (headed, giữ mở) sau → cập nhật tiến độ chỉ sau khi browser xong**.

## ⭐ Một bộ test hoàn chỉnh (nguyên tắc gốc)

**Mọi test đều phục vụ MỘT đích: nuôi bộ test hoàn chỉnh.** Chỉ có **1 bộ duy nhất** trong repo — vừa **retest tính năng cũ**, vừa **test feature mới**. Không có suite riêng cho từng lệnh.

- Đăng ký trung tâm: `tests/` (hoặc theo PROJECT_PROFILE) + `test-registry.json` (manifest mọi test + nguồn gốc).
- Test mới **append/cập nhật** vào bộ này, ghi `origin` — không bao giờ tạo suite song song.
- Retest = chạy lại đúng bộ này (theo `--full` hoặc tag/module); feature mới cũng vào cùng bộ.
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

## Lệnh 1 — `/autotest` (test tính năng MỚI, CÓ tạo test case)

> Check spec → **tạo test case cho spec mới** → chạy ngầm (headless) → autotest browser (headed, bung hẳn) → cập nhật tiến độ. **KHÔNG retest luồng cũ.**

### Giai đoạn 0 — Chuẩn bị
- `/spec-link --sync` — pull spec mới nhất; chưa link → hỏi link
- Đọc `.spec-cache/spec/test-scope/current.json` → xác định **phạm vi mới**: `specRefs` + `impact.direct` + `acceptance`
- `spec-validator` PASS trước khi sinh test

### Giai đoạn 1 — TẠO TEST CASE từ spec mới (bắt buộc)
- **Check spec** → lập danh sách `R-xx` **mới / chưa có test** (đối chiếu `.context/coverage.json` + `test-registry.json`)
- `test-writer` sinh test case cho từng `R-xx` mới (unit + property-based), **traceable**; expected từ spec (không từ code)
- Ghi mapping vào `.context/test-plan.md`. **Gate**: mọi `R-xx` mới có ≥1 test case

### Giai đoạn 2 — Chạy NGẦM (headless, KHÔNG bung browser)
- Chạy test **phần mới**: `npm test` / `npx vitest run` / `pytest` / `npx playwright test` (mặc định headless)
- **LƯU kết quả** `.context/test-results/headless-run.json` — **CHỈ LƯU LOG, KHÔNG cập nhật tiến độ**
- Fail → `test-reflector` phân loại (`bug-in-test` / `bug-in-code`)

### Giai đoạn 3 — AUTOTEST với browser (HEADED — bung hẳn ra)
- `npx playwright test --headed` + `use: { headless: false }` → **browser mở thật** cho user theo dõi
- **Đến bước cuối cùng:** (1) **LƯU kết quả** (screenshot + report + json) `.context/test-results/browser-run.json`; (2) **KHÔNG đóng browser** — `await page.pause()` hoặc fixture `keepBrowserOpen` (env `LEAVE_BROWSER_OPEN=1`)
- `test-reflector` phân loại fail

### Giai đoạn 4 — Cập nhật tiến độ (TỰ ĐỘNG — CHỈ sau Giai đoạn 3)
1. Kiểm tra trùng → ghi/cập nhật `test-registry.json`
2. Cập nhật `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`)
3. Cập nhật `.context/test-status.json`

---

## Lệnh 2 — `/retest` (chạy LẠI test đã có, KHÔNG tạo test mới)

> Dùng khi đã có test trong bộ, chỉ muốn **chạy lại** để xác nhận còn xanh.

| Cách dùng | Phạm vi |
|---|---|
| `/retest --full` | toàn bộ test trong `tests/` (theo `test-registry.json`) |
| `/retest <feature\|module>` | 1 chức năng/module (lọc theo `tags`/module/`refs` trong registry) |
| `/retest ... --no-browser` | chỉ chạy ngầm (headless) |

Flow: chọn phạm vi → **chạy ngầm (headless)** → **chạy browser (headed, giữ mở)** → cập nhật `status`/`lastRunAt` + tiến độ. **KHÔNG sinh test mới.**

---

## Bảng tóm tắt

| Lệnh | Mục đích | Tạo test case? | Phạm vi | Cơ chế chạy |
|---|---|---|---|---|
| `/autotest` | test tính năng MỚI | ✅ CÓ | phần mới (spec/test-scope) | ngầm (headless) → browser (headed, giữ mở) |
| `/retest` | chạy lại test đã có | ❌ KHÔNG | `--full` hoặc 1 chức năng | ngầm (headless) → browser (headed, giữ mở) |

Giai đoạn trong mỗi lần chạy:

| Giai đoạn | Cập nhật tiến độ? |
|---|---|
| Chạy NGẦM (headless) | ❌ chỉ lưu log |
| Chạy BROWSER (headed, giữ mở) | ❌ |
| Sau browser xong | ✅ registry + coverage + test-status |

## Vòng lặp thực tế

```
[DEV] code mới / feature mới / spec đổi ──► sinh .spec-cache/spec/test-scope/current.json
                                                        │
                                                        ▼
[TEST] /autotest ──► (0) sync spec + đọc scope
                      (1) TẠO TEST CASE cho spec mới (test-writer)
                      (2) chạy NGẦM headless (chỉ lưu log)
                      (3) autotest BROWSER headed (bung hẳn, giữ browser mở)
                      (4) CẬP NHẬT TIẾN ĐỘ (sau khi browser xong)
                                                        │
[TEST] /retest --full | /retest <feature> ──► chạy LẠI test đã có (không tạo mới)
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

Sau khi **browser test xong**, agent **tự chạy** các việc sau (giữ 1 bộ duy nhất, không tạo suite song song). Người dùng KHÔNG gõ command:

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
