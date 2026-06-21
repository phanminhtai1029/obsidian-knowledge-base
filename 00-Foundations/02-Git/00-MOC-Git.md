---
title: "MOC: Version Control - Git"
section: 00-Foundations/02-Git
tags: [moc, git, github, ci-cd, version-control, foundations, fresher]
module: L1_AL_NLS_VCG
---

# MOC: Version Control - Git

> Module `L1_AL_NLS_VCG`. Thi: Final Theory Test.
> Học **từ cơ bản → nâng cao**: nền tảng Git → "superpowers" (stash, rebase lịch sử, undo/revert) → **CI/CD & GitHub Actions** → cheatsheet tra nhanh.
> Mỗi note có bảng lệnh, mục *khi nào dùng / bẫy thường gặp*, và khối `★ Insight`.

## Phần A — Cơ bản (nền tảng bắt buộc)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 1 | [[01-How-Git-Works\|How Git works]] | Mô hình snapshot, 3 vùng working/staging/repo, HEAD, commit | ✅ |
| 2 | [[02-Install-Configure-Git\|Install & Configure Git]] | Cài đặt, `git config`, SSH key, kết nối GitHub | ✅ |
| 3 | [[03-Daily-Workflow\|Workflow hằng ngày]] | clone/status/add/commit/diff/log/push/pull/fetch | ✅ |
| 4 | [[04-Gitignore-va-Khai-niem-cot-loi\|.gitignore & khái niệm cốt lõi]] | `.git`, `.gitignore`, Conventional Commits | ✅ |
| 5 | [[05-Branch-Merge-PR\|Branch · Merge · Pull Request]] | branch/switch, merge, PR, giải quyết conflict | ✅ |

## Phần B — Nâng cao (Git superpowers)

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 6 | [[06-Git-Stash\|git stash]] | Cất code tạm: stash/pop/apply/list/save | ✅ |
| 7 | [[07-Staging-chon-loc\|Staging chọn lọc]] | 3 cách add, `add -p`, partial add | ✅ |
| 8 | [[08-Push-tuong-minh\|Push tường minh]] | `push origin <branch>`, `--set-upstream` | ✅ |
| 9 | [[09-Hoan-tac-va-sua-lich-su\|Hoàn tác & sửa lịch sử]] | `reset`, `amend`, `revert` — khi nào dùng gì | ✅ |
| 10 | [[10-Git-Log-nang-cao\|git log nâng cao]] | `--oneline`, `--graph`, dò bug | ✅ |
| 11 | [[11-Git-Hooks-va-meo-GitHub\|Git Hooks & mẹo GitHub]] | pre-commit hooks, fork, phím `.`, Codespaces | ✅ |

## Phần C — CI/CD & Automation

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 12 | [[12-CI-CD-la-gi\|CI/CD là gì]] | CI vs Continuous Delivery vs Deployment, pipeline | ✅ |
| 13 | [[13-GitHub-Actions\|GitHub Actions]] | Workflow YAML, jobs/steps/runners, secrets, ví dụ pipeline | ✅ |

## Phần D — Tra cứu

| # | Note | Nội dung | Trạng thái |
|---|------|----------|------------|
| 14 | [[14-Git-Cheatsheet\|⭐ Cheatsheet lệnh Git]] | Bảng tra nhanh toàn bộ lệnh theo tình huống | ✅ |

## Liên quan
- [[../00-MOC-Foundations|MOC: Foundations]]
- [[../03-Agile-Scrum/00-MOC-Agile-Scrum|MOC: Agile-Scrum]]
