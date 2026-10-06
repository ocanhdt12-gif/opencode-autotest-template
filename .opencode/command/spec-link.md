# /spec-link — Config spec: nhập link git folder spec (bước đầu khi bắt đầu test)

Template TEST **không lưu spec** — trỏ tới folder spec trong repo DEV (1 nguồn duy nhất, không lệch). Là bước **config spec** đầu tiên của `/autotest` (chưa link → `/autotest` sẽ hỏi).

## Cách dùng
```
/spec-link <git-url>              → link repo dev (branch main) — CHỈ LẦN ĐẦU
/spec-link <git-url> --branch x   → branch khác
/spec-link --sync                 → pull spec mới nhất (thủ công; bình thường TỰ ĐỘNG khi chạy /autotest hoặc /retest)
/spec-link --status               → xem link hiện tại + spec version
```

## Flow (link lần đầu)
1. Ghi `gitUrl`/`branch` vào `spec-source.json`
2. Clone **shallow** vào `.spec-cache/` (gitignored):
   ```bash
   git clone --depth 1 --branch <branch> <gitUrl> .spec-cache
   # repo lớn → optional: --filter=blob:none --sparse + sparse-checkout set spec .spec-cache/SPECIFICATIONS.md
   ```
3. Verify `.spec-cache/SPECIFICATIONS.md` + `.spec-cache/spec/test-scope/current.json` tồn tại
4. Đọc `spec_version` → lưu `lastSyncedSpecVersion`; báo user

## Flow (sync)
> ⚙️ Sync **TỰ ĐỘNG** khi bắt đầu `/autotest` hoặc `/retest` — user không cần gõ tay. Dưới đây là việc hệ thống tự làm:
1. `git -C .spec-cache pull --ff-only`
2. Đọc `spec_version` mới → so `.context/test-status.json` → còn phần chưa cover → gợi ý chạy `/autotest` (tạo test case cho phần mới)

## Rule
- ⚙️ **Sync TỰ ĐỘNG** đầu mỗi lần chạy test — `/spec-link --sync` chỉ dùng thủ công khi cần sync riêng
- KHÔNG copy spec vào repo test (tránh lệch) — chỉ cache tạm
- `.spec-cache/` read-only — không sửa gì trong đó
- Chưa link → hỏi link, không tự bịa spec
- Mất mạng → dùng cache cũ + cảnh báo "spec có thể cũ"