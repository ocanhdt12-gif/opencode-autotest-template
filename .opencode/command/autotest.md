# /autotest — Luồng test DUY NHẤT (ngầm trước → browser sau)

> Một luồng, 3 giai đoạn: **(1) chạy ngầm (headless)** cho nhanh → **(2) autotest với browser (headed, bung hẳn)** để user theo dõi → **(3) cập nhật tiến độ**. Chạy ngầm chỉ **lưu log**; tiến độ chỉ cập nhật **sau khi browser xong**.

## Cách dùng
```
/autotest                 → đầy đủ: ngầm (headless) → browser (headed) → cập nhật tiến độ
/autotest <module>        → giới hạn 1 module
/autotest --no-browser     → CHỈ chạy ngầm (khi chỉ cần kết quả nhanh, không cần xem browser)
```

## Giai đoạn 0 — Chuẩn bị (bắt buộc)
1. `/spec-link --sync` — pull spec mới nhất về `.spec-cache/` (chưa link → hỏi link, không tự bịa spec)
2. Đọc `.spec-cache/spec/test-scope/current.json` (nếu có) → xác định **phạm vi ưu tiên**: `impact.direct` + `dependents` + `regression` + `acceptance`. Không có scope → chạy full suite.
3. `spec-validator` PASS trước khi sinh/chạy test
4. Đọc `.agent/PROJECT_PROFILE.md` — framework test + cấu hình browser

## Giai đoạn 1 — Chạy NGẦM (headless, KHÔNG bung browser)
- Chạy test suite **như bình thường**, chế độ headless cho nhanh (không mở cửa sổ):
  - unit/integration: `npm test` / `npx vitest run` / `pytest`
  - E2E (nếu muốn soi trước): `npx playwright test` (mặc định headless)
- Mục đích: kết quả **nhanh** + phát hiện lỗi cơ bản trước khi mở browser chậm hơn.
- **LƯU kết quả** vào `.context/test-results/headless-run.json`:
  ```jsonc
  { "ranAt": "...", "command": "npx vitest run", "total": 42, "passed": 40, "failed": 2,
    "failures": [{ "test": "...", "file": "...", "error": "...", "refs": ["R-01"] }] }
  ```
- ⚠️ **CHỈ LƯU LOG — KHÔNG cập nhật tiến độ** (`test-registry.json`/`coverage.json`/`test-status.json` chỉ ghi ở Giai đoạn 3, sau browser).
- Nếu có fail → `test-reflector` phân loại (`bug-test` / `bug-code`) và báo. Mặc định vẫn tiếp Giai đoạn 2 để user theo dõi trực quan (trừ khi user dừng).

## Giai đoạn 2 — AUTOTEST với browser (HEADED — bung hẳn ra)
- Chạy E2E Playwright chế độ **headed** (`headless: false`) → **browser mở thật** để user theo dõi từng thao tác.
  - `npx playwright test --headed` + config `use: { headless: false }`; tuỳ chọn `slowMo` để user kịp nhìn.
- **Đến bước cuối cùng:**
  1. **LƯU kết quả** (screenshot + report + json) vào `.context/test-results/browser-run.json`
  2. **KHÔNG đóng browser** — để lại màn hình kết quả cho user xem:
     - Cách chuẩn: gọi `await page.pause()` ở cuối test (mở Playwright Inspector, treo browser), hoặc
     - Fixture `keepBrowserOpen` chạy khi env `LEAVE_BROWSER_OPEN=1` (auto `page.pause()` cho mọi page khi không headless).
- `test-reflector` phân loại fail thu được trên browser (`bug-test` / `bug-code`).

## Giai đoạn 3 — Cập nhật tiến độ (TỰ ĐỘNG — CHỈ sau Giai đoạn 2)
1. **Kiểm tra trùng** rồi ghi/cập nhật `test-registry.json` (test mới/đổi append vào bộ hoàn chỉnh)
2. **Cập nhật tiến độ** `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`)
3. **Cập nhật** `.context/test-status.json` (`specVersionCovered` / `scopeVersionCovered`)
- Không cần gõ command nào (kể cả `/coverage`). **Chạy ngầm KHÔNG cập nhật tiến độ.**

## Rule
- Browser **LUÔN headed** khi chạy autotest (không headless) — để user theo dõi thao tác
- Cuối luồng **GIỮ browser mở** (không `browser.close()`) — để lại màn hình kết quả
- **Chạy ngầm trước** (headless, nhanh) → **browser sau** (headed, cho user xem)
- **Tiến độ chỉ cập nhật sau khi browser test xong** (chạy ngầm chỉ lưu log)
- Test mới **append vào bộ test hoàn chỉnh** — không tạo suite song song
- Không đọc implementation khi viết test; expected từ spec, không từ chạy code
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only) — test không tạo/sinh/sửa spec
