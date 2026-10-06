# FLOWS — Luồng chạy test (chuẩn hoá)

> **Luật trục — TEST-CASE-FIRST:** **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.** Mọi test phải follow test case (`skills/test-case-first`).
> **2 lệnh:** `/autotest` = test tính năng **MỚI** (có tạo test case); `/retest` = **chạy lại** test đã có (`--all` / cụm chức năng / 1 test case).

## ⭐ Một bộ test hoàn chỉnh (nguyên tắc gốc)

Mọi test đều nuôi **1 bộ duy nhất** trong repo — vừa **retest tính năng cũ**, vừa **test feature mới**. Không có suite riêng.

- Đăng ký trung tâm: `tests/` + `test-registry.json` (manifest mọi test + nguồn gốc + `testCase`).
- Test mới **append/cập nhật** vào bộ này — không bao giờ tạo suite song song.
- `test-registry.json` + `.context/coverage.json` + `.context/test-tasks.json` là các mặt của cùng bộ: test nào tồn tại/đã chạy · test nào giữ phần nào.

## Test case — nơi user chốt "cần test cái gì"

- Test case gom theo **module**, viết **dạng BẢNG** (mỗi dòng 1 case): `.context/test-cases/<module>.md`, id `TC-<module>-NN`.
- Mỗi cột bảng: ID · Requirement (`R-xx`) · Tầng · Input · Steps · Expected (từ spec) · `Status` (draft→approved) · `Test status` (untested→passed/failed) · `Test ref`.
- 📌 **Task mới cần test**: cùng **module đã có → UPDATE FILE CŨ** (thêm dòng vào bảng, số TC tiếp theo); module **mới → TẠO FILE MỚI** `<module>.md`. 1 module = 1 file, không tạo file trùng.
- Trạng thái task: `.context/test-tasks.json` (module → cases; biết case nào **chưa test**).
- Test code chỉ được sinh **sau khi** test case `approved` — và phải gắn `TC-xx`.

## Hợp đồng bàn giao: `.spec-cache/spec/test-scope/current.json` (versioned)

**Template DEV sinh ra** sau mỗi bug fix / feature update. **Template AUTOTEST đọc** để biết cần test cái gì.

```jsonc
{
  "specVersion": "1.2.0",
  "scopeVersion": 3,
  "generatedAt": "2026-10-05T13:32:00+07:00",
  "trigger": "initial-build | bug-fix | feature-update",
  "workItem": "bug-login-timeout",
  "specRefs": ["R-01", "R-05"],
  "changed": { "files": ["src/auth/login.ts"], "modules": ["auth"] },
  "impact": { "direct": ["auth.login"], "dependents": ["portal.session"], "regression": ["payment.checkout"] },
  "risk": "low | medium | high",
  "acceptance": ["login thành công < 2s"],
  "notes": "đổi timeout 30s → 10s"
}
```

**Producer:** template DEV. **Consumer:** template AUTOTEST giai đoạn 0 của `/autotest`.

---

## Lệnh 1 — `/autotest` (test tính năng MỚI)

### Lần đầu
1. **Config spec** — chưa link → hỏi `/spec-link <git-url>`
2. **Đọc spec + overview** — spec version, modules, `R-xx`, phần chưa test
3. **Brainstorm câu hỏi** — spec mơ hồ (expected, edge, ưu tiên) → hỏi user
4. **Tạo test case** (draft, gom theo module)
5. ⛔ **User check & update** → chốt `approved`
6. **Sinh test code theo test case → chạy NGẦM (headless)** → log bug ra file
7. **Chạy BROWSER TỪNG CASE** (headed, giữ mở) → user chọn case tiếp
8. **Cập nhật trạng thái task**

### Các lần sau
Vào `/autotest` → đọc spec → lấy task **chưa test** (`test-tasks.json`) → tạo test case → chạy luồng bước 4–8.

### Chi tiết giai đoạn

