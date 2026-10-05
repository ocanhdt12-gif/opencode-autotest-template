# FLOWS — 5 luồng test (chuẩn hoá)

> **Ngày:** 05/10/2026 · Định nghĩa 5 luồng test + hợp đồng bàn giao `test-scope` giữa template DEV (viết code) và template AUTOTEST (viết/ chạy test).

## ⭐ Một bộ test hoàn chỉnh (nguyên tắc gốc)

**Mọi luồng test đều phục vụ MỘT đích: nuôi bộ test hoàn chỉnh.** Test chỉ có **1 bộ duy nhất** trong repo — vừa để **retest tính năng cũ**, vừa để **test feature mới**. Không có "bộ test của luồng 1", "bộ test của luồng 2" riêng rẽ.

- Đăng ký trung tâm: `tests/` (hoặc theo PROJECT_PROFILE) + `test-registry.json` (manifest mọi test + nguồn gốc).
- Mỗi luồng chạy xong → **bổ sung/cập nhật** vào bộ này, ghi `origin` (luồng nào sinh ra) — không bao giờ tạo suite song song.
- Retest (luồng 3) = chạy lại đúng bộ này theo `impact.regression`; feature mới cũng vào cùng bộ.
- `test-registry.json` + `.context/coverage.json` là 2 mặt của cùng bộ: cái nào tồn tại/đã chạy (board) · test nào đang giữ phần đó (`testRef`).

| Luồng | Sinh ra gì | Đổ vào bộ hoàn chỉnh? |
|---|---|---|
| 1 Full | test mọi `R-xx` | ✅ thêm mới |
| 2 Test-scope | test cho `direct`/`acceptance` vừa đổi | ✅ thêm mới / cập nhật test cũ |
| 3 Regression | retest luồng cũ | ♻️ chỉ chạy lại, cập nhật `lastRunAt` |
| 4 Manual→Auto | case tay → auto | ✅ thêm mới + đăng ký regression |
| 5 From-cases | test theo case user | ✅ thêm mới + đăng ký regression |
| B Characterization | golden test khóa legacy | ✅ thêm mới (chống vỡ khi refactor) |

> **Bất biến:** sau mỗi luồng, số test trong bộ **chỉ tăng hoặc cập nhật**, không bao giờ bị luồng khác ghi đè/xoá. Xem mục "Đăng ký bộ test hoàn chỉnh" cuối file.

## Hợp đồng bàn giao: `.spec-cache/spec/test-scope/current.json` (versioned)

**Template DEV sinh ra** sau mỗi bug fix / feature update (xem template dev FEATURE_WORKFLOW). **Template AUTOTEST đọc** để biết cần test cái gì — không phải tự mò.

> 📌 Vị trí + version scheme: xem `docs/SPEC_VERSIONING.md`. Scope nằm trong `spec/test-scope/` (cạnh spec), **có `specVersion` + `scopeVersion`** để test biết đang cover đến đâu.

