# Báo cáo: Ứng dụng mô hình IPO và Kỹ năng kiểm chứng thông tin khi làm việc với AI

## 1. Bối cảnh & Nhận diện rủi ro (AI Hallucination)
Khi xử lý lỗi `"Path not found"` lúc cài đặt Python, AI có thể sinh ra các câu lệnh Terminal chứa rủi ro:
- **AI Hallucination (Ảo giác AI):** Tự ghép nối các cờ (flags) hoặc câu lệnh không có thật, hoặc lấy nhầm cú pháp của Linux (Bash) áp dụng cho Windows (CMD/PowerShell).
- **Lệnh phá hủy ngầm (Destructive Commands):** Cú pháp chạy được nhưng gây hậu quả nghiêm trọng (ví dụ: dùng `setx PATH "..."` ghi đè làm mất toàn bộ biến môi trường hệ thống cũ).

---

## 2. Phân tích kiểm chứng theo mô hình IPO

### 📥 INPUT (Đầu vào từ AI)
- **Tình huống:** Hỏi AI cách sửa lỗi `"Path not found"` khi gọi Python trên Terminal.
- **Câu lệnh AI đề xuất:** 
  ```cmd
  setx /M PATH "C:\Users\Admin\AppData\Local\Programs\Python\Python312"
  ```
- **Dấu hiệu cảnh báo:** Lệnh yêu cầu quyền quản trị (`/M`) và tác động trực tiếp vào biến hệ thống `PATH`.

### ⚙️ PROCESS (Quy trình kiểm chứng đa tầng)
1. **Phân tách thành phần lệnh:** 
   - Lệnh gốc: `setx`
   - Tham số: `/M` (cấp độ System-wide), `PATH` (biến đích).
2. **Tra cứu chéo bằng toán tử tìm kiếm (Google Dorking):**
   - Kiểm tra rủi ro thực tế từ cộng đồng:
     ```text
     "setx" "PATH" overwrite problem site:stackoverflow.com
     ```
   - Đối chiếu tài liệu gốc của nhà phát triển hệ điều hành & ngôn ngữ:
     ```text
     "setx" syntax site:learn.microsoft.com
     "Add Python to PATH" site:docs.python.org
     ```
3. **Kiểm tra tĩnh trên máy:** Chạy `setx /?` để đọc cảnh báo từ chính hệ điều hành mà không thực thi lệnh thay đổi dữ liệu.

### 📤 OUTPUT (Kết luận & Hành động)
- **Đánh giá độ an toàn:** ❌ **KHÔNG AN TOÀN (Unsafe)**.
- **Nguyên nhân kỹ thuật:** Theo kết quả từ `site:stackoverflow.com` và `site:learn.microsoft.com`, câu lệnh trên không nối thêm đường dẫn mà **xóa sạch toàn bộ đường dẫn hệ thống cũ** và thay bằng mỗi đường dẫn Python, đồng thời cắt cụt dữ liệu nếu vượt quá 1024 ký tự.
- **Giải pháp chuẩn (Safe Action):** Không chạy lệnh của AI. Chạy lại trình cài đặt Python chính thức (`python-installer.exe`) -> chọn **Modify** -> tích chọn **"Add Python to environment variables"**, hoặc thêm thủ công qua giao diện `Edit environment variables` của Windows.

---

## 3. Thiết kế Prompt "Hỏi ngược lại" AI (Reverse Prompting)

```text
Trước khi tôi thực thi câu lệnh bạn vừa đưa ra, hãy giải trình độ an toàn theo 4 điểm sau:
1. Phân tích từng tham số (flags) trong câu lệnh và xác nhận chúng có tồn tại trong tài liệu chính thức của hệ điều hành tôi đang dùng hay không.
2. Câu lệnh này có ghi đè (overwrite) hay xóa mất cấu hình PATH hiện tại của máy không?
3. Nếu xảy ra lỗi sau khi chạy, làm thế nào để hoàn tác (rollback) về trạng thái trước đó?
4. Đề xuất cho tôi cách thực hiện tương đương thông qua giao diện đồ họa (GUI) an toàn hơn.
```