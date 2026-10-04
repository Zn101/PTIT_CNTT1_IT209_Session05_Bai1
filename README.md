# Bài 1: Khôi phục commit đã mất bằng Git Reflog

**Session 05 – Exercise 1** · Đường dẫn nộp bài: `homework/session_05/ex1/`

## 1. Mục tiêu

- Hiểu cách Git lưu lịch sử hoạt động cục bộ thông qua Reflog.
- Dùng `git reflog` để truy vết các tham chiếu commit cũ.
- Khôi phục commit đã mất sau khi chạy `git reset --hard`.

## 2. Các bước thực hiện

### Bước 1 – Chuẩn bị repository và commit quan trọng

```bash
mkdir session05-ex1
cd session05-ex1
git init
echo "# Session 05 - Exercise 1" > README.md
git add .
git commit -m "Initial commit"

echo "Day la tinh nang quan trong" > feature.txt
git add .
git commit -m "Them tinh nang quan trong"
```

Cần có commit `Initial commit` trước, vì `HEAD~1` ở bước sau phải trỏ tới một commit tồn tại.

### Bước 2 – Giả lập lỗi: reset hard lùi 1 commit

```bash
git reset --hard HEAD~1
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
HEAD is now at a1b2c3d Initial commit
-->
```

### Bước 3 – Phát hiện mất commit

```bash
git log --oneline
ls
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
a1b2c3d (HEAD -> main) Initial commit

README.md
-->
```

Commit `Them tinh nang quan trong` không còn trong `git log` và tệp `feature.txt` cũng biến mất khỏi thư mục làm việc.

### Bước 4 – Tra cứu Reflog để tìm mã hash của commit đã mất

```bash
git reflog
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
a1b2c3d (HEAD -> main) HEAD@{0}: reset: moving to HEAD~1
e3a5b2c HEAD@{1}: commit: Them tinh nang quan trong
a1b2c3d (HEAD -> main) HEAD@{2}: commit (initial): Initial commit
-->
```

Dòng `HEAD@{1}: commit: Them tinh nang quan trong` cho biết mã hash của commit cần khôi phục (ví dụ `e3a5b2c`).

### Bước 5 – Khôi phục commit

```bash
git reset --hard e3a5b2c
```

(Thay `e3a5b2c` bằng mã hash thật trong reflog. Có thể dùng cách viết tương đương: `git reset --hard HEAD@{1}`.)

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
HEAD is now at e3a5b2c Them tinh nang quan trong
-->
```

## 3. Kiểm tra kết quả

### 3.1. Lịch sử commit

```bash
git log --oneline
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
e3a5b2c (HEAD -> main) Them tinh nang quan trong
a1b2c3d Initial commit
-->
```

### 3.2. Nội dung tệp đã được khôi phục

```bash
cat feature.txt
```

Kết quả:

```text
Day la tinh nang quan trong
```

### 3.3. Reflog sau khi khôi phục

```bash
git reflog
```

Kết quả:

```text
<!-- DÁN KẾT QUẢ THỰC TẾ CỦA BẠN VÀO ĐÂY, ví dụ:
e3a5b2c (HEAD -> main) HEAD@{0}: reset: moving to e3a5b2c
a1b2c3d HEAD@{1}: reset: moving to HEAD~1
e3a5b2c (HEAD -> main) HEAD@{2}: commit: Them tinh nang quan trong
a1b2c3d HEAD@{3}: commit (initial): Initial commit
-->
```

### 3.4. Ảnh chụp màn hình

![Kết quả git log --oneline và git reflog](./screenshot.png)

## 4. Phân tích

| Nội dung | Giải thích |
| --- | --- |
| Vì sao commit "biến mất" | `git reset --hard HEAD~1` chỉ dời con trỏ nhánh về commit trước và ghi đè Staging Area, Working Directory. Commit cũ vẫn nằm trong `.git/objects`, chỉ là không còn nhánh nào trỏ tới nên `git log` không hiển thị. |
| Reflog là gì | Nhật ký ghi lại mọi lần `HEAD` thay đổi vị trí (commit, reset, checkout, merge, rebase...). `HEAD@{n}` là vị trí của `HEAD` cách đây n bước. |
| `git log` so với `git reflog` | `git log` chỉ đi theo chuỗi commit cha từ `HEAD`; `git reflog` liệt kê cả những commit không còn được nhánh nào tham chiếu. |
| Khôi phục bằng `reset` hay `checkout` | `git reset --hard <hash>` đưa nhánh hiện tại về lại commit đó. `git checkout <hash>` chỉ xem commit ở trạng thái detached HEAD, cần tạo nhánh mới (`git switch -c <ten-nhanh>`) để giữ lại. |
| Giới hạn | Reflog chỉ tồn tại ở local, không được push lên remote, và các mục cũ sẽ hết hạn (mặc định khoảng 90 ngày; 30 ngày với commit không còn được tham chiếu) rồi bị `git gc` dọn. Thay đổi **chưa từng commit** thì reflog không cứu được. |

## 5. Kết luận

- Đã xác định commit bị mất thông qua `git reflog`.
- Đã khôi phục commit `Them tinh nang quan trong` bằng `git reset --hard <hash>`, không viết lại code thủ công.
- `git log` hiển thị lại đúng commit và `feature.txt` có nội dung ban đầu.
