# opencode-autotest-template

Template OpenCode chuyên **sinh + duy trì Auto Test** — viết test từ spec, khóa behavior legacy, chụp test manual, và test theo case user tạo.

> **Match với template dev:** **SPEC là cái chung duy nhất** — nhưng template test **KHÔNG lưu spec**; nó giữ **link git** tới folder spec của repo DEV (`spec-source.json` + `/spec-link <git-url>`) → sync về `.spec-cache/` mỗi lần chạy, tránh 2 bản spec lệch nhau.

> ⭐ **Một luồng chạy duy nhất — `/autotest`:** chạy **ngầm (headless) 1 lượt trước** cho nhanh → rồi chạy **autotest với browser (headed, bung hẳn ra)** cho user theo dõi thao tác → cuối cùng **lưu kết quả + GIỮ browser mở** để lại màn hình kết quả cho user. Chỉ **sau khi browser xong** mới **cập nhật tiến độ test**; chạy ngầm chỉ **lưu log**.

> ⭐ **Một bộ test hoàn chỉnh (tự động):** mọi test đều nuôi **1 bộ duy nhất** (`tests/`), vừa retest tính năng cũ vừa test feature mới. Chạy xong **tự động** ghi `test-registry.json` + **tự cập nhật tiến độ test** (`.context/coverage.json` + `.context/test-status.json`) — không tạo suite song song. **Spec chỉ đọc từ link git (read-only) — test không tự tạo gì thuộc spec.** Xem `docs/FLOWS.md`.

## Luồng chạy duy nhất `/autotest` (chi tiết: `docs/FLOWS.md`)

| Giai đoạn | Làm gì | Output |
|---|---|---|
| 0. Chuẩn bị | `/spec-link --sync` + đọc `.spec-cache/spec/test-scope/current.json` → xác định phạm vi ưu tiên | phạm vi test |
| 1. **Chạy NGẦM** (headless) | Chạy test suite như bình thường, **không bung browser** — cho nhanh | `.context/test-results/headless-run.json` (**chỉ lưu log**) |
| 2. **Autotest BROWSER** (headed) | **Bung browser thật**, user theo dõi thao tác; cuối cùng **lưu kết quả + giữ browser mở** | `.context/test-results/browser-run.json` |
| 3. **Cập nhật tiến độ** (tự động) | Kiểm tra trùng → ghi registry + cập nhật coverage/test-status — **chỉ sau bước 2** | `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` |

Các nhánh **sinh test** bổ trợ (test sinh ra sẽ chạy qua `/autotest`): `/characterize` (legacy) · `/capture-manual` (case tay) · `/from-cases` (case user) · `/verify-tests` (gate) · `/coverage` (xem tiến độ).

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

### 1. Chạy test — luồng duy nhất `/autotest`
```bash
/autotest
```
- **(1) Chạy ngầm (headless)** — test suite chạy như bình thường, **không bung browser** cho nhanh → **chỉ lưu log** `.context/test-results/headless-run.json`
- **(2) Autotest browser (headed)** — **bung browser thật** cho user theo dõi thao tác; đến bước cuối **lưu kết quả** rồi **GIỮ browser mở** để lại màn hình kết quả
- **(3) Cập nhật tiến độ** — chỉ sau khi browser xong: ghi `test-registry.json` + `.context/coverage.json` + `.context/test-status.json` (tự động, không gõ command)

```bash
/autotest <module>          # giới hạn 1 module
/autotest --no-browser      # CHỈ chạy ngầm khi cần kết quả nhanh
```

### 2. Nhánh sinh test (bổ trợ)

**Test-first (code mới):** `test-writer` sinh test từ `.spec-cache/SPECIFICATIONS.md` cho mọi `R-xx` → chạy **ĐỎ** → (code) → **XANH**. Agent: `test-writer` → `test-reflector`.

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

### 3. Verify chất lượng test
```bash
/verify-tests
```
- Mutation testing (mutmut / Stryker) — mutation score ≥ ngưỡng (mặc định 70%)
- Quality gate — chống expected-từ-code, try-catch nuốt lỗi, assert trivially
- Hidden stash — test ẩn chạy CI, chống overfit

### 4. Tiến độ test (biết đã/chưa test đến đâu) — tự động cập nhật
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
├── AGENT.md              ← pipeline autotest (1 luồng + nhánh sinh test)
├── AGENTS.md             ← router
├── spec-source.json      ← link git tới folder spec của repo DEV (điền khi bắt đầu)
├── test-registry.json    ← manifest BỘ TEST HOÀN CHỈNH (mọi test + nguồn gốc + trạng thái)
├── BRIEF.md
├── opencode.jsonc
├── .spec-cache/          ← spec clone về (gitignored, read-only)
├── docs/
│   ├── FLOWS.md          ← luồng /autotest + hợp đồng test-scope
│   ├── SPEC_VERSIONING.md ← cách link + version spec/test-scope
│   └── generated/        ← inventory (auto-gen)
├── .opencode/
│   ├── agent/            ← test-writer · characterization-writer · manual-capture-writer · scope-planner · spec-source-linker · test-reflector · test-validator
│   └── command/          ← /autotest · /characterize · /capture-manual · /from-cases · /verify-tests · /spec-link · /coverage
├── .agent/               ← spec-validator · workflow
├── skills/               ← property-based-testing · mutation-testing · characterization-golden · manual-to-auto · test-quality-gate · coverage-driven · complete-test-suite
└── scripts/              ← generate-inventory
```

## Nguyên tắc (bất biến)

1. **Spec-first** — test bám spec từ `.spec-cache/` (nguồn duy nhất, không lưu bản riêng → không lệch)
2. **Luồng duy nhất `/autotest`** — chạy ngầm (headless) trước, rồi autotest browser (headed, giữ browser mở); tiến độ cập nhật chỉ sau khi browser xong
3. **Test PHẢI có khả năng bất đồng với code** — mutation score là thước đo, không phải coverage %
4. **Characterization = khóa behavior hiện tại, không phải "đúng"**
5. **Manual case → auto test ngay khi có thể** — chỉ giữ manual-only khi thật cần
6. **Test-scope.json có version** — đọc để test đúng phạm vi + biết đã cover đến đâu; thiếu thì hỏi, không tự đoán rộng
7. **Test sinh bởi AI phải qua gate** — không merge test chưa qua `/verify-tests`
8. **Mọi test mới vào bộ hoàn chỉnh** — tra registry trước, append/cập nhật, không dựng suite song song
