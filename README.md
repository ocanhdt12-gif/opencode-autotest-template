# opencode-autotest-template

Template OpenCode chuyên **sinh + duy trì Auto Test** — test bám spec qua **test case do user chốt**, chạy ngầm rồi chạy browser cho user kiểm tra từng case.

> **Match với template dev:** **SPEC là cái chung duy nhất** — nhưng template test **KHÔNG lưu spec**; nó giữ **link git** tới folder spec của repo DEV (`spec-source.json` + `/spec-link <git-url>`) → sync về `.spec-cache/` mỗi lần chạy.

> ⭐ **Luật trục — TEST-CASE-FIRST:** **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.** Mọi test phải follow test case; test code chỉ hiện thực hoá đúng test case user đã chốt (`skills/test-case-first`).

> ⭐ **2 lệnh độc lập:**
> - **`/autotest`** — test tính năng **MỚI**: (lần đầu) config spec → overview → brainstorm → **tạo test case** → **chờ user chốt** → chạy ngầm (headless, log bug) → **chạy browser TỪNG CASE** cho user kiểm tra → cập nhật trạng thái. Lần sau: lấy task chưa test → tạo test case → chạy.
> - **`/retest`** — **chạy lại** test đã có: `--all` (toàn hệ thống) · `<module>` (1 cụm chức năng) · `<TC-xx>` (1 test case). Không tạo test mới.
>
> Cả 2 dùng chung cơ chế: **chạy ngầm (headless) trước** → **browser (headed, bung hẳn, giữ mở)** sau → **cập nhật trạng thái**.

## Quickstart

```bash
git clone git@github.com:ocanhdt12-gif/opencode-autotest-template.git
cd opencode-autotest-template
npx opencode
```

## Step-by-step

### 0. Config spec (lần đầu) — nhập link git
```bash
/spec-link git@github.com:org/dev-repo.git          # clone shallow folder spec về .spec-cache/ (chỉ lần đầu)
/spec-link --status                                 # xem đang link đâu, spec version nào
```
Template test **không lưu spec** — chỉ trỏ tới folder spec trong repo DEV. Chưa link → `/autotest` sẽ **hỏi** (không tự bịa spec).

> ⚙️ **Sync spec TỰ ĐỘNG:** từ lần thứ 2, spec được **tự pull** (`git -C .spec-cache pull --ff-only`) ngay khi bắt đầu `/autotest` hoặc `/retest` — **KHÔNG cần gõ `/spec-link --sync` thủ công**.

### 1. Test tính năng MỚI — `/autotest`
```bash
/autotest
```

**Lần đầu:**
1. **Config spec** — chưa link thì hỏi `/spec-link <git-url>`
2. **Config web URL** — chưa có `web_app_url` thì hỏi **link deploy web** (vd `https://staging.example.com`) → điền `.agent/PROJECT_PROFILE.md` — đây là **baseURL** cho test browser/E2E
3. **Đọc spec + nêu overview** — spec version, các module, danh sách `R-xx`, phần chưa có test
4. **Brainstorm câu hỏi** — chỗ spec mơ hồ (expected, edge case, ưu tiên) → hỏi user (kèm link deploy web nếu chưa có)
5. **Tạo TEST CASE** (`TC-<module>-NN`) gom theo module → `.context/test-cases/<module>.md`, `Status: draft` — **dạng bảng** (mỗi dòng 1 case); task mới: cùng module → **update file cũ** (thêm dòng), module mới → **tạo file mới**
6. ⛔ **Chờ user sửa + update test case** → chốt (`Status: approved`)
7. **Chạy ngầm (headless)** theo test case → `headless-run.json` + **log bug ra file** nếu có
8. **Chạy browser TỪNG CASE** (headed, bung hẳn ra) — gom theo module, mỗi lần 1 case, xong **giữ browser mở** và **chờ user chọn case tiếp**
9. **Cập nhật trạng thái task** đã test → lần sau không test lại

**Các lần sau:** `/autotest` → đọc spec → lấy task **chưa test** → tạo test case → chạy lại luồng trên.

```bash
/autotest <module>          # giới hạn 1 module
/autotest --no-browser      # CHỈ chạy ngầm (headless)
```

