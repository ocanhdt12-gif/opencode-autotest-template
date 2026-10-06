# opencode-autotest-template

Template OpenCode chuyên **sinh + duy trì Auto Test** — viết test từ spec, khóa behavior legacy, chụp test manual, và test theo case user tạo.

> **Match với template dev:** **SPEC là cái chung duy nhất** — nhưng template test **KHÔNG lưu spec**; nó giữ **link git** tới folder spec của repo DEV (`spec-source.json` + `/spec-link <git-url>`) → sync về `.spec-cache/` mỗi lần chạy, tránh 2 bản spec lệch nhau.

> ⭐ **2 lệnh độc lập:**
> - **`/autotest`** — test tính năng **MỚI**: check spec → **tạo test case cho spec mới** → chạy ngầm (headless) → autotest browser (headed, bung hẳn) → cập nhật tiến độ. **Không retest luồng cũ.**
> - **`/retest`** — **chạy lại** test đã có: `--full` (toàn bộ) hoặc `<feature>` (1 chức năng). **Không tạo test mới.**
>
> Cả 2 dùng chung cơ chế chạy: **chạy ngầm (headless) trước** cho nhanh → **browser (headed, bung hẳn ra, giữ mở)** để user theo dõi → **cập nhật tiến độ chỉ sau khi browser xong**; chạy ngầm chỉ **lưu log**.

> ⭐ **Một bộ test hoàn chỉnh (tự động):** mọi test đều nuôi **1 bộ duy nhất** (`tests/`), vừa retest tính năng cũ vừa test feature mới. Chạy xong **tự động** ghi `test-registry.json` + **tự cập nhật tiến độ test** (`.context/coverage.json` + `.context/test-status.json`) — không tạo suite song song. **Spec chỉ đọc từ link git (read-only) — test không tự tạo gì thuộc spec.** Xem `docs/FLOWS.md`.

## 2 lệnh độc lập (chi tiết: `docs/FLOWS.md`)

### `/autotest` — test tính năng MỚI (có tạo test case)

| Giai đoạn | Làm gì | Output |
|---|---|---|
| 0. Chuẩn bị | `/spec-link --sync` + đọc `.spec-cache/spec/test-scope/current.json` → xác định phạm vi mới | phạm vi test |
| 1. **Tạo test case** | Check spec → `test-writer` sinh test cho `R-xx` **mới/chưa có test** | `.context/test-plan.md` |
| 2. **Chạy NGẦM** (headless) | Chạy test phần mới, **không bung browser** — cho nhanh | `.context/test-results/headless-run.json` (**chỉ lưu log**) |
| 3. **Autotest BROWSER** (headed) | **Bung browser thật**, user theo dõi; cuối cùng **lưu kết quả + giữ browser mở** | `.context/test-results/browser-run.json` |
| 4. **Cập nhật tiến độ** (tự động) | Kiểm tra trùng → ghi registry + cập nhật coverage/test-status — **chỉ sau bước 3** | `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` |

### `/retest` — chạy LẠI test đã có (không tạo test mới)

```bash
/retest --full              # chạy lại TOÀN BỘ test đã có
/retest <feature|module>    # chạy lại 1 chức năng/module
/retest ... --no-browser    # chỉ chạy ngầm (headless)
```

Các nhánh **sinh test** bổ trợ khác (test sinh ra sẽ chạy qua `/autotest`): `/characterize` (legacy) · `/capture-manual` (case tay) · `/from-cases` (case user) · `/verify-tests` (gate) · `/coverage` (xem tiến độ).

## Quickstart

```bash
git clone git@github.com:ocanhdt12-gif/opencode-autotest-template.git
cd opencode-autotest-template
npx opencode
```

## Step-by-step

### 0. Link spec (bắt đầu dự án — nhập link git)
Template test **không lưu spec**. Link tới folder spec trong repo DEV:
```bash
/spec-link git@github.com:org/dev-repo.git          # clone shallow folder spec về .spec-cache/
/spec-link --sync                                   # pull spec mới nhất mỗi lần chạy test
/spec-link --status                                 # xem đang link đâu, spec version nào
```
- Đọc requirements từ `.spec-cache/SPECIFICATIONS.md`, scope từ `.spec-cache/spec/test-scope/current.json`
- Chưa link → hỏi, không tự bịa spec

### 1. Test tính năng MỚI — `/autotest`
```bash
/autotest
```
- **(1) Tạo test case** — check spec → sinh test case cho `R-xx` mới/chưa có test (traceable, expected từ spec)
- **(2) Chạy ngầm (headless)** — test chạy như bình thường, **không bung browser** cho nhanh → **chỉ lưu log** `.context/test-results/headless-run.json`
- **(3) Autotest browser (headed)** — **bung browser thật** cho user theo dõi thao tác; đến bước cuối **lưu kết quả** rồi **GIỮ browser mở** để lại màn hình kết quả
- **(4) Cập nhật tiến độ** — chỉ sau khi browser xong: ghi `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` (tự động, không gõ command)

