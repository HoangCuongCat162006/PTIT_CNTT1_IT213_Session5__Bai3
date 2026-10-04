# BÁO CÁO THỰC HÀNH: BÀI 3 - XỬ LÝ XUNG ĐỘT PHỨC TẠP TRONG QUÁ TRÌNH REBASE

- **Khóa học / Module:** IT_209 - Quản lý Phiên bản với Git (Session 5)
- **Bài tập:** Bài 3 - Xử lý xung đột phức tạp trong quá trình Rebase
- **Đường dẫn nộp bài đề xuất:** `homework/session_05/ex3/`
- **Học viên:** Hoang Cuong Cat (`cathoangfb1@gmail.com`)

---

## 1. MỤC TIÊU BÀI THỰC HÀNH

1. **Hiểu bản chất khác biệt giữa Conflict trong Merge và Rebase:**
   - **Merge:** Gộp toàn bộ thay đổi của hai nhánh lại tại một thời điểm, chỉ xảy ra xung đột duy nhất 1 lần (nếu có), tạo ra một *Merge Commit* để kết nối hai nhánh.
   - **Rebase:** Tách rời từng commit của nhánh tính năng, sau đó "chơi lại" (replay) từng commit một lên trên đỉnh của nhánh mục tiêu. Do đó, xung đột có thể diễn ra lặp lại ở **từng commit (step-by-step)** trong tiến trình rebase.
2. **Kỹ năng giải quyết xung đột từng chặng (Step-by-step Conflict Resolution):**
   - Đọc hiểu conflict marker trong ngữ cảnh rebase (nơi `HEAD` trỏ vào nhánh đích đang rebase lên).
   - Giải quyết thủ công từng commit, kết hợp giữ logic mới của nhánh chính mà không làm hỏng tính năng của nhánh phụ.
3. **Đạt được lịch sử Git tuyến tính (Linear History):**
   - Đưa các commit của nhánh `feature-api` lên trên đỉnh của `main` một cách thẳng hàng, không sinh merge commit thừa.

---

## 2. THIẾT LẬP KỊCH BẢN THỰC NGHIỆM

### Bước 1: Khởi tạo repository và commit gốc trên `main`
Khởi tạo repo và tạo file cấu hình ban đầu `config.json`:
```bash
git init -b main
```

Nội dung ban đầu của `config.json`:
```json
{
  "port": 8080,
  "debug": false
}
```

Commit ban đầu:
```bash
git add config.json
git commit -m "init config"
# Hash commit: 7fdad45
```

---

### Bước 2: Nhánh `feature-api` phát triển độc lập (2 commits)
Tạo và chuyển sang nhánh `feature-api`:
```bash
git checkout -b feature-api
```

- **Commit 1 (`feat: change port`):**
  Sửa `"port": 9000` trong `config.json`:
  ```json
  {
    "port": 9000,
    "debug": false
  }
  ```
  ```bash
  git add config.json
  git commit -m "feat: change port"
  # Hash commit: a7c727e
  ```

- **Commit 2 (`feat: enable debug`):**
  Sửa tiếp `"debug": true` trong `config.json`:
  ```json
  {
    "port": 9000,
    "debug": true
  }
  ```
  ```bash
  git add config.json
  git commit -m "feat: enable debug"
  # Hash commit: 0e3c025
  ```

---

### Bước 3: Nhánh `main` cập nhật đồng thời từ thành viên khác (2 commits)
Quay lại nhánh `main`:
```bash
git checkout main
```

- **Commit 1 (`update port on main`):**
  Thành viên khác đổi port thành `8081`:
  ```json
  {
    "port": 8081,
    "debug": false
  }
  ```
  ```bash
  git add config.json
  git commit -m "update port on main"
  # Hash commit: aca0975
  ```

- **Commit 2 (`add env config`):**
  Thành viên khác thêm trường `"env": "production"`:
  ```json
  {
    "port": 8081,
    "debug": false,
    "env": "production"
  }
  ```
  ```bash
  git add config.json
  git commit -m "add env config"
  # Hash commit: 20c7a8a
  ```

---

### Sơ đồ nhánh trước khi Rebase:
Lệnh kiểm tra: `git log --graph --oneline --all`
```text
* 20c7a8a (main) add env config
* aca0975 update port on main
| * 0e3c025 (HEAD -> feature-api) feat: enable debug
| * a7c727e feat: change port
|/  
* 7fdad45 init config
```
> **Nhận xét:** Hai nhánh đã rẽ nhánh (diverged) từ commit gốc `7fdad45`. Cả hai nhánh đều sửa đổi cùng tệp `config.json` tại các dòng trùng nhau.

