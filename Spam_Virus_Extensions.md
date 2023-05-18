- Sử dụng spamtest và virustest extensions RFC 5235, ngôn ngữ Sieve cung cấp một giao diện lệnh thống nhất và được tiêu chuẩn hóa để đánh giá các bài kiểm tra spam và virus được thực hiện trên message
- Người dùng không còn cần biết những header nào cần được kiểm tra và phán quyết của scanner trong giá trị field header. Họ chỉ cần biết sử dụng extension `spamtest` (`spamtestplus`) và virustest như thế nào
- Điều này cũng cung cấp cho các editors Sieve dựa trên GUI-based để cung cấp giao diện di dộng và dễ cài đặt cho cấu hình bộ lọc spam và virus
### Configuration
- `spamtest`, `spamtestplus`, `virustest` extensions không enable theo mặc định, do đó cần enabled rõ ràng cài đặt sử dụng `sieve_extensions`
- Cho phép setting cần config cho sử dụng `spamtest` và `spamtestplus` extensions
- `virustest` extension có cài đặt cấu hình giống hệt nhau, nhưng với một `sieve_virustest_prefix` thay vì `sieve_spamtest_prefix`:
`sieve_spamtest_status_type = "score" / "strlen" / "text"
  - Đây là loại cụ thể của trạng thái result mà trình quét spam/virus tạo ra.
  - Điều này có thể là một điểm số (score), một chuỗi các ký tự giống hệt nhau (strlen), ví dụ `*******`, hoặc một văn bản miêu tả (text), ví dụ {{{Spam}}} hoặc Not Spam
`sieve_spamtest_status_header = <header-field> [ ":" <regexp> ]

