---
title: "Git flow trong 1 dự án phần mềm thực tế"
date: 2025-07-07 21:38:00  +0700
categories: [Tools, Git, Workflow]
tags: [git, branching, workflow]
---

---

# Git flow trong 1 dự án phần mềm thực tế

<p align="center">
  <img src="/assets/images/git/git-flow.png" alt="Image title_1" />
</p>

## 1. Cấu trúc Branch

```
main (production)
├── develop (integration)
├── develop_xxx (xxx là issue gitlab)
├── test_xxx_vy (VD test_login_v1)
├── hotfix_xxx (xxx có thể issue gitlab hoặc description)
└── release/v1.2.0
```

## 2. Workflow Chi tiết

**Bước 1: Khởi tạo Repository**

```
# Clone repository
    git clone <repository-url>
    cd <project-name>

# Tạo branch develop từ main
    git checkout -b develop
    git push -u origin develop

#### Tạo Issue Branch từ develop

# Từ Develop branch

    git checkout develop
    git pull origin develop

# Tạo issue branch với số issue Gitlab
    git checkout -b develop_123 # Cho issue #123
    git checkout -b develop_456 # Cho issue #456
    git checkout -b develop_789 # Cho issue #789
```

**Bước 2: Development & Commit**

```
# Thực hiện development trên develop_123 branch
    git add .
    git commit -m "manage-task#123: Create UI C601"

# Push changes
    git push origin develop_123
```

**Bước 3: Code Review & Integration**

**Tạo Pull Request**

```
# Từ develop_123 về develop
# Tạo PR: develop_123 -> develop
```

**Merge về Develop**

```
    git checkout develop
    git pull origin develop

# Merge develop_123 vào develop
    git merge develop_123
    git push origin develop

# Xóa develop_123 branch sau khi merge
    git branch -d develop_123
    git push origin --delete develop_123
```

**Bước 4: Testing Flow (Sau khi merge vào Develop)**

**Tạo Test branch cho lần test mới**

```
# Sau khi có features mới trên develop, tạo test branch mới
    git checkout develop
    git pull origin develop

# Tạo test branch với tên screen/flow cần test và số thứ tự
    git checkout -b test_login_v1
    git push -u origin test_login_v1
```

**Bước 5: Release Flow**

**Tạo Release Branch từ Develop**

```
# Sau khi test pass (VD: test/3.0 pass), tạo release từ develop
    git checkout develop
    git pull origin develop

# Tạo release branch từ develop
    git checkout -b release/v1.2.0
    git push -u origin release/v.1.2.0
```

**Deploy to Production**

```
# Merge release vào main
    git  checkout main
    git pull origin main
    git merge release/v1.2.0

# Tạo tag
    git tag -a v1.2.0 -m "Release version 1.2.0"
    git push origin main --tags

# Xóa release branch ( giữ test branches để tracking)
    git branch -d release/v1.2.0
    git push origin --delete release/v1.2.0
```

**Bước 6: Hotfix flow (Khẩn cấp)**

```
# Từ main branch
    git checkout main
    git pull origin main

# Tạo hotfix branch với số issue gitlab
    git checkout -b hotfix_321  # Cho hotfix issue #321
    git push -u origin hotfix_321

# Fix bug
    git add .
    git commit -m "hotfix: resolve security vulnerability for issue manage-task#123"
    git push origin hotfix_321

# Merge vào main
    git checkout main
    git merge hotfix_321
    git tag -a v1.2.1 -m "Hotfix version 1.2.1"
    git push origin main --tags

# Merge vào develop
    git checkout develop
    git merge hotfix_321
    git push origin develop

# Xóa hotfix branch (giữ test branch để tracking)
    git branch -d hotfix_321
    git push origin --delete hotfix_321
```