---

## 3. TIẾN TRÌNH REBASE VÀ GIẢI QUYẾT XUNG ĐỘT TỪNG CHẶNG

Học viên chuyển về nhánh `feature-api` và thực hiện rebase lên `main`:
```bash
git checkout feature-api
git rebase main
```

---

### CHẶNG 1: Xung đột tại commit 1/2 (`feat: change port`)

#### 1. Thông báo lỗi từ Git:
```text
Rebasing (1/2)Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply a7c727e... feat: change port
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
hint: You can instead skip this commit: run "git rebase --skip".
hint: To abort and get back to the state before "git rebase", run "git rebase --abort".
```

#### 2. Kiểm tra `git status`:
```text
interactive rebase in progress; onto 20c7a8a
Last command done (1 command done):
   pick a7c727e # feat: change port
Next command to do (1 remaining command):
   pick 0e3c025 # feat: enable debug
Unmerged paths:
	both modified:   config.json
```

#### 3. Phân tích Conflict Marker trong `config.json`:
```json
{
<<<<<<< HEAD
  "port": 8081,
  "debug": false,
  "env": "production"
=======
  "port": 9000,
  "debug": false
>>>>>>> a7c727e (feat: change port)
}
```
- **Ý nghĩa `HEAD` trong Rebase:** Trong quá trình rebase, `HEAD` tạm thời trỏ đến commit của nhánh đích (`main` - cụ thể là commit `20c7a8a add env config`).
- **Ý nghĩa commit patch (`a7c727e`):** Là commit của nhánh tính năng `feature-api` đang được áp dụng lại.
- **Mục tiêu giải quyết:** Nhánh `feature-api` muốn đặt `"port": 9000`, đồng thời **không được làm mất** cấu hình `"env": "production"` mới được bổ sung trên `main`.

#### 4. Giải quyết xung đột Chặng 1:
Chỉnh sửa file `config.json` thành:
```json
{
  "port": 9000,
  "debug": false,
  "env": "production"
}
```

Đánh dấu đã giải quyết và tiếp tục quá trình rebase:
```bash
git add config.json
git rebase --continue
```
> Git áp dụng thành công commit 1 dưới mã băm mới `e8dca88` và chuyển sang commit 2.

---

### CHẶNG 2: Xung đột tại commit 2/2 (`feat: enable debug`)

#### 1. Thông báo lỗi từ Git:
```text
[detached HEAD e8dca88] feat: change port
 1 file changed, 1 insertion(+), 1 deletion(-)
Rebasing (2/2)Auto-merging config.json
CONFLICT (content): Merge conflict in config.json
error: could not apply 0e3c025... feat: enable debug
hint: Resolve all conflicts manually, mark them as resolved with
hint: "git add/rm <conflicted_files>", then run "git rebase --continue".
```

#### 2. Kiểm tra `git status`:
```text
interactive rebase in progress; onto 20c7a8a
Last commands done (2 commands done):
   pick a7c727e # feat: change port
   pick 0e3c025 # feat: enable debug
No commands remaining.
Unmerged paths:
	both modified:   config.json
```

#### 3. Phân tích Conflict Marker trong `config.json`:
```json
{
  "port": 9000,
<<<<<<< HEAD
  "debug": false,
  "env": "production"
=======
  "debug": true
>>>>>>> 0e3c025 (feat: enable debug)
}
```
- **Phân tích:** 
  - Phần `port: 9000` đã được thống nhất từ chặng 1.
  - Phía `HEAD` hiện tại có `"debug": false` và `"env": "production"`.
  - Phía patch `0e3c025` có `"debug": true`.
- **Mục tiêu giải quyết:** Giữ `"debug": true` theo đúng tính năng của commit, đồng thời giữ `"env": "production"` của nhánh chính với cú pháp JSON hợp lệ (có dấu phẩy phân tách).

#### 4. Giải quyết xung đột Chặng 2:
Chỉnh sửa file `config.json` hoàn chỉnh:
```json
{
  "port": 9000,
  "debug": true,
  "env": "production"
}
```

Đánh dấu đã giải quyết và hoàn tất rebase:
```bash
git add config.json
git rebase --continue
```

Kết quả thông báo thành công:
```text
[detached HEAD 0b41276] feat: enable debug
 1 file changed, 1 insertion(+), 1 deletion(-)
Successfully rebased and updated refs/heads/feature-api.
```

---

