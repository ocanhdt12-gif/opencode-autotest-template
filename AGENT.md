# AGENT.md — Autotest Generation Pipeline

> Template xử lý bài toán: **sinh + duy trì auto test**. Match với template dev: **SPEC là cái chung** — nhưng test **không lưu spec**, chỉ **link git** (`spec-source.json` → `.spec-cache/`).

> ⭐ **2 lệnh độc lập:**
> - **`/autotest`** — test tính năng **MỚI**: check spec → **tạo test case cho spec mới** → chạy ngầm (headless) → autotest browser (headed, bung hẳn) → cập nhật tiến độ. **Không retest luồng cũ.**
> - **`/retest`** — **chạy lại** test đã có: `--full` (toàn bộ) hoặc `<feature>` (1 chức năng). **Không tạo test mới.**
>
> Cả 2 dùng chung cơ chế chạy: **ngầm (headless) trước** → **browser (headed, bung hẳn, giữ mở)** sau → **cập nhật tiến độ chỉ sau khi browser xong**; chạy ngầm chỉ **lưu log**.

> ⭐ **Một bộ test hoàn chỉnh:** mọi test đều nuôi **1 bộ duy nhất** (`tests/` + `test-registry.json`) — vừa retest tính năng cũ, vừa test feature mới. Không tạo suite song song.

## 2 lệnh độc lập (xem chi tiết `docs/FLOWS.md`)

### `/autotest` — test tính năng MỚI (có tạo test case)

| # | Giai đoạn | Làm gì | Output |
|---|---|---|---|
| 0 | Chuẩn bị | `/spec-link --sync` + đọc test-scope → xác định phạm vi mới | phạm vi test |
| 1 | **Tạo test case** | Check spec → `test-writer` sinh test cho `R-xx` **mới/chưa có test** | `.context/test-plan.md` |
| 2 | **Chạy NGẦM** (headless) | Chạy test phần mới, **KHÔNG bung browser** — cho nhanh | `.context/test-results/headless-run.json` (**chỉ lưu log**) |
| 3 | **Autotest BROWSER** (headed) | **Bung browser thật**, user theo dõi; cuối cùng **lưu kết quả + GIỮ browser mở** | `.context/test-results/browser-run.json` |
| 4 | **Cập nhật tiến độ** (TỰ ĐỘNG) | Kiểm tra trùng → ghi registry + coverage/test-status — **CHỈ sau Giai đoạn 3** | `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` |

### `/retest` — chạy LẠI test đã có (không tạo test mới)

| Cách dùng | Phạm vi |
|---|---|
| `/retest --full` | toàn bộ test trong `tests/` (theo `test-registry.json`) |
| `/retest <feature\|module>` | 1 chức năng/module (lọc theo tag/module) |

Cùng cơ chế chạy: ngầm (headless) → browser (headed, giữ mở) → cập nhật `status`/`lastRunAt`. **Không sinh test mới** — muốn tạo test mới dùng `/autotest`.

## Nhánh sinh test (bổ trợ — test sinh ra sẽ chạy qua `/autotest`)

| Nhánh | Input | Output | Agent |
|---|---|---|---|
| A. Test-first | `.spec-cache/SPECIFICATIONS.md` + task | test trước code (red → green) | `test-writer` → `test-reflector` |
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
/autotest  ── tính năng MỚI (có tạo test case)
      │
      ├── (0) sync spec + đọc test-scope
      ├── (1) TẠO TEST CASE từ spec mới (test-writer)  → test-plan.md
      ├── (2) CHẠY NGẦM (headless, KHÔNG browser)      → headless-run.json  (chỉ log)
      ├── (3) AUTOTEST BROWSER (headed, giữ mở)        → browser-run.json
      └── (4) CẬP NHẬT TIẾN ĐỘ (tự động, chỉ sau bước 3)

/retest  ── chạy LẠI test đã có (KHÔNG tạo test mới)
      ├── --full | <feature>
      ├── chạy ngầm (headless) → browser (headed, giữ mở)
      └── cập nhật status/lastRunAt

         └────────► 📦 BỘ TEST HOÀN CHỈNH (tests/ + test-registry.json)
```

## Gate bắt buộc

1. `spec-validator` PASS trước khi sinh test
2. `/autotest`: **tạo test case cho mọi `R-xx` mới trước khi chạy**
3. Test-first: test **ĐỎ đúng cách** trước khi code
4. Characterization: **mutation check bắt được**
5. Manual→Auto / từ-cases: test xanh, assert thật, không ép tự động hóa case cần người
6. `verify-tests`: mutation score ≥ ngưỡng + quality gate + hidden stash
7. Mọi fail → test-reflector phân loại → sửa đúng chỗ
8. **Browser LUÔN headed** khi chạy test; cuối luồng **GIỮ browser mở** (không close)
9. **Tiến độ chỉ cập nhật sau giai đoạn browser** — chạy ngầm chỉ lưu log
10. **Spec chỉ ĐỌC từ link** (`.spec-cache/`, read-only) — test KHÔNG tạo/sinh/sửa spec

## Conventions

- Test file trong `tests/` hoặc cạnh code; test gắn `R-xx` (spec) hoặc case id (user/manual)
- Test name phản ánh hành vi, không implementation
- Không try-catch nuốt lỗi; không assert trivially
- Scrub unstable fields trong golden test
- `manual-only` case ghi rõ lý do
- Nhánh manual/from-cases: test tự động hoá xong **đánh dấu regression**
- ⭐ **Một bộ duy nhất**: mọi test ghi vào `tests/` hiện có + `test-registry.json`; tra registry trước để **cập nhật** thay vì thêm bản sao
- Mỗi slice 1 commit
