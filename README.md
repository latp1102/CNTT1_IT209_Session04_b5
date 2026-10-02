# Bài 5: Khôi phục trạng thái và Đảo ngược commit (Reset vs Revert)

Môn: DevOps Fundamentals  
Thực hiện: Học viên  
Ngày: 01/10/2026

---

## Mục tiêu
- Thực hành `git reset` để khôi phục lịch sử **cục bộ** (local).
- Thực hành `git revert` để đảo ngược commit **công khai** (public/shared) một cách an toàn.

---

## Trường hợp 1: Sửa đổi cục bộ — `git reset` (mixed)

### Bối cảnh
Đã tạo commit chứa mã nguồn Python bị lỗi ở local. Cần lùi HEAD lại 1 commit, giữ thay đổi dưới dạng **Working Directory Modified** (chưa staged) để viết lại code.

### Các bước thực hiện
```bash
git init
git config user.name "Student"
git config user.email "student@example.com"
cat > app.py << 'EOF'
# Good code
def hello():
    return 'Hello'
EOF
git add app.py
git commit -m "Add good app"

cat > app.py << 'EOF'
# Bad code
def hello():
    return 'Bad'  # error
EOF
git add app.py
git commit -m "Add bad feature"

git reset HEAD~1
```

### Kết quả kiểm tra
```bash
git status
```
Output:
```
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
	modified:   app.py

no changes added to commit (use "git add" and/or "git commit -a")
```

```bash
git log --oneline -n 5
```
Output:
```
68b53da Add good app
```

### Giải thích
- `git reset HEAD~1` (mixed) di chuyển HEAD và nhánh `master` lùi về commit trước đó.
- Thay đổi của commit bị lỗi **không bị mất**, chuyển về trạng thái **Modified** trong Working Directory, chưa staged.
- Không dùng `--hard` để đảm bảo an toàn, không làm mất mã nguồn.
- Phù hợp cho **công việc cá nhân** trên nhánh local chưa chia sẻ.

---

## Trường hợp 2: Sửa đổi công cộng — `git revert`

### Bối cảnh
Commit chứa lỗi đã tồn tại trên nhánh chung/remote. Cần đảo ngược thay đổi bằng cách **tạo commit mới** đối lập, đảm bảo lịch sử an toàn khi làm việc nhóm.

### Các bước thực hiện
```bash
git init
git config user.name "Student"
git config user.email "student@example.com"
cat > app.py << 'EOF'
# Good code
def hello():
    return 'Hello'
EOF
git add app.py
git commit -m "Initial good app"

cat > app.py << 'EOF'
# Bad code
def hello():
    return 'Bad'  # error
EOF
git add app.py
git commit -m "Add bad feature (public)"

git revert HEAD --no-edit
```

### Kết quả kiểm tra
```bash
git log --oneline -n 5
```
Output:
```
70e0ec6 Revert "Add bad feature (public)"
ec1a32e Add bad feature (public)
8475011 Initial good app
```

### Giải thích
- `git revert HEAD` tạo ra **commit mới** có nội dung đối lập với commit bị lỗi, khôi phục mã nguồn về trạng thái trước đó.
- Lịch sử cũ được giữ nguyên, có chứng từ rõ ràng (`Revert "..."`).
- Rất an toàn cho **nhánh chung/shared** vì không ghi đè lịch sử, không ảnh hưởng đến thành viên khác trong nhóm.

---

## So sánh Reset vs Revert

| Tiêu chí | `git reset` | `git revert` |
|---|---|---|
| **Mục đích** | Khôi phục trạng thái **cục bộ** | Đảo ngược commit **công khai** |
| **Lịch sử** | Xóa commit khỏi nhánh (rewrite history) | Thêm commit mới đối lập |
| **Thay đổi** | HEAD và nhánh lùi về; thay đổi có thể giữ trong Working Dir (mixed) hoặc mất (hard) | Tạo commit mới chứa delta đảo ngược |
| **An toàn nhóm** | **Không** dùng cho nhánh đã đẩy lên remote/shared | **Có** dùng được cho nhánh public |
| **Khi nào dùng** | Sửa lỗi local trước khi push | Sửa lỗi đã có trên remote/nhánh chung |

### Khi làm việc cá nhân
- Ưu tiên `git reset` (mixed hoặc soft) để dọn dẹp lịch sử local trước khi chia sẻ, tiết kiệm và gọn gàng.

### Khi làm việc nhóm trên nhánh chung
- **Tuyệt đối không dùng `reset`** (đặc biệt là `--hard` hoặc `--force`) vì sẽ ghi đè lịch sử, gây xung đột cho các thành viên khác.
- **Luôn dùng `git revert`** để tạo commit đảo ngược rõ ràng, đảm bảo tính minh bạch và an toàn.

---

## Tài liệu tham khảo
- Git Reference: https://git-scm.com/docs/git-reset
- Git Reference: https://git-scm.com/docs/git-revert
