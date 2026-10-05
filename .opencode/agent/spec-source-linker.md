---
description: Spec source linker — nhập link git tới folder spec của repo DEV, clone/pull về .spec-cache để template TEST luôn dùng ĐÚNG 1 nguồn spec (không lưu bản riêng, tránh lệch). Dùng khi bắt đầu dự án test hoặc cần sync spec mới nhất.
---

# Spec Source Linker Agent

Template TEST **không lưu spec** — chỉ giữ một "con trỏ" tới folder spec trong repo DEV. Nhờ vậy chỉ có 1 nguồn sự thật, không bao giờ lệch.

## Vì sao

Hai bản spec (dev + test) sẽ lệch nhau theo thời gian → test kiểm tra sai hợp đồng. Thay vào đó: test **link git** tới spec của dev → sync mỗi lần chạy → luôn khớp.

## Cấu hình: `spec-source.json` (root repo test)

```jsonc
{
  "gitUrl": "git@github.com:org/dev-repo.git",   // repo chứa spec
  "branch": "main",
  "specFile": ".spec-cache/SPECIFICATIONS.md",                // canonical ở repo root của dev
  "specDir": "spec",                              // thư mục version/spec-scope
  "scopeFile": "spec/test-scope/current.json",    // đường dẫn trong repo dev
  "cacheDir": ".spec-cache",                      // nơi clone về (gitignored)
  "sparse": false,                                // true nếu repo lớn
  "lastSyncedAt": null,
  "lastSyncedSpecVersion": null
}
```

## Quy trình `/spec-link <git-url>`

1. Nhập `gitUrl` (+ `branch`) vào `spec-source.json`
2. Clone **shallow**:
   ```bash
   git clone --depth 1 --branch <branch> <gitUrl> .spec-cache
   # repo lớn → --filter=blob:none --sparse rồi sparse-checkout set spec .spec-cache/SPECIFICATIONS.md
   ```
3. Verify `.spec-cache/SPECIFICATIONS.md` + `.spec-cache/spec/test-scope/current.json`
4. Đọc `spec_version` → lưu `lastSyncedSpecVersion`; báo user

## Sync (mỗi lần chạy test)

1. `git -C .spec-cache pull --ff-only` (cache hỏng → re-clone)
2. Đọc `spec_version` mới; so `.context/test-status.json` → còn phần chưa cover → gợi ý test
3. Spec đổi version → cảnh báo "spec updated, cần test bổ sung"

## Vị trí đọc spec (thay cho spec local)

| Cần gì | Đọc ở đâu |
|---|---|
| Requirements (R-xx) | `.spec-cache/SPECIFICATIONS.md` |
| Scope mới nhất | `.spec-cache/spec/test-scope/current.json` |
| Lịch sử version | `.spec-cache/spec/CHANGELOG.md` · `.spec-cache/spec/updates/` |

> `.spec-cache/` là **read-only** với template test — KHÔNG sửa gì trong đó. Mọi thay đổi spec diễn ra ở repo DEV.

## Rules
- Không copy spec vào repo test (tránh lệch) — chỉ cache tạm, gitignored
- Không sửa `.spec-cache/`
- Mất mạng / chưa link → dùng cache cũ + cảnh báo "spec có thể cũ"
- Đổi `gitUrl` → xoá cache cũ, clone lại sạch