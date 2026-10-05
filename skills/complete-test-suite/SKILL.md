---
name: complete-test-suite
description: "Nuôi MỘT bộ test hoàn chỉnh + giữ tiến độ test — mọi luồng test (full/test-scope/regression/manual/from-cases/characterize) TỰ ĐỘNG bổ sung/cập nhật vào cùng bộ (tests/ + test-registry.json) và TỰ CẬP NHẬT tiến độ (.context/coverage.json + .context/test-status.json) khi chạy xong, không cần gõ command. Spec CHỈ đọc từ link git (read-only) — test không tự tạo/không tự sinh gì thuộc spec; lần sau pull spec về để biết req nào đã/chưa test. Dùng khi chạy bất kỳ luồng test nào."
---

# Một bộ test hoàn chỉnh + tiến độ test (nguyên tắc gốc)

Test chỉ có **1 bộ duy nhất** trong repo. Mọi luồng chỉ để **nuôi** bộ này — vừa **retest tính năng cũ**, vừa **test feature mới**. Không bao giờ có suite riêng cho từng luồng.

> ⚙️ **TỰ ĐỘNG — không có command riêng.** Cuối MỌI luồng test, agent **tự chạy** bước đăng ký + **tự cập nhật tiến độ** dưới đây. Người dùng KHÔNG gõ gì thêm (kể cả `/coverage`). Đây là 1 bước ẩn của luồng, không phải thao tác tay.

## 🔒 Spec: chỉ ĐỌC từ link — test KHÔNG tạo gì thuộc spec

Spec **100% đến từ repo DEV qua link git** (`/spec-link` → `.spec-cache/`, read-only). Phía test:

- **KHÔNG** tự tạo / tự viết / tự sinh `SPECIFICATIONS.md` hay bất cứ file spec nào.
- **KHÔNG** ghi vào `.spec-cache/` (read-only).
- **KHÔNG** copy spec vào repo test (tránh 2 bản lệch).
- Chưa link → **hỏi link**, không tự bịa spec.
- `test-scope` (phạm vi cần test) do **template DEV sinh** ra, test chỉ đọc.

> Test **chỉ** giữ **trạng thái tiến độ test** của chính nó — không giữ spec, không tạo nội dung spec.

## Ai ghi gì

| File | Vai trò | Ai ghi |
|---|---|---|
| `.spec-cache/**` (SPECIFICATIONS.md, spec/test-scope/…) | **Spec — nguồn từ DEV** | ❌ test KHÔNG ghi (read-only, sync qua `/spec-link`) |
| `tests/` (theo PROJECT_PROFILE) | Toàn bộ test thật — **1 bộ duy nhất** | mọi luồng (tự động) |
| `test-registry.json` (root repo TEST) | Manifest bộ test: `file` + `refs` + `origin` + `status` + `lastRunAt` | bước tự động cuối luồng |
| `.context/coverage.json` | **Tiến độ test theo req** (req → covered/failing + `testRef`) | bước tự động cuối luồng |
| `.context/test-status.json` | **Tiến độ theo version** (specVersionCovered/scopeVersionCovered) | bước tự động cuối luồng |

> **Danh sách req** luôn lấy từ spec (`.spec-cache/SPECIFICATIONS.md`); **trạng thái** là do test tự ghi. Board = danh sách req (từ spec) + tiến độ (của test).

## Schema `test-registry.json`

```jsonc
{
  "updatedAt": "2026-10-05T14:30:00+07:00",
  "tests": [
    { "id": "tests/auth.test.ts::login_r01",
      "file": "tests/auth.test.ts",
      "refs": ["R-01"],                       // R-xx (spec) hoặc case id (user/manual)
      "origin": "full | test-scope | regression | manual | from-cases | characterization",
      "tags": ["auth", "smoke"],
      "regression": true,                     // có nằm trong bộ retest luồng cũ không
      "status": "pass | fail | skip",
      "lastRunAt": "2026-10-05T14:35:00+07:00" }
  ]
}
```

## Cơ chế tự động (chạy cuối MỌI luồng)

1. **Thu thập** test mới/đổi từ luồng vừa chạy (file + test name + `refs` + kết quả).
2. **Kiểm tra trùng** (BẮT BUỘC, trước khi ghi) — tra `test-registry.json` + `tests/`:
   - Entry đã có **cùng `refs` VÀ cùng behavior** → **cập nhật** (`status`, `lastRunAt`, `regression`; `origin` giữ nguyên gốc).
   - Entry chưa có → **append**.
   - Test mới phủ requirement đã có test khác phủ → ghi chú, ưu tiên **bổ sung** vào test hiện có thay vì tạo bản sao.
   - **Không** bao giờ thêm 2 entry trùng `id`/`refs`+behavior.
