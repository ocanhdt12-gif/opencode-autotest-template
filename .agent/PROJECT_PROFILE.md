# PROJECT_PROFILE — Autotest Template

> Điền khi dùng cho dự án thật (theo template dev). Mặc định: stack linh hoạt (TS/JS + Python).

- **project_name:** (trống — dự án thật)
- **stack_family:** web | mobile | backend | ai — (trống)
- **test_framework:** vitest | pytest | jest | e2e — mặc định `vitest` (TS) + `pytest` (Python)
- **browser_framework:** playwright (mặc định — E2E, mở browser thật + giữ cửa sổ mở cuối case)
- **browser_headless_first:** true (chạy ngầm headless 1 lượt trước, rồi mới chạy browser headed)
- **browser_case_by_case:** true (chạy browser TỪNG CASE 1, gom theo module, xong 1 case chờ user chọn case tiếp)
- **keep_browser_open:** true (cuối mỗi case KHÔNG đóng browser — để lại màn hình kết quả cho user)
- **test_cases_dir:** .context/test-cases/ (TEST CASE gom theo module — <module>.md, id TC-<module>-NN, draft→approved)
- **test_tasks_file:** .context/test-tasks.json (trạng thái task: case nào đã/chưa test → không test lại)
- **property_testing:** hypothesis (Python) · fast-check (TS)
- **mutation_tool:** mutmut (Python) · stryker (TS)
- **mutation_score_floor:** 70
- **hidden_stash_dir:** tests/hidden
- **test_registry:** test-registry.json (manifest BỘ TEST HOÀN CHỈNH — mỗi test có `testCase`; ghi TỰ ĐỘNG sau khi chạy)
- **coverage_board:** .context/coverage.json (tiến độ theo req — TỰ CẬP NHẬT sau khi chạy; danh sách req đọc từ spec)
- **test_results_dir:** .context/test-results/ (headless-run.json + browser-run.json + bugs.md)
- **source_roots:** ["src"]
- **manual_cases_dir:** .context/manual-cases/
- **package_manager:** npm | uv | pip — (trống)
- **forbidden_branch:** main
- **branch_model:** staging-direct

## Bắt buộc khi khởi tạo
1. `/spec-link <git-url>` — link folder spec của repo DEV (spec KHÔNG lưu ở đây)
2. Chạy spec-validator PASS trước khi sinh test case
3. Khởi tạo `tests/` + `.context/test-cases/` + `.context/test-tasks.json` + `.context/test-results/` + `.context/manual-cases/` + `tests/hidden/` + `test-registry.json`
