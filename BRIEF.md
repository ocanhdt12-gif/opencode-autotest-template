# BRIEF — Autotest Template

> Dùng khi bắt đầu dự án thật: mô tả ngắn dự án + mục tiêu (để brainstorm sinh SPECIFICATIONS.md).

## Dự án
_(tên + 1-2 câu mô tả)_

## Yêu cầu test mong muốn
- Stack test: TS (Vitest) / Python (pytest) — chọn theo dự án
- Nhánh cần dùng: `/autotest` (test-case-first, mặc định) · `/characterize` (legacy) · `/capture-manual` (case test tay) · `/from-cases` (theo case user)
- Mutation score mục tiêu: ≥ 70% (mặc định)

## Mục tiêu
_(kết quả mong muốn khi test suite hoàn chỉnh — vd: "mọi requirement R-xx có test, legacy được khóa behavior, case manual chuyển auto")_