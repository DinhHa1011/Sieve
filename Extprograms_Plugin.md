- "sieve_extprograms" plugin cung cấp một extension cho ngôn ngữ Sieve filtering thêm dòng lệnh action mới để gọi một bộ chương trình bên ngoài được xác định trước
- Message có thể được chuyển đến hoặc lọc qua các chương trình và dữ liệu chuỗi có thể được nhập và truy xuất từ các chương trình đó
- Để giảm thiểu các lo ngại về bảo mật, các chương trình bên ngoài không được chọn tùy ý; các chương trình khả dụng bị hạn chế thông qua cấu hình admin
- Plugin này chỉ có sẵn cho Pigeonhole v0.3 trở lên và cao hơn (có sẵn cho Dovecot v2.1) 
- Đối với Pigeonhole v0.4. plugin này là một phần của bản phát hành. Đây là một sự phát triển của Pipe plugin cho Pigeonhole v0.2 và hiện cung cấp các `filter` và dòng lệnh `execute` (và các phần mở rộng tương ứng) ngoài lệnh `pipe` đã được cung cấp trước đó bởi pipe
### Configuration
- Plugin được hoạt động bằng cách thêm nó vào setting `sieve_plugins`
```
sieve_plugin = sieve_extpreograms
```
- Plugin này đăng ký `vnd.dovecot.pipe`, `vnd.dovecot.filter` và `vnd.dovecot.execute` extension với thông dịch viên Sieve. 
- Tuy nhiên, extension đó thì không enabled theo mặc định và do đó cần enabled rõ ràng. Nó thì được recommend để hạn chế việc sử dụng các extension này trong ngữ cảnh global bằng cách thêm chúng vào cài đặt `sieve_global_extensions`
- Nếu các user script cá nhân cũng cần truy cập trực tiếp vào các chương trình bên ngoài, extension cần được thêm vào cài đặt `sieve_extensions`
- Các lệnh được giới thiệu bởi extension ngôn ngữ Sievetrong plugin này có thể chuyển trược tiếp một message hoặc chuỗi dữ liệu sang 1 chương trình bên ngoài (thường là shell script) bằng cách rẽ nhánh một process mới
- Cách khác, chúng ta có thể kết nối một socket unix mà dịch vụ tập lệnh dovecot đang lắng nghe để bắt đầu chương trình bên ngoài, ví dụ: để thực thi với tư cách người dùng khác hoặc để tăng cường bảo mật
- Tên chương trình được chỉ định cho các lệnh pipe, filter, execute mới được sử dụng để tìm chương trình hoặc socket trong một thư mục được định cấu hình
- Các thư mục riêng biệt được chỉ định cho các socket và các file nhị phân được thực thi trực tiếp, thư mục socket được tìm kiếm đầu tiên. Vì sử dụng "/" trong tên chương trình bị cấm nên không thể xây dựng cấu trúc phân cấp
- Các chương trình rẽ nhánh trực tiếp được thực thi với một tập hợp giới hạn cacsbieesn môi trường: `HOME`, `USE`, `SENDER`, `RECIPIENT` và `ORIG_RECIPIENT`
- Các chương trình được thực thi thông qua dịch vụ socket script-pipe hiện không có môi trường nào được cài đặt
- Nếu một shell script được thực thi để đọc một message họăc chuỗi dữ liệu, nó phải đọc đầy đủ đầu vào được cung cấp cho đến khi dữ liệu kết thúc bằng EOF, nếu không hành động sieve gọi chương trình sẽ thất bại. Hoạt động cũng sẽ fail khi shell script trả về một nonzero exit code
- Đầu ra tiêu chuẩn có sẵn để trả về một thông báo (đỗi với lệnh filter) hoặc chuỗi dữ liệu (đối với lệnh thực thi) cho trình thông dịch Sieve. Lỗi tiêu chuẩn được ghi vào file log LDA
- Ba extension được giới thiệu trong plugin này - `vnd.dovecot.pipe`, `vnd.dovecot.filter` và `vnd.dovecot.execute` - mỗi cái cấu hình riêng biệt nhưng giống nhau
- Các cài đặt cấu hình sau được sử dụng, "<extension>" trong cài đặt tên được thay thế bởi `pipe`, `filter`, `execute` tùy thuộc vào extension nào đang được config:
  
`sieve_extension_socket_dir=`
    - Trỏ đến một thư mục liên quan tới Dovecot base_dir nơi mà plugin tìm socket dịch vụ script
`sieve_extension_bin_dir=`
    - Trỏ đến thư mục mà plugin tìm chương trình (shell scripts) để thực thi trực tiếp và chuyển message tới
`sieve_extension_exec_timeout = 10s`
    - Config thời gian thực thi max sau đó chương trình bị buộc chấm dứt
`sieve_extension_input_eol = crlf`
    - Xác định chuỗi ký tự end-of-line được sử dụng cho dữ liệu được dẫn đến các chương trình bên ngoài
    - Theo mặc định `crlf` hiện nay, đại diện cho một chuỗi các ký tự carriage return (CR) và line feed (LF)
    - Điều này phù hợp với format tin nhắn Internet Message (RFC 5322) và bản thân Sieve sử dụng làm kết thúc dòng. Đặt cái này thành "lf" để sử dụng một ký tự LF thay thế
#### Configuration Example 1: socket service cho "pipe" và "execute"