| Bước | Làm gì | Output |
|---|---|---|
| A. Tạo test case | `test-case-author` soạn `TC-xx` từ `R-xx` mới, **dạng bảng**, gom theo module, `Status: draft`; cùng module → update file cũ (thêm dòng), module mới → tạo file mới | `.context/test-cases/<module>.md` + `.context/test-tasks.json` |
| B. ⛔ Human checkpoint | **DỪNG**, user sửa/chốt test case (`approved`) | test case `approved` |
| C. Test code + chạy ngầm | `test-writer` hiện thực hoá đúng test case → chạy headless → log bug | `.context/test-results/headless-run.json` (+ `bugs.md`) |
| D. Chạy browser từng case | list task theo module → đi **từng case 1**, browser headed bung hẳn, giữ mở; xong 1 case **chờ user chọn case tiếp** | `.context/test-results/browser-run.json` |
| E. Cập nhật trạng thái | `Test status` + `test-tasks.json` + `test-registry.json` + coverage → không test lại cái đã test | cập nhật state |

**Biến thể:** `/autotest <module>` · `/autotest --no-browser` (chỉ chạy ngầm).

---

## Lệnh 2 — `/retest` (chạy LẠI test đã có, KHÔNG tạo mới)

| Cách dùng | Phạm vi |
|---|---|
| `/retest --all` | toàn bộ hệ thống |
| `/retest <module\|feature>` | 1 cụm chức năng |
| `/retest <TC-xx>` | 1 test case |
| `/retest ... --no-browser` | chỉ chạy ngầm (headless) |

Flow: chọn phạm vi → chạy ngầm (headless) → chạy browser (headed, giữ mở; đi từng case như `/autotest`) → cập nhật trạng thái. **KHÔNG sinh test case/test mới.**

---

## Bảng tóm tắt

| Lệnh | Mục đích | Tạo test case? | Phạm vi |
|---|---|---|---|
| `/autotest` | test tính năng MỚI | ✅ CÓ | phần mới (spec/test-scope) |
| `/retest` | chạy lại test đã có | ❌ KHÔNG | `--all` / 1 cụm chức năng / 1 test case |

| Giai đoạn | Cập nhật trạng thái? |
|---|---|
| Chạy NGẦM (headless) | ❌ chỉ lưu log + bug |
| Chạy BROWSER (headed, từng case, giữ mở) | ✅ sau mỗi case |

## Vòng lặp thực tế

```
[DEV] code mới / feature mới / spec đổi ──► sinh .spec-cache/spec/test-scope/current.json
                                                        │
[TEST] /autotest ──► (0) config spec + overview + brainstorm
                      (A) TẠO TEST CASE (gom theo module, draft)
                      (B) ⛔ user check & update → approved
                      (C) test code theo test case → chạy NGẦM headless → log bug
                      (D) chạy BROWSER từng case (headed, giữ mở) → user chọn case tiếp
                      (E) cập nhật trạng thái (không test lại cái đã test)
                                                        │
[TEST] /retest --all | <module> | <TC-xx> ──► chạy LẠI test đã có
                                                        │
[TEST TAY] case pass ──► /capture-manual ──► auto test ─┤
[USER] đưa test case ──► /from-cases ──────► auto test ─┤
[LEGACY] ─────────────► /characterize ─────► golden test ┘
                                                        ▼
                            📦 BỘ TEST HOÀN CHỈNH (tests/ + test-registry.json)
```

---

## Đăng ký bộ test + cập nhật trạng thái (TỰ ĐỘNG, sau khi chạy)

Sau khi case chạy xong (headless + browser), agent **tự chạy**:

1. **Ghi manifest** `test-registry.json` — mỗi test 1 entry (có `testCase: "TC-xx"`):
   ```jsonc
   { "id": "tests/auth.spec.ts::login", "file": "tests/auth.spec.ts",
     "testCase": "TC-auth-01", "refs": ["R-01"], "origin": "autotest",
     "tags": ["auth"], "status": "pass", "lastRunAt": "..." }
   ```
2. **Cập nhật trạng thái task**: `Test status` trong `.context/test-cases/<module>.md` + `headless`/`browser`/`lastRunAt` trong `.context/test-tasks.json`.
3. **Cập nhật tiến độ**: `.context/coverage.json` (req → covered/failing) + `.context/test-status.json` (specVersionCovered). Danh sách req lấy từ spec (đọc, không ghi).

**Rule chống trùng:** tra `test-registry.json` trước — cùng `testCase`/`requirement` + behavior → **cập nhật** thay vì thêm bản sao.

**Rule trạng thái:** `/coverage` chỉ để **xem** — trạng thái do bước tự động sau khi chạy cập nhật. **Spec chỉ đọc từ link** (read-only).
