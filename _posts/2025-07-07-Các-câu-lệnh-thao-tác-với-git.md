---
title: "Một số câu lệnh thao tác với git cần phải biết"
date: 2025-07-07 21:38:00  +0700
categories: [Tools, Git]
tags: [git]
---

---

# Một số câu lệnh thao tác với git cần phải biết

<p align="center">
  <img src="/assets/images/git/architecture.png" alt="Image title_1" />
</p>

**1. Khởi tạo và clone Repository**

`git cclone <url>`

**Công dụng**: Tải một repo từ xa về máy tính cục bộ. Đây là bước đầu tiên khi muốn làm việc với một dụ án đã có sẵn

**2. Thêm và lưu thay đổi**

`git add .` hoặc `git add <tên file>`

**Công dụng**: Đưa các thay đổi từ thư mục làm việc (working directory) vào khu vực staging.

- `git add .`: Thêm tất cả các file
- `git add <tên file>`: Thêm một tệp cụ thể

`git commit -m"message"`

**Công dụng**: Lưu lại các thay đổi từ khu vực dàn dựng vào lịch sử của repo. Mỗi commit là 1 bản ghi về những thay đổi và thông điệp commit giúp mô tả sự thay đổi đó

> Message nên viết rõ ràng, ý nghĩa để dễ dàng theo dõi

`git commit --amend`

**Công dụng**: Sửa đổi commit gần nhất Dùng khi quen thêm một tệp, sai chính tả trong thông điệp commit, hoặc muốn gộp một vài thay đổi nhỏ vào commit trước đó

> _Lưu ý:_ Tránh sử dụng `--amend` trên các commit đã được push lên remote repository mà người khác đang làm việc cùng, vì nó sẽ thay đổi lịch sử commit

**3. Đồng bộ hóa với Remote repository**

`git push`

**Công dụng**: Đẩy các commit từ repository cục bộ lên remote repo (Github,Gitlab). Đây là cách chia sẻ công việc của mình với người khác hoặc sao lưu mã nguồn

- `git push -u origin <remote>`: Thiết lập liên kết giữa các nhánh cục bộ hiện tại và nhánh `<remote>` trên `origin`. Sau này chỉ cần dùng `git push`
- `git push origin <remote>`: Đẩy các thay đổi lên nhánh `<remote>` trên origin
- `git push --force`: Buộc git ghi đề lịch sử commit repo bằng lịch sử cục bộ

> Cảnh báo về `--force`: Nó có thể ghi đè công việc của người khác và làm mất lịch sử commit. Chỉ sử dụng khi thực sự hiểu rõ hậu quả và trong những trường hợp đặc biệt

`git pull`

**Công dụng**: Tải các thay đổi mới nhất từ remote repository và hợp nhất chúng vào nhánh cục bộ. Đây là cách cập nhật mà nguồn với những gì người khác đã đẩy lên

**4. Quản lý trạng thái và thay đổi tạm thời**

`git status`

**Công dụng**: Hiển thị trạng thái của thư mục làm việc và khu vực staging. Nó cho biết những tệp nào đã được sửa đổi. đã được thêm vào staging, và những tệp nào chưa được theo dõi

```
Ví dụ:
git status

Kết quả thường hiển thị

- On branch main
- Your branch is up to date with 'origin/main'
- Changes to be committed: (Các tệp đã git add)
- Changes not staged for commited: (Các tệp đã sửa nhưng chưa git add )
- Untracked files: (Các tệp mới nhưng chưa được theo dõi bởi git)
```

`git stash`

**Công dụng**: Tạm thời lưu trữ các thay đổi chưa được commit trong thư mục làm việc. Cho phép chuyển sang một nhánh hoặc làm việc khác mà không cần comit những thay đổi đang dở

`git stash list`

CÔng dụng: Liệt kê các stash đã được lưu trữ

`git stash apply @{}`

**Công dụng**: Áp dụng một stash cụ thể vào thư mục làm việc mà không xóa nó khỏi danh sách stash

`git stash pop`

**Công dụng**: Áp dụng stash gần nhất và đồng thời xóa nó khỏi danh sách stash

Ví dụ

```
git stash # Lưu các thay đổi hiện tại
git stash list # Xem danh sách các stash
git stash apply stash@{0} # Áp dụng stash đầu tiên
git stash pop # Áp dụng và xóa stash gần đây nhất

```

**5. Hợp nhất và quản lý lịch sử**

`git merge <branch>`

**Công dụng**: hợp nhất lịch sử của một nhánh vào nhánh hiện thị. Tạo ra 1 commit hợp nhất để ghi lại quá trình này

Ví dụ

```
git checkout main
git merge feature/new-design # Hợp nhất nhánh 'feature/new-design' vào 'main'
```

`git rebase <branch>`

**Công dụng**: Di chuyển hoặc kết hợp một chuỗi các commit mới vào 1 commit cơ sở khác. Thay vì tạo ra một merge commit, `rebase` "viết lại" lịch sử commit, làm cho nó trông tuyến tính hơn

> Cảnh báo: Không nên `rebase` các comit đã được đẩy lên remote và có người khác đang làm việc cùng, vì nó thay đổi lịch sử và có thể gây ra xung đột lớn. `rebase` thường được dùng để giữ lịch sử nhánh feature trước khi merge vào nhánh chính

Ví dụ:

```
git checkout feature/my-feature
git rebase main # Di chuyển các commit của 'feature/my-feature' lên đầu nhánh 'main'
```

