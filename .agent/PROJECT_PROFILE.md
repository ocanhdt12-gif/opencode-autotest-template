# PROJECT_PROFILE — Autotest Template

> Điền khi dùng cho dự án thật (theo template dev). Mặc định: stack linh hoạt (TS/JS + Python).

- **project_name:** (trống — dự án thật)
- **stack_family:** web | mobile | backend | ai — (trống)
- **test_framework:** vitest | pytest | jest | e2e — mặc định `vitest` (TS) + `pytest` (Python)
- **property_testing:** hypothesis (Python) · fast-check (TS)
- **mutation_tool:** mutmut (Python) · stryker (TS)
- **mutation_score_floor:** 70 (ngưỡng mutation score mặc định)
- **hidden_stash_dir:** tests/hidden (test ẩn chống overfit — không đưa vào prompt agent)
- **test_registry:** test-registry.json (manifest BỘ TEST HOÀN CHỈNH — mọi luồng ghi vào TỰ ĐỘNG khi chạy xong, không cần command)
- **coverage_board:** .context/coverage.json (tiến độ test theo req — TỰ CẬP NHẬT cuối mỗi luồng test, không gõ /coverage; danh sách req đọc từ spec, KHÔNG tạo spec)
- **source_roots:** ["src"]
- **manual_cases_dir:** .context/manual-cases/
- **package_manager:** npm | uv | pip — (trống)
- **forbidden_branch:** main
- **branch_model:** staging-direct

## Bắt buộc khi khởi tạo
1. `.spec-cache/SPECIFICATIONS.md` — spec chung (cùng format template dev) hoặc `BRIEF.md` → brainstorm
2. Chạy spec-validator PASS trước khi sinh test
3. Khởi tạo `tests/` + `.context/manual-cases/` + `tests/hidden/` + `test-registry.json` (bộ test hoàn chỉnh)