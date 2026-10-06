# AGENTS.md — Router (Autotest Template)

Entry point mọi session. Phân loại intent rồi route.

## 2 lệnh chính (xem `docs/FLOWS.md`)

| User nói | Route |
|---|---|
| "test tính năng mới" / "spec mới, tạo test case đi" / "bắt đầu test" | **`/autotest`** — lần đầu: hỏi config spec → overview → brainstorm → tạo test case → **chờ user chốt** → chạy ngầm → **chạy browser từng case** → cập nhật trạng thái. Lần sau: lấy task chưa test → tạo test case → chạy. |
| "retest lại toàn bộ" / "retest 1 cụm chức năng" / "retest case X" | **`/retest --all`** · **`/retest <module>`** · **`/retest <TC-xx>`** — chạy lại test đã có, KHÔNG tạo mới |
| "chỉ chạy nhanh cho biết kết quả" | thêm `--no-browser` vào `/autotest` hoặc `/retest` |
| "case này test tay xong rồi" | **`/capture-manual`** — nhánh: case tay → auto test |
| "test theo case anh/khách đưa" | **`/from-cases`** — nhánh: bám test case user |
| "khóa behavior code cũ / refactor an toàn" | **`/characterize <path>`** — nhánh B |
| "kiểm tra chất lượng test trước merge" | **`/verify-tests`** |
| "đã test đến đâu / còn gì chưa test" | **`/coverage`** (chỉ xem) — tiến độ tự cập nhật sau khi chạy |
| "test fail vì sao" | test-reflector phân loại → sửa test/chỉ báo dev |
| review/check | reviewer (nếu dự án có) |

## Phân biệt nhanh `/autotest` vs `/retest`

| | `/autotest` | `/retest` |
|---|---|---|
| Mục đích | Test tính năng **MỚI** | **Chạy lại** test đã có |
| Tạo test case? | ✅ CÓ (từ spec mới) | ❌ KHÔNG |
| Phạm vi | phần mới (spec/test-scope) | `--all` / 1 cụm chức năng / 1 test case |
| Dùng khi | spec mới / feature mới / bắt đầu test | xác nhận còn xanh |

## ⭐ Luật trục: TEST-CASE-FIRST (`skills/test-case-first`)

- **SPEC → TEST CASE (user check/sửa/chốt) → TEST CODE → chạy.** Mọi test phải follow test case.
- Test case gom theo **module** ở `.context/test-cases/<module>.md`, id `TC-<module>-NN`, `Status: draft→approved`.
- **User phải chốt test case trước khi chạy** (human checkpoint).
- Trạng thái task ở `.context/test-tasks.json` → **không test lại cái đã test**.

## Mặc định khi session bắt đầu
1. ⚙️ **TỰ ĐỘNG sync spec** — `git -C .spec-cache pull --ff-only` (không cần gõ `/spec-link --sync`; chỉ lần đầu chưa link mới hỏi `/spec-link <git-url>`)
2. Đọc `.spec-cache/SPECIFICATIONS.md` — nguồn truth
3. Đọc `.context/test-tasks.json` — case nào đã/chưa test
4. Chạy test suite hiện tại xem pass không (`npm test` / `pytest`)
5. Check `.context/manual-cases/` — case manual chưa chuyển auto

## Quy tắc nền
- ⭐ **TEST-CASE-FIRST**: test case do máy soạn (draft) → **user sửa & chốt** → mới sinh test code + chạy
- ⭐ **2 lệnh**: `/autotest` (tạo test case mới + chạy phần mới) · `/retest` (chạy lại đã có — all/cụm/case)
- ⭐ Cơ chế chạy chung: ngầm (headless) trước → browser (headed, bung hẳn, giữ mở) sau; browser đi **từng case**, xong chờ user chọn case tiếp; cập nhật trạng thái sau khi chạy
- ⚙️ **Sync spec tự động** trước mỗi lần test (không gõ tay `/spec-link --sync`)
- Spec = link git tới `.spec-cache/` (KHÔNG lưu bản riêng)
- Mọi test → traceability `R-xx` ↔ `TC-xx` ↔ test code
- Test-scope là hợp đồng từ dev; thiếu → hỏi, không tự đoán rộng
- ⭐ **1 bộ test hoàn chỉnh**: mọi test vào cùng bộ (`tests/` + `test-registry.json`) — không suite song song
- **Spec chỉ đọc từ link** (read-only); test chỉ giữ **tiến độ test**
- Mọi fail → test-reflector phân loại
- Không merge nếu chưa `/verify-tests` pass
