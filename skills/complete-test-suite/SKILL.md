---
name: complete-test-suite
description: "Nuôi MỘT bộ test hoàn chỉnh + giữ trạng thái test — mọi test phải follow TEST CASE (skills/test-case-first); sau khi chạy, TỰ ĐỘNG bổ sung/cập nhật vào cùng bộ (tests/ + test-registry.json, mỗi test có testCase=TC-xx) và cập nhật trạng thái (.context/test-tasks.json + .context/coverage.json + .context/test-status.json), không cần gõ command. Spec CHỈ đọc từ link git (read-only) — test không tự tạo gì thuộc spec. Dùng khi chạy /autotest hoặc /retest."
---

# Một bộ test hoàn chỉnh + tiến độ test (nguyên tắc gốc)

Test chỉ có **1 bộ duy nhất** trong repo. Mọi test chỉ để **nuôi** bộ này — vừa **retest tính năng cũ**, vừa **test feature mới**. Không bao giờ có suite riêng.

> ⚙️ **TỰ ĐỘNG — không có command riêng.** Sau khi **browser test (`/autotest` giai đoạn 2) chạy xong**, agent **tự chạy** bước đăng ký + **tự cập nhật tiến độ** dưới đây. Người dùng KHÔNG gõ gì thêm (kể cả `/coverage`). **Chạy ngầm (headless) ở giai đoạn 1 chỉ LƯU LOG — không cập nhật tiến độ.**

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
| `tests/` (theo PROJECT_PROFILE) | Toàn bộ test thật — **1 bộ duy nhất** | mọi nhánh sinh test (tự động) |
| `test-registry.json` (root repo TEST) | Manifest bộ test: `file` + `refs` + `origin` + `status` + `lastRunAt` | bước tự động sau browser test |
| `.context/coverage.json` | **Tiến độ test theo req** (req → covered/failing + `testRef`) | bước tự động sau browser test |
| `.context/test-status.json` | **Tiến độ theo version** (specVersionCovered/scopeVersionCovered) | bước tự động sau browser test |
| `.context/test-results/headless-run.json` | Kết quả **chạy ngầm** (headless) — chỉ log | giai đoạn 1 của `/autotest` |
| `.context/test-results/browser-run.json` | Kết quả **browser test** (headed) | giai đoạn 2 của `/autotest` |

> **Danh sách req** luôn lấy từ spec (`.spec-cache/SPECIFICATIONS.md`); **trạng thái** là do test tự ghi. Board = danh sách req (từ spec) + tiến độ (của test).

## Khi nào chạy bước tự động

| Thời điểm | Cập nhật tiến độ? |
|---|---|
| `/autotest` giai đoạn 1 — chạy ngầm (headless) | ❌ **KHÔNG** — chỉ lưu `.context/test-results/headless-run.json` |
| `/autotest` giai đoạn 2 — browser (headed) xong | ✅ **CÓ** — registry + coverage + test-status |

## Schema `test-registry.json`

```jsonc
{
  "updatedAt": "2026-10-05T14:30:00+07:00",
  "tests": [
    { "id": "tests/auth.test.ts::login_r01", "file": "tests/auth.test.ts",
      "refs": ["R-01"], "requirement": "R-01",
      "origin": "autotest | manual | from-cases | characterization",
      "tags": ["auth", "smoke"], "regression": true,
      "status": "pass", "lastRunAt": "2026-10-05T14:35:00+07:00" }
  ]
}
```

## Quy trình (bước tự động sau browser test)

1. **Kiểm tra trùng**: tra `test-registry.json` — đã có test phủ cùng `requirement` + cùng behavior → **cập nhật** thay vì thêm bản sao.
2. **Ghi registry**: test mới/đổi → append/cập nhật (`origin` = nguồn sinh; `status` + `lastRunAt` từ lần chạy browser).
3. **Cập nhật tiến độ**:
   - Board thiếu → khởi tạo danh sách req từ spec (mọi `R-xx` = `untested`) — chỉ liệt kê, không tạo nội dung spec.
   - req trong `refs` vừa chạy → `covered`/`failing` + `testRef` + `lastRunAt`.
   - req mới trong spec → `pending`/`untested`; req bỏ khỏi spec → `n/a`.
   - Ghi `.context/coverage.json` + `.context/test-status.json`.
4. **Không tạo suite song song** — test mới append vào bộ hiện có.

## Rules

- Tiến độ **chỉ cập nhật sau browser test** — chạy ngầm chỉ lưu log.
- Không "tự nhận" covered khi chưa chạy test thật.
- Mọi nhánh sinh test (test-first / characterization / manual / from-cases) đều đổ vào **cùng 1 bộ**.
- Spec **read-only** từ `.spec-cache/` — test chỉ ghi tiến độ của chính nó.
