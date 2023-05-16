#### 1. Sieve

  - Là một ngôn ngữ lậ trình có thể được sử dụng cho email filtering 
  - Sieve là một ngôn ngữ data-driven programming, tương tự như các ngôn ngữ lọc email trước đó là procmail và maildrop và các ngôn ngữ định hướng dòng trước đó như sed và AWK: nó chỉ định các điều kện để phù hợp với các hành động cần phải phù hợp 
  - Điều này khác với `general-purpose programming languages` ở chỗ nó rất hạn chế:
    - The base standard has no variables
    - No loop (but does allow conditional branching)
    - Ngăn chặn runaway programs and limit language to simple filtering operations\
  - Mặc dù, extensions đã được đưa ra để mở rộng ngôn ngữ để bao gồm biến `and`, một limit degree, loop, ngôn ngữ vẫn bị hạn chế cao, và do đó phù hợp để chạy các chương trình do người dùng phát triển như một phần của mail system
  - Ngoài ra còn một số lượng lớn các hạn chế đối với grammar của language, để giảm sự phức tạp của việc phân tích ngôn ngữ, nhưng ngôn ngữ cũng support người dùng nhiều phương thức để so sánh các chuỗi cục bộ và fully Unicode-aware
  - Trong khi Sieve ban đầu được hình thành là tool bên ngoài SMTP, RFC 5492 đã mở rộng nó để cho phép từ chối ở cấp độ giao thức SMTP

#### 2. Use

  - The Sieve script có thể tạo ra bởi 1 GUI-based rules editor hoặc chúng có thể được nhập trực tiếp từ text editor
  - The script được chuyển đến mail server theo cách server-dependent
  - ManageSieve protocol cho phép người dùng quản lý Sieve scripts của họp trên một remote server
  - Mail servers với local users có thể cho phép scripts lưu trữ trong  e.g. a .sieve file trong users' home directories
