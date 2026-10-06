# /autotest — Tạo test case mới từ spec + chạy luồng mới (KHÔNG retest)

> Luồng **test tính năng MỚI**: **check spec → tạo test case cho spec mới → chạy ngầm (headless) → autotest browser (headed, giữ mở) → cập nhật tiến độ**.
> Chỉ chạy phần **MỚI** (theo spec / test-scope) — **KHÔNG retest lại toàn bộ**. Muốn retest luồng cũ dùng `/retest`.

## Cách dùng
```
/autotest                  → tạo test case cho spec mới + chạy phần mới
/autotest <module>         → giới hạn 1 module
/autotest --no-browser      → chỉ chạy ngầm (kết quả nhanh, không cần xem browser)
```

## Giai đoạn 0 — Chuẩn bị (bắt buộc)
1. `/spec-link --sync` — pull spec mới nhất về `.spec-cache/` (chưa link → hỏi link, không tự bịa spec)
2. Đọc `.spec-cache/spec/test-scope/current.json` (nếu có) → xác định **phạm vi mới**: `specRefs` + `impact.direct` + `acceptance`. Không có scope → so spec version để tìm phần mới.
3. `spec-validator` PASS trước khi sinh test
4. Đọc `.agent/PROJECT_PROFILE.md` — framework test + cấu hình browser

## Giai đoạn 1 — TẠO TEST CASE từ spec mới (bắt buộc)
- **Check spec**: đọc `.spec-cache/SPECIFICATIONS.md` + test-scope → lập danh sách `R-xx` **mới / chưa có test** (đối chiếu `.context/coverage.json` + `test-registry.json`).
- Gọi **`test-writer`** → sinh test case cho từng `R-xx` mới (unit + property-based), **traceable** về requirement.
  - Expected **từ spec**, KHÔNG lấy từ chạy code; không try-catch nuốt lỗi; không assert trivially.
- Ghi mapping test ↔ `R-xx` vào `.context/test-plan.md`.
- **Gate**: mọi `R-xx` mới có ≥1 test case. Thiếu → bổ sung trước khi chạy.

## Giai đoạn 2 — Chạy NGẦM (headless, KHÔNG bung browser)
- Chạy test **phần mới** như bình thường, chế độ headless cho nhanh (không mở cửa sổ):
  - unit/integration: `npm test` / `npx vitest run` / `pytest`
  - E2E (nếu muốn soi trước): `npx playwright test` (mặc định headless)
- **LƯU kết quả** vào `.context/test-results/headless-run.json`:
  ```jsonc
  { "ranAt": "...", "command": "npx vitest run", "total": 42, "passed": 40, "failed": 2,
    "failures": [{ "test": "...", "file": "...", "error": "...", "refs": ["R-01"] }] }
  ```
- ⚠️ **CHỈ LƯU LOG — KHÔNG cập nhật tiến độ** (tiến độ chỉ ghi ở Giai đoạn 4, sau browser).
- Fail → `test-reflector` phân loại (`bug-test` / `bug-code`). Mặc định vẫn tiếp Giai đoạn 3 để user theo dõi trực quan.

## Giai đoạn 3 — AUTOTEST với browser (HEADED — bung hẳn ra)
- Chạy E2E Playwright chế độ **headed** (`headless: false`) → **browser mở thật** để user theo dõi từng thao tác.
  - `npx playwright test --headed` + config `use: { headless: false }`; tuỳ chọn `slowMo` để user kịp nhìn.
- **Đến bước cuối cùng:**
  1. **LƯU kết quả** (screenshot + report + json) vào `.context/test-results/browser-run.json`
  2. **KHÔNG đóng browser** — để lại màn hình kết quả cho user xem:
     - Cách chuẩn: `await page.pause()` ở cuối test (mở Playwright Inspector, treo browser), hoặc
     - Fixture `keepBrowserOpen` chạy khi env `LEAVE_BROWSER_OPEN=1` (auto `page.pause()` cho mọi page khi không headless).
- `test-reflector` phân loại fail thu được trên browser (`bug-test` / `bug-code`).

## Giai đoạn 4 — Cập nhật tiến độ (TỰ ĐỘNG — CHỈ sau Giai đoạn 3)
1. **Kiểm tra trùng** rồi ghi/cập nhật `test-registry.json` (test case mới append vào bộ hoàn chỉnh)
2. **Cập nhật tiến độ** `.context/coverage.json` (req → `covered`/`failing` + `testRef` + `lastRunAt`)
3. **Cập nhật** `.context/test-status.json` (`specVersionCovered` / `scopeVersionCovered`)
- Không cần gõ command nào (kể cả `/coverage`). **Chạy ngầm KHÔNG cập nhật tiến độ.**

## Rule
- **Tạo test case trước, chạy sau** — luôn có bước 1 sinh test theo spec mới
- Browser **LUÔN headed** khi chạy autotest; cuối luồng **GIỮ browser mở** (không `browser.close()`)
- **Chạy ngầm trước** (headless, nhanh) → **browser sau** (headed, cho user xem)
- **Tiến độ chỉ cập nhật sau khi browser test xong** (chạy ngầm chỉ lưu log)
- **KHÔNG retest luồng cũ** ở đây — việc đó thuộc `/retest`
- Test case mới **append vào bộ test hoàn chỉnh** — không tạo suite song song
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only) — test không tạo/sinh/sửa spec