### 2. Retest lại test đã có — `/retest`
```bash
/retest --all                # retest LẠI TOÀN BỘ hệ thống
/retest <module|feature>     # retest 1 cụm chức năng
/retest <TC-xx>              # retest 1 test case
```
Chạy lại test **đã có** (không sinh test mới), cập nhật trạng thái.

### 3. Nhánh sinh test bổ trợ

**Characterization (legacy chưa test):** `/characterize <path>` — golden test khóa behavior hiện tại → mutation verify.

**Manual → Auto (case đã test tay):** `/capture-manual <case-id>` — test tay xong → sinh auto test → XANH → đánh dấu regression.

**Theo case user:** `/from-cases <path>` — đọc file test case user → chuyển thành auto test, bám đúng case user.

### 4. Verify chất lượng test
```bash
/verify-tests
```
- Mutation testing (mutmut / Stryker) — mutation score ≥ ngưỡng (mặc định 70%)
- Quality gate — chống expected-từ-code, try-catch nuốt lỗi, assert trivially
- Hidden stash — test ẩn chạy CI, chống overfit

### 5. Tiến độ test — tự động cập nhật
```bash
/coverage            # XEM: req/case nào covered/pending/untested/failing
/coverage --gaps     # chỉ phần CHƯA test
```
Tiến độ **tự cập nhật sau khi chạy** (không gõ `/coverage` để cập nhật). **Spec chỉ đọc từ link** — test không tạo gì thuộc spec.

## Cấu trúc thư mục

```
├── AGENT.md              ← pipeline autotest (test-case-first)
├── AGENTS.md             ← router
├── spec-source.json      ← link git tới folder spec của repo DEV
├── test-registry.json    ← manifest BỘ TEST HOÀN CHỈNH (mọi test + testCase + trạng thái)
├── BRIEF.md
├── opencode.jsonc
├── .spec-cache/          ← spec clone về (gitignored, read-only)
├── .context/
│   ├── test-cases/       ← TEST CASE dạng BẢNG theo module (<module>.md, TC-xx, draft→approved; task mới: cùng module=update file cũ, module mới=file mới)
│   ├── test-tasks.json   ← trạng thái task (case nào đã/chưa test)
│   ├── test-results/     ← headless-run.json · browser-run.json · bugs.md
│   ├── coverage.json     ← tiến độ theo requirement
│   └── test-status.json  ← tiến độ theo spec version
├── docs/
│   ├── FLOWS.md          ← luồng /autotest + /retest + hợp đồng test-scope
│   ├── SPEC_VERSIONING.md ← cách link + version spec/test-scope
│   └── generated/        ← inventory (auto-gen)
├── .opencode/
│   ├── agent/            ← test-case-author · test-writer · characterization-writer · manual-capture-writer · test-reflector
│   └── command/          ← /autotest · /retest · /characterize · /capture-manual · /from-cases · /verify-tests · /spec-link · /coverage
├── .agent/               ← spec-validator · workflow (FEATURE_WORKFLOW + PROJECT_PROFILE)
├── skills/               ← test-case-first · property-based-testing · mutation-testing · characterization-golden · manual-to-auto · test-quality-gate · coverage-driven · complete-test-suite · test-scope-contract
└── scripts/              ← generate-inventory
```

## Nguyên tắc (bất biến)

1. **Test-case-first** — SPEC → TEST CASE (user chốt) → TEST CODE → chạy; mọi test follow test case
2. **Spec-first** — test bám spec từ `.spec-cache/` (nguồn duy nhất, không lưu bản riêng)
3. **2 lệnh độc lập** — `/autotest` (tạo test case mới + chạy) và `/retest` (chạy lại: all/cụm/case)
4. **Cơ chế chạy chung** — ngầm (headless) trước → browser (headed, từng case, giữ mở) sau
5. **Không test lại cái đã test** — trạng thái task lưu ở `.context/test-tasks.json`
6. **Test PHẢI có khả năng bất đồng với code** — mutation score là thước đo, không phải coverage %
7. **Characterization = khóa behavior hiện tại, không phải "đúng"**
8. **Test sinh bởi AI phải qua gate** — không merge test chưa qua `/verify-tests`
9. **Mọi test mới vào bộ hoàn chỉnh** — tra registry trước, append/cập nhật, không dựng suite song song