## 4. KẾT QUẢ KIỂM TRA SAU KHI HOÀN THÀNH

### 1. Trạng thái repository (`git status`):
```text
$ git status
On branch feature-api
nothing to commit, working tree clean
```

### 2. Kiểm tra lịch sử Git Graph (`git log --graph --oneline --decorate --all`):
```text
$ git log --graph --oneline --decorate --all
* 0b41276 (HEAD -> feature-api) feat: enable debug
* e8dca88 feat: change port
* 20c7a8a (main) add env config
* aca0975 update port on main
* 7fdad45 init config
```

> **Đánh giá kết quả:**
> - Toàn bộ lịch sử các commit nằm trên **một đường thẳng duy nhất (linear history)**.
> - Không hề xuất hiện bất kỳ merge commit phụ nào.
> - Hai commit của `feature-api` (`e8dca88`, `0b41276`) được đặt nối tiếp ngay sau commit mới nhất của `main` (`20c7a8a`).

### 3. Kiểm tra nội dung tệp `config.json`:
- **Trên nhánh `feature-api` (`git show feature-api:config.json`):**
  ```json
  {
    "port": 9000,
    "debug": true,
    "env": "production"
  }
  ```
  *(Đầy đủ port 9000, debug true của nhánh feature và giữ nguyên env production của nhánh main)*

- **Trên nhánh `main` (`git show main:config.json`):**
  ```json
  {
    "port": 8081,
    "debug": false,
    "env": "production"
  }
  ```
  *(Nhánh main hoàn toàn nguyên vẹn, không bị phá vỡ hay ảnh hưởng)*

---

## 5. SO SÁNH GIẢI QUYẾT XUNG ĐỘT: MERGE VS REBASE

| Tiêu chí | Git Merge | Git Rebase |
| :--- | :--- | :--- |
| **Cơ chế xử lý** | Gộp toàn bộ thay đổi của 2 nhánh lại tại một điểm commit duy nhất. | Tua lại (replay) từng commit của nhánh tính năng lên trên đầu nhánh đích. |
| **Số lần phát sinh xung đột** | Tối đa **1 lần** duy nhất cho toàn bộ quá trình merge. | Có thể phát sinh **nhiều lần** (tại từng commit xung đột). |
| **Ý nghĩa của `HEAD` khi conflict** | `HEAD` trỏ tới commit hiện tại của nhánh bạn đang đứng. | `HEAD` trỏ tới commit của **nhánh đích** mà bạn đang rebase lên. |
| **Lịch sử Git (Git History)** | Phi tuyến tính, có các nhánh rẽ và các commit gộp (`Merge branch ...`). | **Tuyến tính tuyệt đối (thẳng hàng)**, sạch sẽ, dễ đọc và dễ `git bisect`. |
| **Hash của Commit** | Giữ nguyên hash cũ của các commit tính năng, sinh thêm 1 hash mới cho merge commit. | **Tất cả các commit được rebase đều được tạo lại với commit hash mới**. |
| **Khả năng khôi phục / Hủy bỏ** | Dễ dàng hủy bỏ bằng `git merge --abort`. | Dễ dàng hủy bỏ bất cứ lúc nào bằng `git rebase --abort`. |

---

## 6. QUY TẮC VÀNG VÀ BÀI HỌC KINH NGHIỆM

1. **Quy tắc vàng của Rebase (The Golden Rule of Rebase):**
   > *Chỉ rebase trên nhánh cục bộ (local branch) chưa đẩy lên remote hoặc nhánh riêng của bạn. Tuyệt đối không bao giờ rebase trên các nhánh công khai/nhánh chung (như `main`, `develop`) khi đã có người khác kéo về.*
2. **Kỹ năng xử lý xung đột từng bước:**
   - Luôn đọc kỹ `git status` ở từng chặng để biết đang ở commit nào.
   - Hiểu rõ ngữ cảnh của commit hiện tại: commit đó muốn làm gì, nhánh chính đã thay đổi gì, để đưa ra quyết định hòa giải chuẩn xác nhất.
   - Sau khi sửa conflict, chỉ dùng `git add <file>` và `git rebase --continue`, tuyệt đối không dùng `git commit` thủ công trong quá trình rebase vì sẽ làm sai lệch flow.
3. **Lợi ích thực tế:**
   - Giúp Pull Request (PR) cực kỳ gọn gàng trước khi tích hợp vào nhánh chính.
   - Giúp việc review mã nguồn và truy vết lỗi (debugging / bisecting) nhanh chóng và chính xác.
