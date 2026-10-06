# AGENT.md — Autotest Generation Pipeline

> Template xử lý bài toán: **sinh + duy trì auto test**. Match với template dev: **SPEC là cái chung** — nhưng test **không lưu spec**, chỉ **link git** (`spec-source.json` → `.spec-cache/`).

> ⭐ **Một luồng chạy duy nhất — `/autotest`**: chạy **ngầm (headless) 1 lượt trước** cho nhanh → rồi chạy **autotest với browser (headed, bung hẳn ra)** để user theo dõi thao tác → cuối cùng **lưu kết quả + GIỮ browser mở** cho user xem màn hình kết quả. Chỉ **sau khi browser xong** mới **cập nhật tiến độ test**; chạy ngầm chỉ **lưu log**.

> ⭐ **Một bộ test hoàn chỉnh:** mọi test đều nuôi **1 bộ duy nhất** (`tests/` + `test-registry.json`) — vừa retest tính năng cũ, vừa test feature mới. Không tạo suite song song.

## Luồng chạy DUY NHẤT — `/autotest` (xem chi tiết `docs/FLOWS.md`)

| # | Giai đoạn | Làm gì | Output |
|---|---|---|---|
| 0 | Chuẩn bị | `/spec-link --sync` + đọc `.spec-cache/spec/test-scope/current.json` → xác định phạm vi ưu tiên (direct/dependents/regression/acceptance) | phạm vi test |
| 1 | **Chạy NGẦM** (headless) | Chạy test suite như bình thường, **KHÔNG bung browser** — cho nhanh | `.context/test-results/headless-run.json` (**chỉ lưu log**) |
| 2 | **Autotest BROWSER** (headed) | **Bung browser thật**, user theo dõi thao tác; cuối cùng **lưu kết quả + GIỮ browser mở** | `.context/test-results/browser-run.json` |
| 3 | **Cập nhật tiến độ** (TỰ ĐỘNG) | Kiểm tra trùng → ghi registry + cập nhật coverage/test-status — **CHỈ sau Giai đoạn 2** | `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` |

## Nhánh sinh test (bổ trợ — test sinh ra sẽ chạy qua luồng `/autotest`)

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
/autotest  ── luồng DUY NHẤT
      │
      ├── (0) sync spec + đọc test-scope (phạm vi ưu tiên)
      │
      ├── (1) CHẠY NGẦM (headless, KHÔNG browser)
      │        └── lưu .context/test-results/headless-run.json  (chỉ log, KHÔNG cập nhật tiến độ)
      │
      ├── (2) AUTOTEST BROWSER (headed, BUNG HẲN ra)
      │        └── user theo dõi thao tác
      │        └── cuối cùng: lưu kết quả + GIỮ browser mở  → browser-run.json
      │
      └── (3) CẬP NHẬT TIẾN ĐỘ (TỰ ĐỘNG, chỉ sau bước 2)
               └── test-registry.json + .context/coverage.json + .context/test-status.json
                    (⬆ luôn có 1 bộ test hoàn chỉnh để retest; không cần gõ command)
```

## Gate bắt buộc

1. `spec-validator` PASS trước khi sinh test
2. Test-first: test **ĐỎ đúng cách** trước khi code
3. Characterization: **mutation check bắt được**
4. Manual→Auto / từ-cases: test xanh, assert thật, không ép tự động hóa case cần người
5. `verify-tests`: mutation score ≥ ngưỡng + quality gate + hidden stash
6. Mọi fail → test-reflector phân loại → sửa đúng chỗ
7. **Browser LUÔN headed** khi chạy autotest; cuối luồng **GIỮ browser mở** (không close)
8. **Tiến độ chỉ cập nhật sau giai đoạn browser** — chạy ngầm chỉ lưu log
9. **Spec chỉ ĐỌC từ link** (`.spec-cache/`, read-only) — test KHÔNG tạo/sinh/sửa spec; test chỉ giữ **tiến độ test** của chính nó

## Conventions

- Test file trong `tests/` hoặc cạnh code; test gắn `R-xx` (spec) hoặc case id (user/manual)
- Test name phản ánh hành vi, không implementation
- Không try-catch nuốt lỗi; không assert trivially
- Scrub unstable fields trong golden test
- `manual-only` case ghi rõ lý do
- Nhánh manual/from-cases: test tự động hoá xong **đánh dấu regression**
- ⭐ **Một bộ duy nhất**: mọi test ghi vào `tests/` hiện có + `test-registry.json`; tra registry trước để **cập nhật** thay vì thêm bản sao
- Mỗi slice 1 commit
