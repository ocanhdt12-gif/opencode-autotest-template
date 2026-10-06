# FEATURE_WORKFLOW — Autotest Template (test-first + characterization + manual capture)

> ⚠️ **Maintenance-mode override:** state dùng `features[]`/`bugs[]`; **cấm push thẳng `forbidden_branch`** (mặc định `main`); branch/push model: staging-direct (cấu hình trong `PROJECT_PROFILE.md`).

## Luồng chạy test DUY NHẤT — `/autotest`

Mọi lần test đều đi qua **1 luồng duy nhất** `/autotest`, gồm 3 giai đoạn:
1. **Chạy NGẦM (headless)** — chạy suite như bình thường, **không bung browser**, cho nhanh → **chỉ lưu log** `.context/test-results/headless-run.json`
2. **Autotest BROWSER (headed)** — **bung browser thật** cho user theo dõi thao tác; cuối cùng **lưu kết quả** + **GIỮ browser mở** để lại màn hình kết quả
3. **Cập nhật tiến độ (tự động, chỉ sau bước 2)** — registry + coverage + test-status

Chi tiết: `.opencode/command/autotest.md` + `docs/FLOWS.md`.

## Workflow chính (test-first — mỗi feature/task)

1. **Spec**: đọc `.spec-cache/SPECIFICATIONS.md` → xác định requirement `R-xx` liên quan → spec-validator đảm bảo spec PASS
2. **Test trước**: test-writer viết test từ spec (unit + property-based), traceable `R-xx`
3. **Verify ĐỎ**: chạy test → phải fail đúng cách (chưa có code) — không red = test sai
4. **Code**: implement tới khi test XANH (có thể dùng template dev để code)
5. **Verify chất lượng** (`/verify-tests`): mutation testing + quality gate + hidden stash → PASS mới merge
6. **Chạy `/autotest`**: ngầm (headless) → browser (headed, giữ mở) → cập nhật tiến độ
7. **Error flow**: test fail → test-reflector phân loại (bug-test / bug-code) → sửa đúng chỗ

## Workflow characterization (legacy chưa test)

1. `/characterize <path>` → characterization-writer: snapshot + scrub unstable + coverage bổ sung
2. **Mutation verify** bắt buộc (phá code → test fail) — test vô nghĩa nếu không bắt
3. Refactor/sửa an toàn trên nền golden — đổi behavior chủ đích phải xác nhận

## Workflow manual capture

1. Case test tay xong → mô tả vào `.context/manual-cases/<id>.md`
2. `/capture-manual <id>` → viết test auto → XANH → đánh dấu `converted-to-auto`
3. Không tự động hóa được → `manual-only` + lý do (không ép)

## Gates (bắt buộc trước merge)

| Gate | Khi nào | Fail khi |
|---|---|---|
| spec-validator | trước khi sinh test | requirement thiếu/mâu thuẫn |
| test ĐỎ đúng cách | trước khi code | test xanh khi chưa code |
| mutation verify | nhánh B | test không bắt mutation |
| mutation score ≥ floor | `/verify-tests` | survivors không giải thích được |
| quality gate | `/verify-tests` | expected từ code, try-catch nuốt, assert trivially |
| hidden stash | `/verify-tests` + CI | test ẩn fail |
| test traceability | test-validator | requirement R-xx không có test |
| browser headed + giữ mở | `/autotest` giai đoạn 2 | chạy headless / tự đóng browser |

## Lỗi test (xử lý nội bộ)

Test fail → test-reflector phân loại: `bug-in-test` (test sai → sửa test) · `bug-in-code` (test đúng code sai → báo loop agent fix theo spec). Có thể ghi chú ngắn vào `.context/test-notes.md` nếu đáng nhớ.
