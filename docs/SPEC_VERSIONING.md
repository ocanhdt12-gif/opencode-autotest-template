# Spec Versioning & Link (mô hình nguồn duy nhất)

> **Template TEST KHÔNG lưu spec.** Nó giữ 1 con trỏ git tới folder spec của repo DEV → luôn dùng đúng 1 nguồn, không bao giờ lệch.

## Vì sao không lưu spec ở template test

2 bản spec (dev + test) sẽ lệch nhau theo thời gian → test kiểm tra sai hợp đồng. Thay vào đó: test **link git** tới spec của dev, sync mỗi lần chạy → luôn khớp.

## Nguồn sự thật (thuộc repo DEV)

```
DEV repo/
├── SPECIFICATIONS.md            ← spec canonical (frontmatter spec_version)
└── spec/
    ├── CHANGELOG.md             ← lịch sử version spec
    ├── updates/YYYY-MM-DD-slug.md  ← delta mỗi lần đổi
    ├── archive/SPECIFICATIONS-<version>.md
    └── test-scope/current.json  ← hợp đồng bàn giao (specVersion + scopeVersion)
```

## Phía TEMPLATE TEST: chỉ giữ link

`spec-source.json` (root repo test):
```jsonc
{
  "gitUrl": "git@github.com:org/dev-repo.git",
  "branch": "main",
  "specPath": "spec",
  "specFile": "SPECIFICATIONS.md",
  "scopeFile": ".spec-cache/spec/test-scope/current.json",
  "cacheDir": ".spec-cache",
  "lastSyncedAt": null,
  "lastSyncedSpecVersion": null
}
```

### `/spec-link <git-url>` (bắt đầu dự án — anh nhập link vào đây)
1. Ghi `gitUrl`/`branch` vào `spec-source.json`
2. Clone **shallow + sparse** (chỉ folder spec):
   ```bash
   git clone --depth 1 --filter=blob:none --sparse <gitUrl> .spec-cache
   cd .spec-cache && git sparse-checkout set spec
   ```
3. Verify `.spec-cache/SPECIFICATIONS.md` + `.spec-cache/spec/test-scope/current.json`
4. Lưu `lastSyncedSpecVersion` = `spec_version`; báo user

### `/spec-link --sync` (mỗi lần chạy test)
- `git -C .spec-cache pull --ff-only` → đọc version mới → so `.context/test-status.json` → còn phần chưa cover → gợi ý test

## Vị trí đọc spec (template test)

| Cần gì | Đọc ở đâu |
|---|---|
| Requirements R-xx | `.spec-cache/SPECIFICATIONS.md` |
| Scope mới nhất | `.spec-cache/spec/test-scope/current.json` |
| Lịch sử version | `.spec-cache/spec/CHANGELOG.md` · `.spec-cache/spec/updates/` |

## Test biết "cần test đến đâu"

`.context/test-status.json` (do template TEST ghi):
```jsonc
{
  "specVersionCovered": "1.2.0",
  "scopeVersionCovered": 3,
  "lastRun": "2026-10-05T14:00:00+07:00",
  "pendingSpecVersion": null
}
```
**Quy tắc:** `.spec-cache` spec_version > `specVersionCovered` → còn phần mới chưa cover → chạy `/test-scope` hoặc `/autotest --full`.

## Version scheme (spec, thuộc dev)

| Loại | Khi nào | Ví dụ |
|---|---|---|
| MAJOR | Xoá/đổi ngữ nghĩa requirement | 1.4.0 → 2.0.0 |
| MINOR | Thêm requirement | 1.4.0 → 1.5.0 |
| PATCH | Sửa wording | 1.4.0 → 1.4.1 |

## Ai ghi gì

| File | Ai ghi | Ở đâu |
|---|---|---|
| `SPECIFICATIONS.md` + `spec/*` | **DEV** | repo DEV (test chỉ đọc qua cache) |
| `.spec-cache/spec/test-scope/current.json` | **DEV** | repo DEV |
| `spec-source.json` | **TEST** (1 lần) | repo TEST |
| `.context/test-status.json` | **TEST** | repo TEST |

## Rules
- `.spec-cache/` là **read-only** với template test — KHÔNG sửa gì trong đó
- KHÔNG copy spec vào repo test — chỉ cache tạm (gitignored)
- Mất mạng → dùng cache cũ + cảnh báo "spec có thể cũ"
- Đổi `gitUrl` → xoá cache cũ, clone lại sạch