`git squash`

**Công dụng**: `Squash` là 1 kỹ thuật kết hợp nhiều commit nhỏ thành 1 commit lớn. Điều này thường được thực hiện thông qua `git rebase -i` để tạo ra lịch sử commit gọn hơn trước khi merge

Ví dụ:

```
git rebase -i HEAD~3 # Mở trình chỉnh sửa để squasnh 3 commit gần nhất
```

`git reset`

**Công dụng**: Di chuyển con trỏ HEAD và thay đổi trạng thái của staging area và thư mục làm việc. Có 3 chế độ phổ biến:

- `git reset --soft <commit-id>`: Di chuyển HEAD đến `<commit-id>`, giữ nguyên các thay đổi trong staging area và working directory
- `git reset --mixed <commit-id>`: Di chuyển HEAD dến `<commit-id>`, đưa các thay đổi từ staging area về working directory
- `git reset --hard <commit-id>`: Di chuyển HEAD đến `<commit-id`, loại bỏ tất cả các thay đổi trong staging area và working directory. Cực kỳ cẩn thận với `--hard` vì nó sẽ xóa vĩnh viễn các thay đổi chưa commit

Ví dụ

```
git log --oneline # Xem lịch sử để lấy commit-id
# Giả sử lịch sử:
# abcdef1 Commit mới nhất
# 1234567 Commit trước đó
# fedcba9 Commit rất cũ

git reset --soft 1234567 # HEAD về 1234567, thay đổi của abcdef1 vẫn staged
git reset --mixed 1234567 # HEAD về 1234567, thay đổi của abcdef1 về unstaged
git reset --hard 1234567 # HEAD về 1234567, tất cả thay đổi của abcdef1 bị xóa

git reset HEAD^ # Quay lại một commit trước đó (mặc định là --mixed)
git reset --hard HEAD~2 # Quay lại hai commit trước đó và xóa mọi thay đổi
```

`git revert <commit-id>`

**Công dụng**: Tạo một commit mới để hoàn tác các thay đổi được giới thiệu bởi một commit trước đó. Lệnh này không xóa lịch sử, mà thêm một "commit hoàn tác", an toàn hơn khi làm việc trên các nhánh đã được chia sẻ.

Ví dụ:

```
git revert 2a3b4c5 # Hoàn tác các thay đổi của commit có ID 2a3b4c5
```

**6. Xem lịch sử và quản lý nhánh**

`git branch`

**Công dụng**: Liệt kê, tạo và xóa các nhánh

- `git branch`: Liệt kê tất cả các nhánh cục bộ
- `git branch -a`: Liệt kê tất cả các nhánh cục bộ và remote
- `git branch <tên-nhánh>`: Tạo 1 nhánh mới
- `git branch -d <tên-nhánh>`: Xóa một nhánh cục bộ (Chỉ khi nó đã được hợp nhất)
- `git branch -D <tên-nhánh>`: Buộc phải xóa 1 nhánh cục bộ (Ngay cả khi nó chưa được hợp nhất)

Ví dụ

```
git branch             # Liệt kê các nhánh cục bộ
git branch -a          # Liệt kê tất cả các nhánh (cục bộ và remote)
git branch feature/new-design # Tạo nhánh 'feature/new-design'
git branch -d bugfix/typo     # Xóa nhánh 'bugfix/typo'
git branch -D experimental-branch # Buộc xóa nhánh
```

`git log`

**Công dụng**: Hiển thị lịch sử commit của repository. Có nhiều tùy chọn để tùy chỉnh hiển thị.\

Một số tùy chọn khác:

- `git log --oneline`: Hiển thị mỗi commit trên 1 dòng duy nhất
- `git log --graph --oneline --decorate`: Hiển thị lịch sử dưới dạng đồ thị ASCII, rất hữu ích để hình dung các nhánh và merge
- `git log -p`: Hiển thị sử khác biệt của mỗi commit

`git remote`

**Công dụng**: Quản lý các kết nối đến các commit repository. Remote là nơi pull và push code

Các lệnh con:

- `git remote`: Liệt kê tên của các remote repo đã được cấu hình. Mặc định, tên của remote đầu tiên khi `git clone` là `origin`
- `git remote -v`: Liệt kê tên của các remote repository cùng với URL của chúng (bao gồm cả URL cho việc fetch và push). Dùng khi muốn kiểm tra xem remote `origin` đang trỏ đến đâu
- `git remote add <tên remote>`: Thêm một remote repository mới vào cấu hình. `origin` là tên remote phổ biến nhất, nhưng có thể đặt tên bất kỳ.
- `git remote set-url origin <tên remote>`: Lệnh này cho phép thay đổi địa chỉ URL của một remote repository đã tồn tại. Đây là một lệnh hữu ích khi cần di chuyển dự án của mình sang một máy chủ Git mới, hoặc khi URL của repository thay đổi (ví dụ: từ HTTP sang SSH).

**7. Bổ sung**

- Thay đổi lịch sử commit với `git rebase`:

> 1.  Chọn commit cần thay đổi message, copy mã commit nằm ngay sau commit đó. Cú pháp `git rebase -i <mã commit>`
> 2.  Tùy chọn `r` thay vì `pick` để thay đổi message commit sau đó nhấn `:wq`
> 3.  Giao diện màn hình hiển thị ra, nhập message cần thay đổi. Sau đó nhấn `:wq`
> 4.  Sử dụng `git push --force` để ghi đè lịch sử commit trước đó
