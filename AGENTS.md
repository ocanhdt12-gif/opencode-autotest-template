# AGENTS.md — Router (Autotest Template)

Entry point mọi session. Phân loại intent rồi route.

## 2 lệnh độc lập (xem `docs/FLOWS.md`)

| User nói | Route |
|---|---|
| "test tính năng mới" / "spec mới, tạo test case đi" / "vừa update spec, làm test cho phần mới" | **`/autotest`** — check spec → **tạo test case cho spec mới** → chạy ngầm (headless) → autotest browser (headed, giữ mở) → cập nhật tiến độ. **KHÔNG retest luồng cũ.** |
| "retest lại đi" / "chạy lại test xem còn xanh không" / "test lại toàn bộ" / "test lại chức năng X" | **`/retest --full`** (toàn bộ) hoặc **`/retest <feature>`** (1 chức năng) — chạy lại test **đã có**, KHÔNG tạo test mới |
| "chỉ chạy nhanh cho biết kết quả" | thêm `--no-browser` vào `/autotest` hoặc `/retest` — chỉ chạy ngầm (headless) |
| "case này test tay xong rồi" | **`/capture-manual`** — nhánh C: case tay → auto test |
| "test theo case anh/khách đưa" | **`/from-cases`** — nhánh: bám test case user |
| "khóa behavior code cũ / refactor an toàn" | **`/characterize <path>`** — nhánh B |
| "kiểm tra chất lượng test trước merge" | **`/verify-tests`** |
| "đã test đến đâu / còn gì chưa test" | **`/coverage`** (chỉ xem) — board tiến độ `.context/coverage.json` **tự cập nhật** sau khi browser test xong |
| "test fail vì sao" | test-reflector phân loại → sửa test hoặc báo loop agent |
| review/check | reviewer (nếu dự án có) |

## Phân biệt nhanh `/autotest` vs `/retest`

| | `/autotest` | `/retest` |
|---|---|---|
| Mục đích | Test tính năng **MỚI** (spec mới) | **Chạy lại** test đã có |
| Sinh test case? | ✅ **CÓ** (từ spec mới) | ❌ KHÔNG |
| Phạm vi | phần mới (theo spec/test-scope) | `--full` hoặc 1 chức năng |
| Dùng khi | spec vừa đổi / feature mới | muốn xác nhận còn xanh |

## Mặc định khi session bắt đầu
1. `/spec-link --sync` — pull spec mới nhất về `.spec-cache/` (nếu chưa link → hỏi link)
2. Đọc `.spec-cache/SPECIFICATIONS.md` — nguồn truth
3. Đọc `.spec-cache/spec/test-scope/current.json` — scope mới từ dev; đối chiếu board `.context/coverage.json` (req nào chưa test)
4. Chạy test suite hiện tại xem pass không (`npm test` / `pytest`)
5. Check `.context/manual-cases/` — case manual chưa chuyển auto

## Quy tắc nền
- ⭐ **2 lệnh độc lập**: `/autotest` (tạo test case mới từ spec + chạy phần mới) và `/retest` (chạy lại test đã có — full hoặc 1 chức năng). Đừng retest tràn lan mỗi lần test feature mới.
- ⭐ Cả 2 lệnh dùng **cùng cơ chế chạy**: (1) chạy ngầm headless trước cho nhanh → (2) autotest browser **headed** (bung hẳn ra) cho user theo dõi → cuối cùng **lưu kết quả + GIỮ browser mở**; (3) **cập nhật tiến độ chỉ sau khi browser xong** — chạy ngầm chỉ lưu log
- Spec = link git tới `.spec-cache/` (KHÔNG lưu bản riêng — tránh lệch)
- Mọi test → traceability về spec (`test-validator`) hoặc case user/manual
- Test-scope là hợp đồng từ dev; thiếu → hỏi, không tự đoán rộng
- ⭐ **1 bộ test hoàn chỉnh**: mọi test đều bổ sung/cập nhật vào cùng bộ (`tests/` + `test-registry.json`) — không tạo suite song song; cuối luồng **tự động** đăng ký + **cập nhật tiến độ test** (kiểm tra trùng trước), không cần command
- **Spec chỉ đọc từ link** (`.spec-cache/`, read-only); test chỉ giữ **tiến độ test** — không tạo/sinh/sửa spec
- Mọi fail → test-reflector phân loại → sửa đúng chỗ
- Không merge nếu chưa `/verify-tests` pass