```jsonc
{
  "specVersion": "1.2.0",       // scope này bám spec version nào
  "scopeVersion": 3,             // lần sinh thứ mấy (tăng mỗi lần)
  "generatedAt": "2026-10-05T13:32:00+07:00",
  "trigger": "initial-build | bug-fix | feature-update",
  "workItem": "bug-login-timeout | feature-stripe-selfserve",
  "specRefs": ["R-01", "R-05"],              // requirement liên quan (bám .spec-cache/SPECIFICATIONS.md)
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

**Producer:** template DEV (`builder`, sau mỗi sửa) — bắt buộc, không bỏ.
**Consumer:** template AUTOTEST (`scope-planner` + commands) — đọc để chọn test, ghi `.context/test-status.json` để theo dõi version đã cover.

---

## 5 luồng test

### Luồng 1 — Full test lần đầu (sau khi code xong từ template dev)
- **Khi nào:** code hoàn thành lần đầu (initial build), chưa có test suite
- **Command:** `/autotest --full`
- **Làm gì:** đọc `.spec-cache/SPECIFICATIONS.md` → sinh test cho **toàn bộ requirement R-xx** → chạy ĐỎ → (code) → XANH → `/verify-tests`
- **Đầu ra:** test suite đầy đủ phủ spec

### Luồng 2 — Test phần vừa sửa (bug fix / feature mới) 🔑 anh nhấn mạnh
- **Khi nào:** vừa fix bug / thêm tính năng → template dev đã sinh `.spec-cache/spec/test-scope/current.json`
- **Command:** `/test-scope` (mặc định đọc `.spec-cache/spec/test-scope/current.json`)
- **Làm gì:** đọc scope → chỉ test **changed.direct + changed.dependents + acceptance** (không test lại cả repo)
- **Đầu ra:** test cho đúng phạm vi vừa đổi — nhanh, tập trung
- **Ghi chú:** đây chính là "template code gen scope cho template test dùng luôn" — anh không phải tự check

### Luồng 3 — Regression (retest luồng cũ sau update)
- **Khi nào:** sau update, cần chắc **luồng cũ không bị ảnh hưởng**
- **Command:** `/regression` (đọc `impact.regression` + dependents trong test-scope)
- **Làm gì:** chạy lại các test **đã có** thuộc luồng cũ có nguy cơ ảnh hưởng + test lân cận (dependents)
- **Đầu ra:** xác nhận cũ không vỡ; nếu vỡ → test-reflector phân loại (bug-code → báo dev)

### Luồng 4 — Manual → Auto (case test tay đã xong)
- **Khi nào:** case đã test tay PASS
- **Command:** `/capture-manual <case-id>`
- **Làm gì:** mô tả case → sinh auto test → XANH → **add vào luồng regression** (đăng ký vào bộ chạy tự động)
- **Đầu ra:** case được tự động hoá, lần sau chạy chung suite

### Luồng 5 — Test theo test case do USER tạo
- **Khi nào:** user/khách đưa bộ test case (file Excel/MD/sheet, checklist nghiệm thu)
- **Command:** `/from-cases <path>`
- **Làm gì:** đọc test case user → chuyển từng case thành auto test (map input/expected/bước) → chạy → báo case nào pass/fail/không tự động hoá được
- **Đầu ra:** bộ auto test bám đúng test case user (không tự bịa thêm yêu cầu)

---

## Bảng tóm tắt

| # | Luồng | Command | Đọc gì | Test gì |
|---|---|---|---|---|
| 1 | Full lần đầu | `/autotest --full` | .spec-cache/SPECIFICATIONS.md | toàn bộ R-xx |
| 2 | Phần vừa sửa | `/test-scope` | .spec-cache/spec/test-scope/current.json | direct + dependents + acceptance |
| 3 | Regression | `/regression` | .spec-cache/spec/test-scope/current.json | regression list + dependents (test cũ) |
| 4 | Manual→Auto | `/capture-manual` | manual-cases/ | case tay → auto (rồi add regression) |
| 5 | Theo case user | `/from-cases` | file test case user | đúng case user đưa |

## Vòng lặp thực tế

```
[DEV] code lần đầu ──► (1) /autotest --full            ← test toàn bộ spec
[DEV] fix bug/feature ─► sinh .spec-cache/spec/test-scope/current.json ──► (2) /test-scope   ← test phần sửa
                                          └──────► (3) /regression   ← retest luồng cũ
[TEST TAY] case pass ──► (4) /capture-manual ──► add vào regression suite
[USER] đưa test case ──► (5) /from-cases ──► auto test bám case user

         └──────── tất cả ────────► 📦 BỘ TEST HOÀN CHỈNH (tests/ + test-registry.json)
                                          └──► dùng để retest luồng cũ + test feature mới
```

---

## Đăng ký bộ test hoàn chỉnh (mọi luồng đều gọi)

Cuối MỌI luồng sinh/chạy test, thực hiện 3 việc sau (giữ 1 bộ duy nhất, không tạo suite song song):

1. **Ghi manifest** `test-registry.json` (root repo TEST) — mỗi test 1 entry:
   ```jsonc
   {
     "updatedAt": "2026-10-05T14:30:00+07:00",
     "tests": [
       { "id": "tests/auth.test.ts::login_r01", "file": "tests/auth.test.ts",
         "refs": ["R-01"], "requirement": "R-01",
         "origin": "full | test-scope | regression | manual | from-cases | characterization",
         "tags": ["auth", "smoke"], "regression": true,
         "status": "pass", "lastRunAt": "2026-10-05T14:35:00+07:00" }
     ]
   }
   ```
2. **Không tạo file/suite test mới tách rời** — test mới append vào file hiện có theo module, hoặc thêm file nhưng cùng `tests/` và cùng được liệt kê trong registry.
3. **Cập nhật board** `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`) — retest sau này bám board để biết chạy cái gì.

**Rule chống trùng:** trước khi thêm test, tra `test-registry.json` — đã có test phủ cùng `requirement` + cùng behavior → **cập nhật** thay vì thêm bản sao.

**Rule additive:** luồng nào cũng chạy bước 1-3. Luồng 3 (regression) chỉ cập nhật `status`/`lastRunAt`, KHÔNG sinh test mới (trừ khi `test-reflector` phát hiện thiếu).