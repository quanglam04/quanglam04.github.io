---
title: "Git Branching Deep Dive"
date: 2026-06-22 21:38:00 +0700
categories: [git, tool]
tags: [git, tool, ops]
---

# Git Branching Deep Dive: Branch, Tag, HEAD và Stash

> **Đối tượng đọc:** Developer, DevOps Engineer, SRE muốn hiểu Git từ bên trong thay vì chỉ học lệnh thuộc lòng.

---

## Mục lục

1. [Git Branch thực chất là gì?](#1-git-branch-thực-chất-là-gì)
2. [Branch mặc định: `master` / `main`](#2-branch-mặc-định-master--main)
3. [Tạo và chuyển Branch](#3-tạo-và-chuyển-branch)
4. [Commit trên Branch — Cơ chế song song](#4-commit-trên-branch--cơ-chế-song-song)
5. [Git Tag — Đánh dấu cột mốc bất biến](#5-git-tag--đánh-dấu-cột-mốc-bất-biến)
6. [Branch vs Tag — Điểm khác biệt cốt lõi](#6-branch-vs-tag--điểm-khác-biệt-cốt-lõi)
7. [HEAD là gì?](#7-head-là-gì)
8. [Attached HEAD vs Detached HEAD](#8-attached-head-vs-detached-head)
9. [Git Stash — Cứu cánh khi làm việc dở dang](#9-git-stash--cứu-cánh-khi-làm-việc-dở-dang)
10. [Tổng kết và Roadmap tiếp theo](#10-tổng-kết-và-roadmap-tiếp-theo)

---

## 1. Git Branch thực chất là gì?

Nhiều người mới học Git hình dung **branch** như một bản sao của toàn bộ source code — kiểu như "copy thư mục ra rồi làm việc riêng". Hiểu như vậy là sai, và chính sự hiểu nhầm này khiến người ta sợ tạo branch.

### Sự thật: Branch chỉ là một con trỏ (pointer)

Một Git branch về mặt kỹ thuật chỉ bao gồm hai thứ:

- **Tên** của branch (ví dụ: `main`, `feature-login`)
- **Một pointer** trỏ đến commit mới nhất của branch đó

Không hơn không kém.

Hãy xem ví dụ sau:

```
A → B → C
         ↑
       master
```

Branch `master` chỉ đơn giản là một file text trong `.git/refs/heads/master` chứa hash của commit `C`. Không có bất kỳ bản sao file nào được tạo ra.

Khi bạn tạo một commit mới `D`:

```
A → B → C → D
             ↑
           master
```

Git tự động cập nhật pointer của branch để trỏ sang `D`. Đây là lý do tại sao:

| Đặc điểm | Ý nghĩa thực tế |
|---|---|
| Tạo branch gần như tức thời | Chỉ tạo một file ~40 bytes |
| Không nhân bản source code | Không tốn disk space |
| Có thể tạo hàng trăm branch | Mà không ảnh hưởng hiệu năng |

### Tại sao điều này quan trọng trong DevOps?

Git được thiết kế nhẹ từ đầu, phù hợp với các workflow hiện đại:

- **Feature Branch Workflow** — Mỗi tính năng là một branch riêng biệt
- **GitFlow** — Phân tách rõ `develop`, `release`, `hotfix`
- **Trunk-Based Development** — Branch ngắn, merge nhanh, CI/CD liên tục
- **CI/CD Pipeline** — Mỗi PR/branch kích hoạt một pipeline độc lập

---

## 2. Branch mặc định: `master` / `main`

Khi bạn chạy:

```bash
git init
```

Git tạo ra một branch đầu tiên ngay lập tức. Trước đây tên mặc định là `master`, ngày nay hầu hết repository — đặc biệt trên GitHub — đã chuyển sang dùng `main`.

Branch này được tạo sẵn để bạn có thể bắt đầu commit ngay:

```bash
git add .
git commit -m "Initial commit"
```

Từ thời điểm đó, chuỗi commit của dự án bắt đầu hình thành.

> **Tip:** Nếu muốn đổi tên branch mặc định khi `git init`:
> ```bash
> git init --initial-branch=main
> # hoặc cấu hình toàn cục
> git config --global init.defaultBranch main
> ```

---

## 3. Tạo và chuyển Branch

### Tạo branch mới

```bash
git branch feature-login
```

Lệnh này chỉ tạo thêm **một pointer mới**, không thêm, không bớt bất kỳ file nào:

```
A → B → C
         ↑
       master
         ↑
   feature-login
```

Cả hai branch đều trỏ đến cùng một commit `C`. Dung lượng tăng thêm? Gần như bằng không.

### Chuyển sang branch

Để bắt đầu làm việc trên branch mới, bạn cần **checkout** (hoặc dùng cú pháp mới hơn `switch`):

```bash
# Cách cũ (vẫn hoạt động)
git checkout feature-login

# Cách mới (Git 2.23+, rõ ràng hơn)
git switch feature-login
```

### Tạo và chuyển cùng lúc

Trong thực tế, 99% trường hợp bạn muốn tạo xong là dùng luôn:

```bash
# Cách cũ
git checkout -b feature-login

# Cách mới
git switch -c feature-login
```

---

## 4. Commit trên Branch — Cơ chế song song

Đây là phần nhiều magic của Git nằm ở đây.

Giả sử bạn đang trên `feature-login` và tạo một commit mới:

**Trước khi commit:**
```
A → B → C
         ↑
       master
         ↑
   feature-login  ← HEAD đang ở đây
```

**Sau khi commit:**
```
A → B → C → D
     ↑       ↑
   master   feature-login ← HEAD
```

Nhận xét quan trọng:

- Chỉ `feature-login` di chuyển sang `D`
- `master` vẫn đứng yên ở `C`
- **Hai branch không hề ảnh hưởng lẫn nhau**

Đây chính là nền tảng cho phép nhiều developer làm việc song song trên cùng một repository mà không gây xung đột ngay lập tức.

### Ví dụ thực tế: Team 3 người

```
              ← feature-auth (Người A)
             /
A → B → C ← main
             \
              ← feature-payment (Người B)
               \
                ← bugfix-dashboard (Người C)
```

Mỗi người làm việc độc lập, và chỉ khi merge mới cần giải quyết conflict (nếu có).

---

## 5. Git Tag — Đánh dấu cột mốc bất biến

Branch di chuyển theo từng commit. Đôi khi bạn cần đánh dấu một commit cụ thể và **không bao giờ thay đổi** — đó là lúc dùng **Tag**.

### Tạo Tag

```bash
# Lightweight tag (chỉ là pointer, không có metadata)
git tag v1.0

# Annotated tag (có author, date, message — recommended)
git tag -a v1.0 -m "Release version 1.0"

# Tag một commit cũ (theo hash)
git tag -a v0.9 9fceb02 -m "Hotfix release"
```

### Xem và chia sẻ Tag

```bash
# Liệt kê tất cả tag
git tag

# Xem chi tiết một tag
git show v1.0

# Push tag lên remote (tag KHÔNG tự động push)
git push origin v1.0

# Push tất cả tag
git push origin --tags
```

### Tại sao Tag quan trọng trong DevOps?

Hãy xem một CI/CD pipeline điển hình:

```
Git Commit
    ↓
  Build
    ↓
  Test
    ↓
  Tag  ←── git tag -a build-203 -m "Build 203 passed"
    ↓
 Deploy
    ↓
 Release
```

Khi tester phát hiện lỗi ở **Build #202**:

```bash
# Tái tạo chính xác môi trường lúc build lỗi
git checkout build-202

# Hoặc checkout theo tag version
git checkout v1.2.3
```

Không có tag, bạn sẽ phải nhớ (hoặc tìm kiếm) hash của commit tương ứng — tốn thời gian và dễ nhầm.

---

## 6. Branch vs Tag — Điểm khác biệt cốt lõi

| | **Branch** | **Tag** |
|---|---|---|
| **Bản chất** | Pointer di động | Pointer cố định |
| **Thay đổi sau commit?** | ✅ Có, luôn trỏ commit mới nhất | ❌ Không, đứng yên mãi mãi |
| **Mục đích** | Phát triển tính năng, sửa lỗi | Đánh dấu release, milestone |
| **Ví dụ tên** | `feature/login`, `hotfix/payment` | `v1.0`, `v2.3.1`, `build-203` |
| **Có thể xóa?** | Nên xóa sau khi merge | Nên giữ lại vĩnh viễn |
| **Tương tự trong thực tế** | Nhánh sông đang chảy | Cột mốc lịch sử trên bản đồ |

```
# Branch: luôn di chuyển
master → C → D → E → F → ...

# Tag: đứng yên mãi mãi
v1.0 → C  (mãi mãi là C, không bao giờ thay đổi)
```

---

## 7. HEAD là gì?

`HEAD` là một **pointer đặc biệt** trong Git. Nếu branch là dấu trang trong cuốn sách, thì `HEAD` là ngón tay bạn đang đặt lên trang hiện tại.

`HEAD` cho Git biết:
- **Working Directory** hiện tại phải chứa những file nào
- Commit tiếp theo sẽ được đặt ở đâu trong lịch sử
- Branch nào đang được theo dõi

### HEAD được lưu ở đâu?

```bash
cat .git/HEAD
# ref: refs/heads/main
```

Thông thường, `HEAD` chứa **tên của branch** mà bạn đang đứng, không phải hash của commit.

---

## 8. Attached HEAD vs Detached HEAD

### Trạng thái bình thường: Attached HEAD

```
HEAD
 ↓
feature-login
 ↓
A → B → C
```

HEAD không trỏ thẳng vào commit — nó trỏ vào **branch**, và branch trỏ vào commit. Khi có commit mới:

```
A → B → C → D
             ↑
       feature-login
             ↑
            HEAD
```

Cả `HEAD` lẫn branch cùng tiến về phía trước. Đây là trạng thái mà bạn sẽ làm việc 95% thời gian.

---

### Trạng thái đặc biệt: Detached HEAD

Đôi khi bạn muốn "nhảy về quá khứ" để xem một commit cũ:

```bash
git checkout a1b2c3
# hoặc checkout theo tag
git checkout v1.0
# hoặc checkout theo hash ngắn
git checkout 9fceb
```

Lúc này, `HEAD` không còn trỏ vào branch nữa — nó trỏ **thẳng vào commit**:

```
A → B → C
         ↑
        HEAD  (không còn gắn với branch nào)
```

```bash
$ cat .git/HEAD
# a1b2c3d4e5f6...  (hash trực tiếp, không phải tên branch)
```

Git cũng sẽ cảnh báo bạn:

```
You are in 'detached HEAD' state. You can look around, make experimental
changes and commit them, and you can discard any commits you make in this
state without impacting any branches by switching back to a branch.
```

### Khi nào dùng Detached HEAD?

| Use case | Lệnh |
|---|---|
| Debug lỗi xuất hiện từ version cũ | `git checkout v1.2.3` |
| Tái tạo môi trường khi build fail | `git checkout build-202` |
| Xem file ở thời điểm cụ thể | `git checkout 9fceb02` |
| CI/CD checkout theo commit hash | `git checkout $GIT_COMMIT` |

CI/CD pipeline thực tế **thường xuyên** chạy trong Detached HEAD vì chúng checkout theo commit hash hoặc tag, không theo tên branch.

### ⚠️ Rủi ro của Detached HEAD

Nếu bạn tạo commit trong trạng thái Detached HEAD:

```
A → B → C
          \
           D  ← HEAD (orphan commit, không thuộc branch nào)
```

Khi bạn chuyển sang branch khác:

```bash
git checkout main
```

Commit `D` trở thành **orphan commit** — không branch nào biết đến sự tồn tại của nó. Sau một thời gian, Git garbage collector sẽ xóa nó đi mà không báo trước.

### Cách thoát Detached HEAD an toàn

**Tạo branch mới để "cứu" các commit đã tạo:**

```bash
git switch -c hotfix-from-old-version
# hoặc
git checkout -b hotfix-from-old-version
```

Lúc này:

```
A → B → C → D
             ↑
    hotfix-from-old-version
             ↑
            HEAD  (attached trở lại)
```

**Hoặc đơn giản quay về branch cũ (nếu chưa commit gì):**

```bash
git checkout main
# hoặc
git switch main
```

---

## 9. Git Stash — Cứu cánh khi làm việc dở dang

### Bài toán quen thuộc

Bạn đang code dở `feature/payment-gateway`, tự nhiên Production Alert nổ lên. Bạn cần:

1. Chuyển sang `hotfix/critical-bug`
2. Fix lỗi
3. Deploy
4. Quay về tiếp tục feature

Vấn đề: Working Directory đầy file đang chỉnh sửa. Git **không cho phép** chuyển branch khi có:

- Modified files chưa commit
- Staged files chưa commit

```bash
$ git switch hotfix/critical-bug
error: Your local changes to the following files would be overwritten by checkout:
    src/payment/gateway.py
Please commit your changes or stash them before you switch branches.
```

### Git Stash: Snapshot tạm thời

Stash hoạt động như một "ngăn kéo tạm" — bạn cất công việc vào đó, làm việc khác, rồi mở ra tiếp tục.

```bash
# Lưu toàn bộ công việc đang làm
git stash push -m "WIP: Payment gateway integration"
```

**Trước khi stash:**
```bash
$ git status -s
M  src/payment/gateway.py
?? src/payment/webhook.py
```

**Sau khi stash:**
```bash
$ git status
On branch feature/payment-gateway
nothing to commit, working tree clean
```

Working Directory sạch hoàn toàn. Bạn có thể chuyển branch tự do.

### Quy trình đầy đủ

```bash
# 1. Cất công việc vào stash
git stash push -m "WIP: Payment gateway"

# 2. Chuyển sang hotfix branch
git switch hotfix/critical-bug

# 3. Fix bug, commit, merge, deploy...
git commit -m "fix: resolve null pointer in auth middleware"
git switch main && git merge hotfix/critical-bug

# 4. Quay lại feature branch
git switch feature/payment-gateway

# 5. Lấy lại công việc từ stash
git stash pop
```

### Các lệnh Stash quan trọng

```bash
# Xem danh sách tất cả stash
git stash list
# stash@{0}: On feature/payment: WIP: Payment gateway
# stash@{1}: On feature/auth: WIP: JWT refresh token

# Lưu stash (có message để dễ nhận biết)
git stash push -m "Terraform module for RDS"

# Lấy ra stash mới nhất VÀ xóa nó khỏi danh sách
git stash pop

# Lấy ra stash mới nhất NHƯNG GIỮ LẠI trong danh sách
git stash apply

# Lấy ra một stash cụ thể theo index
git stash apply stash@{2}

# Xóa một stash cụ thể
git stash drop stash@{1}

# Xóa toàn bộ stash (cẩn thận!)
git stash clear
```

### Stash với untracked files

Mặc định, stash **không** lưu file chưa được `git add` (untracked files):

```bash
# Stash cả untracked files
git stash push -u -m "Include new files"

# Stash cả untracked VÀ ignored files
git stash push -a -m "Include everything"
```

### Tại sao DevOps Engineer dùng Stash rất nhiều?

Trong môi trường DevOps, bạn thường xuyên làm việc với nhiều loại file cùng lúc:

```
Đang viết:         Bị interrupt bởi:
─────────────      ─────────────────────
Terraform module   Production incident
Ansible playbook   Security patch urgent
Jenkinsfile        Customer-reported bug
K8s manifest       On-call alert
```

Thay vì phải:
- Tạo branch tạm (`temp/wip-dont-merge`)
- Commit code chưa hoàn thiện với message "WIP" xấu xí
- Push lên remote (tiềm năng trigger CI/CD sai)

Bạn chỉ cần:

```bash
git stash push -m "K8s HPA config WIP - pausing for incident"
git switch hotfix/prod-incident
# ... xử lý incident ...
git switch feature/k8s-autoscaling
git stash pop
# Tiếp tục đúng nơi đã dừng
```

---

## 10. Tổng kết và Roadmap tiếp theo

### Bảng tổng hợp các khái niệm cốt lõi

| Khái niệm | Bản chất | Đặc điểm chính |
|---|---|---|
| **Branch** | Pointer di động | Luôn trỏ commit mới nhất sau mỗi commit |
| **Tag** | Pointer cố định | Không bao giờ thay đổi sau khi tạo |
| **HEAD** | Con trỏ vị trí hiện tại | Cho Git biết bạn đang "đứng" ở đâu |
| **Attached HEAD** | HEAD → Branch → Commit | Trạng thái bình thường khi làm việc |
| **Detached HEAD** | HEAD → Commit trực tiếp | Dùng khi checkout commit/tag cũ |
| **Stash** | Snapshot tạm thời | Lưu/khôi phục công việc chưa hoàn thành |

### Sơ đồ quan hệ tổng thể

```
Repository
│
├── Commits: A → B → C → D → E
│                    ↑       ↑
│                  v1.0    main (branch)
│                           ↑
│                          HEAD (attached)
│
└── Stash Stack
    ├── stash@{0}: "WIP: Feature X"
    └── stash@{1}: "Hotfix investigation"
```

### Roadmap tiếp theo

Nắm chắc các khái niệm trong bài này là điều kiện tiên quyết để học:

- **Merge vs Rebase** — Hai chiến lược tích hợp branch với đặc điểm hoàn toàn khác nhau
- **GitFlow** — Mô hình quản lý branch cho dự án có release cycle cố định
- **Trunk-Based Development** — Phương pháp CI/CD-first với branch tồn tại ngắn
- **Pull Request / Merge Request** — Code review workflow trên GitHub/GitLab
- **GitOps** — Dùng Git làm source of truth cho infrastructure
- **Release Management** — Kết hợp Tag, Branch và CI/CD để quản lý release
- **Troubleshooting Production** — Dùng `git bisect`, Detached HEAD và Tag để tìm nguyên nhân lỗi

---
