# opencode-autotest-template

Template OpenCode chuyên **sinh + duy trì Auto Test** — viết test từ spec trước khi code, khóa behavior legacy, chụp test manual, và test theo case user tạo.

> **Match với template dev:** **SPEC là cái chung duy nhất** — nhưng template test **KHÔNG lưu spec**; nó giữ **link git** tới folder spec của repo DEV (`spec-source.json` + `/spec-link <git-url>`) → sync về `.spec-cache/` mỗi lần chạy, tránh 2 bản spec lệch nhau. Template DEV sinh `.spec-cache/...` — xem `docs/SPEC_VERSIONING.md`.

## 5 luồng test (chi tiết: `docs/FLOWS.md`)

| # | Luồng | Command | Khi nào |
|---|---|---|---|
| 1 | **Full lần đầu** | `/autotest --full` | Code xong lần đầu từ template dev → test toàn bộ spec |
| 2 | **Phần vừa sửa** 🔑 | `/test-scope` | Vừa fix bug / thêm feature → test đúng phạm vi (đọc `.spec-cache/spec/test-scope/current.json` từ dev) |
| 3 | **Regression** | `/regression` | Sau update → retest luồng cũ, đảm bảo không vỡ |
| 4 | **Manual → Auto** | `/capture-manual` | Case đã test tay xong → chụp thành auto test |
| 5 | **Theo case user** | `/from-cases` | User/khách đưa bộ test case → chuyển thành auto test |
| — | **Độ phủ** | `/coverage` | Xem req nào đã/chưa test (board `.context/coverage.json` — do test tự lưu) |

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
/spec-link git@github.com:org/dev-repo.git          # clone shallow+sparse folder spec về .spec-cache/
/spec-link --sync                                   # pull spec mới nhất mỗi lần chạy test
/spec-link --status                                 # xem đang link đâu, spec version nào
```
- Đọc requirements từ `.spec-cache/SPECIFICATIONS.md`, scope từ `.spec-cache/spec/test-scope/current.json`
- Chưa link → hỏi, không tự bịa spec

### 1. Luồng 1 — Full test lần đầu
```bash
/autotest --full
```
`test-writer` sinh test cho **mọi** `R-xx` → chạy **ĐỎ** → (code) → **XANH** → `/verify-tests`

### 2. Luồng 2 — Test phần vừa sửa 🔑
```bash
/test-scope
```
- Đọc `.spec-cache/spec/test-scope/current.json` (template DEV sinh, có `specVersion`+`scopeVersion`) → `scope-planner` phân loại:
  - `impact.direct` → sinh test mới
  - `impact.dependents` → test lại
  - `acceptance` → đảm bảo có test
- Đối chiếu version: spec hiện tại > `specVersionCovered` → còn phần mới chưa cover
- `risk: high` → mutation verify bắt buộc
- Xong → cập nhật `.context/test-status.json`

### 3. Luồng 3 — Regression
```bash
/regression
```
Chạy lại test **đã có** thuộc `impact.regression` + dependents → xác nhận luồng cũ không vỡ.

### 4. Luồng 4 — Manual → Auto
```bash
/capture-manual <case-id>
```
Test tay xong → sinh auto test → XANH → **add vào regression suite**. Không tự động hoá được → `manual-only` + lý do.

### 5. Luồng 5 — Test theo case user
```bash
/from-cases <path>
```
Đọc file test case user (md/csv/sheet) → chuyển từng case thành auto test → báo pass/fail/không tự động hoá được. Bám đúng case user, không tự bịa.

### 6. Nhánh bổ trợ — Characterization (legacy chưa test)
```bash
/characterize <path>
```
Golden test khóa behavior hiện tại (scrub timestamp/id/random) → mutation verify → an toàn refactor.

### 7. Verify chất lượng test
```bash
/verify-tests
```
- Mutation testing (mutmut / Stryker) — mutation score ≥ ngưỡng
- Quality gate — chống expected-từ-code, try-catch nuốt lỗi, assert trivially
- Hidden stash — test ẩn chạy CI, chống overfit

### 8. Độ phủ test (biết đã/chưa test đến đâu)
```bash
/coverage --init     # khởi tạo board từ spec (.spec-cache/SPECIFICATIONS.md → mọi R-xx = untested)
/coverage            # xem board: req nào covered/pending/untested/failing
/coverage --gaps     # chỉ phần CHƯA test
```
- Board **thuộc template TEST**: lưu ở `.context/coverage.json` (repo test) — vì chỉ test mới biết nó đã chạy gì
- Danh sách req lấy từ spec (qua link git); **trạng thái do test tự cập nhật** sau mỗi lần chạy
- DEV không giữ board này

## Cấu trúc thư mục

```
├── AGENT.md              ← pipeline autotest (5 luồng + 3 nhánh sinh test)
├── AGENTS.md             ← router
├── spec-source.json      ← link git tới folder spec của repo DEV (điền khi bắt đầu)
├── BRIEF.md
├── opencode.jsonc
├── .spec-cache/          ← spec clone về (gitignored, read-only)
├── docs/
│   ├── FLOWS.md          ← 5 luồng + hợp đồng test-scope
│   ├── SPEC_VERSIONING.md ← cách link + version spec/test-scope
│   └── generated/        ← inventory (auto-gen)
├── .opencode/
│   ├── agent/            ← test-writer · characterization-writer · manual-capture-writer · scope-planner · spec-source-linker · test-reflector · test-validator
│   └── command/          ← /autotest · /test-scope · /regression · /capture-manual · /from-cases · /characterize · /verify-tests · /spec-link · /coverage
├── .agent/               ← spec-validator · workflow
├── skills/               ← property-based-testing · mutation-testing · characterization-golden · manual-to-auto · test-quality-gate · coverage-driven · test-scope-contract
└── scripts/              ← generate-inventory
```

## Nguyên tắc (bất biến)

1. **Spec-first** — test bám spec từ `.spec-cache/` (nguồn duy nhất, không lưu bản riêng → không lệch)
2. **Test PHẢI có khả năng bất đồng với code** — mutation score là thước đo, không phải coverage %
3. **Characterization = khóa behavior hiện tại, không phải "đúng"**
4. **Manual case → auto test ngay khi có thể** — chỉ giữ manual-only khi thật cần
5. **Test-scope.json có version** — đọc để test đúng phạm vi + biết đã cover đến đâu; thiếu thì hỏi, không tự đoán rộng
6. **Test sinh bởi AI phải qua gate** — không merge test chưa qua `/verify-tests`
7. **Luồng 4/5 tự add vào regression** — mọi test mới trở thành lưới an toàn cho lần sau