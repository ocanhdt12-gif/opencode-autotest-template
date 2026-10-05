# opencode-autotest-template

Template OpenCode chuyên **sinh + duy trì Auto Test** cho AI agent — viết test từ spec trước khi code, khóa behavior legacy, và chụp test manual thành regression test.

> **3 nhánh hoạt động:**
> - **A. Test-first generation** — code MỚI: spec → viết test TRƯỚC → verify ĐỎ → code → XANH
> - **B. Characterization** — legacy CHƯA có test: khóa behavior hiện tại bằng golden test → an toàn sửa/refactor
> - **C. Manual→Auto capture** — case đã test TAY xong: chụp thành auto test → lần sau chỉ chạy lại
>
> **Match với template dev:** chỉ **SPEC là cái chung** — cả 2 template cùng dùng `SPECIFICATIONS.md` (format giống nhau); test và code cùng bám spec. Error handling mỗi template tự quản lý, không cầu nối.

## Quickstart

```bash
git clone git@github.com:ocanhdt12-gif/opencode-autotest-template.git
cd opencode-autotest-template
npx opencode
```

## Step-by-step

### 1. Nạp Spec (dùng chung với template dev)
- Đặt `SPECIFICATIONS.md` vào repo (hoặc `BRIEF.md` → brainstorm sinh spec)
- Chạy spec-validator → đảm bảo spec PASS trước khi sinh test
- Mỗi requirement trong spec có **id** (vd `R-01`) — test traceable về spec

### 2. Nhánh A — Sinh test từ spec (code mới)
```bash
/autotest <layer-or-module>
```
1. `test-writer` đọc spec → viết test trước (unit + property-based), KHÔNG đọc implementation
2. Chạy test → phải **ĐỎ** (đúng cách — failing vì chưa có code)
3. (Đưa sang) code implement tới khi test **XANH**
4. `/verify-tests` — mutation + quality gate chặn test "vô hại"

### 3. Nhánh B — Characterization (legacy chưa test)
```bash
/characterize <path-to-file-or-module>
```
1. `characterization-writer` đọc code → sinh **golden/snapshot test** khóa behavior hiện tại
2. Scrub trường unstable (timestamp, id, random)
3. **Mutation check** — cố tình phá code → test PHẢI bắt được (nếu không, test vô nghĩa)
4. Xong → an toàn để sửa/refactor

### 4. Nhánh C — Manual→Auto capture (chụp test đã test tay)
```bash
/capture-manual <case-id-or-path>
```
Khi bạn/test manual đã verify 1 case xong, chụp lại thành auto test:
1. Mô tả case (input / expected / bước kiểm tra) vào `.context/manual-cases/`
2. Agent viết test tự động hóa đúng case đó (dùng test framework của repo)
3. Chạy → xanh → lưu vào `tests/` → từ nay chạy `npm test`/`pytest` là biết case đó còn pass không
4. Case nào **không tự động hóa được** (cần người xác nhận, visual...) → đánh dấu `manual-only` + lý do, không ép

### 5. Verify chất lượng test
```bash
/verify-tests
```
- Chạy toàn bộ test suite
- **Mutation testing** (mutmut / Stryker) — mutation score = test có bắt bug thật không
- **Quality gate** — chống: expected lấy từ chạy code, try-catch nuốt lỗi, assert trivially pass
- **Hidden stash** — test ẩn (không nằm trong prompt agent) chạy trong CI, chống agent overfit

### 6. Vòng lặp lỗi test
- Test fail → `test-reflector` phân loại: bug trong TEST (sửa test) / bug trong CODE (báo loop agent fix theo spec)
- Ghi chú ngắn `.context/test-notes.md` nếu là lỗi đáng nhớ (tùy chọn)

## Cấu trúc thư mục

```
├── AGENT.md              ← pipeline autotest (3 nhánh)
├── AGENTS.md             ← router (bug/feature/review/autotest)
├── SPECIFICATIONS.md     ← spec chung (code + test dùng chung)
├── BRIEF.md
├── opencode.jsonc        ← permission gate
├── .opencode/
│   ├── agent/            ← test-writer · characterization-writer · manual-capture-writer · test-reflector · test-validator
│   └── command/          ← /autotest · /characterize · /capture-manual · /verify-tests
├── .agent/               ← spec-validator · workflow (match template dev — spec là cái chung)
├── skills/               ← property-based-testing · mutation-testing · characterization-golden · manual-to-auto · test-quality-gate · coverage-driven
├── scripts/              ← generate-inventory · mutation-scan
└── docs/generated/       ← inventory (auto-gen)
```

## Nguyên tắc (bất biến)

1. **Spec-first** — test và code cùng bám `SPECIFICATIONS.md`, không "test xác nhận bug"
2. **Test PHẢI có khả năng bất đồng với code** — mutation score là thước đo, không phải coverage %
3. **Characterization = khóa behavior hiện tại, không phải khóa "đúng"** — test không chứng minh code đúng, nó chứng minh hành vi không đổi
4. **Manual case → auto test ngay khi có thể** — giảm test tay lặp lại, chỉ giữ manual-only khi thật cần
5. **Spec là cái chung duy nhất với template dev** — test và code cùng bám `SPECIFICATIONS.md`
6. **Test sinh bởi AI phải qua gate** — không merge test chưa qua verify-tests