3. **Ghi/append vào `tests/`** — test mới vào file hiện có theo module, hoặc file mới nhưng thuộc `tests/`; KHÔNG dựng thư mục/suite riêng cho luồng.
4. **Cập nhật TIẾN ĐỘ test** (không gõ `/coverage`):
   - **Nguồn req** = `.spec-cache/SPECIFICATIONS.md` (đọc, không ghi).
   - Board `.context/coverage.json` thiếu/chưa có → **khởi tạo danh sách req từ spec** (mọi `R-xx` = `untested`) — chỉ liệt kê, không tạo nội dung spec.
   - **Re-sync danh sách req với spec:** req mới trong spec chưa có trong board → thêm `pending`/`untested`; req đã bị bỏ khỏi spec → đánh dấu `n/a`.
   - Req trong `refs` của test vừa chạy → `covered` (pass) / `failing` (fail) + `testRef` + `lastRunAt`.
   - **Không** "tự nhận" covered khi test chưa chạy thật.
5. **Cập nhật tiến độ version** `.context/test-status.json` (`specVersionCovered`/`scopeVersionCovered` + `lastRun`).
6. **Đối chiếu version:** spec hiện tại (`.spec-cache/SPECIFICATIONS.md`) version > `specVersionCovered` → còn phần spec mới chưa test → nhắc chạy `/test-scope` hoặc `/autotest --full`.

> **Mục đích tiến độ:** lần sau `/spec-link --sync` pull spec mới về → so board (`coverage.json`) + test-status → biết ngay **req nào đã test, req nào còn thiếu** — không test lại mù.

## Bất biến

- **Spec read-only:** test không tạo/sửa spec; mọi req lấy từ `.spec-cache/` (link git).
- **Tự động:** không cần người dùng gõ command; mỗi luồng tự kết thúc bằng bước này (đăng ký test **+ cập nhật tiến độ**).
- **Additive:** sau mỗi luồng, bộ test **chỉ tăng/cập nhật**, không bị luồng khác ghi đè/xoá.
- **Không trùng:** kiểm tra trùng trước khi ghi; cùng `refs`+behavior → cập nhật, không thêm bản sao.
- **Tiến độ không cập nhật tay:** `.context/coverage.json` + `.context/test-status.json` chỉ do bước tự động cuối luồng ghi; `/coverage` = xem báo cáo.
- **Không suite song song:** cấm tạo `tests-full/`, `tests-regression/`... tách khỏi bộ chính.
- **Regression = 1 phần của bộ:** luồng 3 chỉ chạy lại entry `regression: true` (hoặc `--all`), cập nhật `status`/`lastRunAt` — không sinh test mới trừ khi `test-reflector` phát hiện thiếu.
- **Traceable:** mọi entry có `refs` — bám `R-xx` (spec) hoặc case id (user/manual).

## Nối vào các luồng (tự động, không command)

| Luồng | Sinh gì | Vào bộ + tiến độ (tự động) |
|---|---|---|
| 1 `/autotest --full` | test mọi `R-xx` | ✅ append (`origin=full`) + cập nhật tiến độ |
| 2 `/test-scope` | test cho `direct`/`acceptance` vừa đổi | ✅ append/cập nhật (`origin=test-scope`) + cập nhật tiến độ |
| 3 `/regression` | retest luồng cũ | ♻️ chỉ cập nhật `status`/`lastRunAt` + tiến độ |
| 4 `/capture-manual` | case tay → auto | ✅ append (`origin=manual`) + `regression=true` + tiến độ |
| 5 `/from-cases` | test theo case user | ✅ append (`origin=from-cases`) + `regression=true` + tiến độ |
| B `/characterize` | golden test legacy | ✅ append (`origin=characterization`) + tiến độ |

## Output (mỗi luồng tự báo)
- `test-registry.json` cập nhật (kèm kết quả kiểm tra trùng)
- `.context/coverage.json` + `.context/test-status.json` cập nhật (tiến độ test)
- Báo: thêm N test mới · cập nhật M · trùng bỏ qua K · tiến độ: covered X/Y req · req còn `pending/untested`