```bash
/autotest <module>          # giới hạn 1 module
/autotest --no-browser      # CHỈ chạy ngầm (không tạo test mới, chỉ chạy phần mới)
```

### 2. Retest lại test đã có — `/retest`
```bash
/retest --full              # chạy lại toàn bộ test đã có
/retest <feature|module>    # chạy lại 1 chức năng
```
Chạy lại test **đã có** trong bộ (không sinh test mới), cập nhật `status`/`lastRunAt`.

### 3. Nhánh sinh test (bổ trợ)

**Characterization (legacy chưa test):**
```bash
/characterize <path>
```
Golden test khóa behavior hiện tại (scrub timestamp/id/random) → mutation verify → an toàn refactor.

**Manual → Auto (case đã test tay):**
```bash
/capture-manual <case-id>
```
Test tay xong → sinh auto test → XANH → đánh dấu regression. Không tự động hoá được → `manual-only` + lý do.

**Theo case user:**
```bash
/from-cases <path>
```
Đọc file test case user (md/csv/sheet) → chuyển từng case thành auto test → báo pass/fail/không tự động hoá được. Bám đúng case user, không tự bịa.

### 4. Verify chất lượng test
```bash
/verify-tests
```
- Mutation testing (mutmut / Stryker) — mutation score ≥ ngưỡng (mặc định 70%)
- Quality gate — chống expected-từ-code, try-catch nuốt lỗi, assert trivially
- Hidden stash — test ẩn chạy CI, chống overfit

### 5. Tiến độ test (biết đã/chưa test đến đâu) — tự động cập nhật
```bash
/coverage            # XEM board: req nào covered/pending/untested/failing
/coverage --gaps     # chỉ phần CHƯA test
```
- Tiến độ **tự cập nhật sau khi browser test xong** (không gõ `/coverage` để cập nhật)
- **Danh sách req lấy từ spec** (`.spec-cache/SPECIFICATIONS.md`, qua link git); **trạng thái do test tự ghi**
- Board **thuộc template TEST**: lưu ở `.context/coverage.json` (repo test)
- **Test KHÔNG tạo gì thuộc spec** — không viết/sinh file spec, không ghi vào `.spec-cache/`

## Cấu trúc thư mục

```
├── AGENT.md              ← pipeline autotest (2 lệnh + nhánh sinh test)
├── AGENTS.md             ← router
├── spec-source.json      ← link git tới folder spec của repo DEV (điền khi bắt đầu)
├── test-registry.json    ← manifest BỘ TEST HOÀN CHỈNH (mọi test + nguồn gốc + trạng thái)
├── BRIEF.md
├── opencode.jsonc
├── .spec-cache/          ← spec clone về (gitignored, read-only)
├── docs/
│   ├── FLOWS.md          ← 2 lệnh + hợp đồng test-scope
│   ├── SPEC_VERSIONING.md ← cách link + version spec/test-scope
│   └── generated/        ← inventory (auto-gen)
├── .opencode/
│   ├── agent/            ← test-writer · characterization-writer · manual-capture-writer · scope-planner · spec-source-linker · test-reflector · test-validator
│   └── command/          ← /autotest · /retest · /characterize · /capture-manual · /from-cases · /verify-tests · /spec-link · /coverage
├── .agent/               ← spec-validator · workflow
├── skills/               ← property-based-testing · mutation-testing · characterization-golden · manual-to-auto · test-quality-gate · coverage-driven · complete-test-suite · test-scope-contract
└── scripts/              ← generate-inventory
```

## Nguyên tắc (bất biến)

1. **Spec-first** — test bám spec từ `.spec-cache/` (nguồn duy nhất, không lưu bản riêng → không lệch)
2. **2 lệnh độc lập** — `/autotest` (tạo test case mới từ spec + chạy phần mới) và `/retest` (chạy lại test đã có: full hoặc 1 chức năng)
3. **Cơ chế chạy chung** — ngầm (headless) trước → browser (headed, giữ mở) sau; tiến độ cập nhật chỉ sau khi browser xong
4. **Test PHẢI có khả năng bất đồng với code** — mutation score là thước đo, không phải coverage %
5. **Characterization = khóa behavior hiện tại, không phải "đúng"**
6. **Manual case → auto test ngay khi có thể** — chỉ giữ manual-only khi thật cần
7. **Test-scope.json có version** — đọc để test đúng phạm vi + biết đã cover đến đâu; thiếu thì hỏi, không tự đoán rộng
8. **Test sinh bởi AI phải qua gate** — không merge test chưa qua `/verify-tests`
9. **Mọi test mới vào bộ hoàn chỉnh** — tra registry trước, append/cập nhật, không dựng suite song song
