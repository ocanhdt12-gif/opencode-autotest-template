# AGENT.md — Autotest Generation Pipeline

> Template xử lý bài toán: **sinh + duy trì auto test**. Match với template dev: **SPEC là cái chung** — nhưng test **không lưu spec**, chỉ **link git** (`spec-source.json` → `.spec-cache/`).

> ⭐ **Luật trục — TEST-CASE-FIRST:** **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.** Mọi test phải follow test case; test code chỉ hiện thực hoá đúng test case user đã chốt. Xem `skills/test-case-first`.

> ⭐ **2 lệnh độc lập:**
> - **`/autotest`** — test tính năng MỚI/spec mới: (lần đầu) hỏi config spec → đọc spec + overview → brainstorm câu hỏi → **tạo test case** → **chờ user sửa & chốt** → chạy ngầm (headless, log bug) → **chạy browser TỪNG CASE** cho user kiểm tra → cập nhật trạng thái. Các lần sau: đọc spec → lấy task **chưa test** → tạo test case → chạy y hệt.
> - **`/retest`** — chạy LẠI test đã có: `--all` (toàn hệ thống) · `<module>` (1 cụm chức năng) · `<TC-xx>` (1 test case). KHÔNG tạo test mới.
>
> Cả 2 dùng chung cơ chế: **ngầm (headless) trước** → **browser (headed, bung hẳn, giữ mở)** sau → **cập nhật trạng thái**.

> ⭐ **Một bộ test hoàn chỉnh:** mọi test đều nuôi **1 bộ duy nhất** (`tests/` + `test-registry.json`) — vừa retest tính năng cũ, vừa test feature mới. Không tạo suite song song.

## `/autotest` — test tính năng MỚI (test-case-first)

| # | Giai đoạn | Làm gì | Output |
|---|---|---|---|
| 0 | (Lần đầu) Config + spec | Hỏi `/spec-link` nếu chưa link → đọc spec + nêu **overview** → **brainstorm câu hỏi** cần hỏi | — |
| A | **Tạo TEST CASE** (draft) | `test-case-author` soạn test case `TC-xx` từ `R-xx` mới, **gom theo module** | `.context/test-cases/<module>.md` + `.context/test-tasks.json` |
| B | ⛔ **User check & update** | **DỪNG** — user sửa/chốt test case (`approved`) | test case `approved` |
| C | Sinh test code + chạy **NGẦM** (headless) | `test-writer` hiện thực hoá đúng test case → chạy headless → **log bug ra file** | `.context/test-results/headless-run.json` (+ `bugs.md`) |
| D | **Chạy BROWSER từng case** (headed) | Tạo list task theo module → **đi từng case 1**, bung browser thật, giữ mở; **xong 1 case chờ user chọn case tiếp** | `.context/test-results/browser-run.json` |
| E | Cập nhật trạng thái | Cập nhật `Test status` + `test-tasks.json` + `test-registry.json` + coverage → **không test lại cái đã test** | `test-registry.json` + `.context/test-tasks.json` + `.context/coverage.json` |

**Các lần sau:** vào `/autotest` → đọc spec → lấy task **chưa test** (`test-tasks.json`) → tạo test case → chạy lại luồng A–E.

## `/retest` — chạy LẠI test đã có (không tạo mới)

| Cách dùng | Phạm vi |
|---|---|
| `/retest --all` | toàn bộ hệ thống |
| `/retest <module\|feature>` | 1 cụm chức năng |
| `/retest <TC-xx>` | 1 test case |

Cùng cơ chế: ngầm (headless) → browser (headed, giữ mở) → cập nhật trạng thái. **Không sinh test mới.**

## Nhánh sinh test bổ trợ (đều đổ về test case → chạy qua `/autotest`)

| Nhánh | Input | Output | Agent |
|---|---|---|---|
| B. Characterization | Legacy chưa test | golden test khóa behavior | `characterization-writer` → `test-reflector` |
| C. Manual→Auto | Case đã test tay | auto test + add regression | `manual-capture-writer` → `test-reflector` |
| — Theo case user | File test case user | auto test bám đúng case user | `test-writer` |

## Hợp đồng bàn giao

Template DEV sinh `.spec-cache/spec/test-scope/current.json` (có `specVersion`+`scopeVersion`) sau mỗi sửa (xem `skills/test-scope-contract` + `docs/SPEC_VERSIONING.md`) → template AUTOTEST đọc để biết cần test gì, ghi `.context/test-status.json` để theo dõi version đã cover. Spec lấy qua link git (`/spec-link`) — **không lưu bản riêng**.

## Pipeline

```
.spec-cache/SPECIFICATIONS.md (chung với template dev)
      │  spec-validator PASS
      ▼
/autotest  ── tính năng MỚI (test-case-first)
      │
      ├── (0) lần đầu: config spec + overview + brainstorm câu hỏi
      ├── (A) TẠO TEST CASE (gom theo module)        → .context/test-cases/<module>.md (draft)
      ├── (B) ⛔ user check & update → approved      ← human checkpoint
      ├── (C) sinh test code theo test case → chạy NGẦM (headless) → log bug
      ├── (D) chạy BROWSER TỪNG CASE (headed, giữ mở) → user chọn case tiếp
      └── (E) CẬP NHẬT TRẠNG THÁI (không test lại cái đã test)

/retest  ── chạy LẠI (--all | <module> | <TC-xx>), KHÔNG tạo test mới
      └── ngầm (headless) → browser (headed, giữ mở) → cập nhật trạng thái

         └────────► 📦 BỘ TEST HOÀN CHỈNH (tests/ + test-registry.json)
```

## Gate bắt buộc

1. `spec-validator` PASS trước khi sinh test case
2. **Test case `approved` bởi user trước khi sinh test code/chạy** (checkpoint B)
3. **Mọi test code gắn `TC-xx`** — không test ngoài test case
4. Characterization: **mutation check bắt được**
5. Manual→Auto / từ-cases: test xanh, assert thật, không ép tự động hóa case cần người
6. `verify-tests`: mutation score ≥ ngưỡng + quality gate + hidden stash
7. Mọi fail → test-reflector phân loại → sửa đúng chỗ
8. **Browser LUÔN headed**; chạy **từng case**, cuối case **GIỮ browser mở**
9. **Cập nhật trạng thái sau khi chạy** → không test lại cái đã test
10. **Spec chỉ ĐỌC từ link** (`.spec-cache/`, read-only)

## Conventions

- Test case: `.context/test-cases/<module>.md` (gom theo module), id `TC-<module>-NN`
- Trạng thái task: `.context/test-tasks.json` (case nào đã/chưa test)
- Test file trong `tests/`; test gắn `TC-xx` + `R-xx`
- Test name phản ánh hành vi, không implementation
- Không try-catch nuốt lỗi; không assert trivially
- Scrub unstable fields trong golden test
- `manual-only` case ghi rõ lý do
- ⭐ **Một bộ duy nhất**: mọi test ghi vào `tests/` hiện có + `test-registry.json`; tra registry trước để **cập nhật** thay vì thêm bản sao
- Mỗi slice 1 commit
