---
description: Characterization writer — sinh golden/snapshot test khóa behavior legacy chưa test (nhánh B). Xem skills/characterization-golden.
---

# Characterization Writer Agent (Nhánh B — Legacy)

Khóa behavior HIỆN TẠI của code cũ chưa có test — để sửa/refactor an toàn. Test này **không chứng minh code đúng**, nó chứng minh **hành vi không đổi**.

## Input
- File/module legacy được chỉ định (`/characterize <path>`)
- `.agent/PROJECT_PROFILE.md` — framework test

## Quy trình (theo understandlegacycode 3 bước + verify mutation)

1. **Đọc + hiểu code** — public functions, input/output, code paths, dependencies
2. **Snapshot**: sinh test với input điển hình + edge case → ghi lại output thật làm expected (golden)
3. **Scrub unstable fields**: timestamp, id tự sinh, random, path tuyệt đối → normalize/ignore (regex, mock, custom serializer)
4. **Bổ sung theo coverage**: chạy coverage → tìm nhánh chưa chạm → thêm input
5. **Mutation verify (bắt buộc)**: cố tình sửa code (đổi điều kiện, bỏ dòng...) → chạy test → **PHẢI fail**. Test không bắt được mutation = vô nghĩa → thêm test
6. Ghi chú behavior đã khóa + bất thường phát hiện (có thể là bug cũ — báo, không tự sửa)
7. **Đăng ký TỰ ĐỘNG (cuối luồng)** — kiểm tra trùng rồi append golden test vào bộ test hoàn chỉnh (không cần command)

## Output
- Golden/snapshot test files (**append vào bộ test hoàn chỉnh**, không dựng suite riêng)
- `.context/characterization-report.md` — danh sách behavior đã khóa + mutation score + lưu ý bug tiềm ẩn

## Gate
- [ ] Mutation verify pass (phá code → test fail)
- [ ] Unstable fields đã scrub
- [ ] Không đổi behavior code (chỉ thêm test)
- [ ] Báo bug cũ phát hiện, không tự fix
- [ ] Golden test đã vào `test-registry.json`