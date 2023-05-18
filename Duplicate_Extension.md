- Phần mở rộng duplicate RFC 7352 thêm một lệnh test mới được gọi là `duplicate` vào ngôn ngữ Sieve
- Test này thêm khả năng phát hiện duplications
- Ứng dụng chính cho test mới này là xử lý việc gửi duplicate thường do đăng ký danh sách gửi mail hoặc địa chỉ mail được chuyển hướng
- Việc phát hiện thường được thực hiện bằng cách match message IP với danh sách nội bộ gồm các message ID từ các message đã gửi trước đó
- Cho nhiều ứng dụng phức tạp, test duplicate có thể cũng sử dụng nội dung của một field header cụ thể hoặc phần khác của message
- Trước tiên, tiện ích mở rộng này dành riêng cho Dovecot và có sẵn dưới tên vnd.dovecot.duplicate.
- Cách triển khai khác biệt đáng kể so với những gì hiện được xuất bản dưới dạng RFC 7353, nhưng phần mở rộng ban đầu vẫn được hỗ trợ để tương thích ngược
### Configuration
- Tiện ích duplicate có sẵn theo mặc định
- Tiện ích mở rộng duplicate có cài đặt cụ thể của riêng nó
- Các cài đặt sau khả dụng (giá trị mặc định được chỉ định):
  `sieve_duplicate_default_period =14d`
  `sieve_duplicate_max_period = 7d`
    - Các tùy chọn này lần lượt chỉ định giá trị mặc định và giá trị tối đa cho ktg sau đó các giá trị được theo dõi sẽ bị xóa khỏi cơ sở dữ liệu theo dõi duplicate. Thời gian mặc định là (s), trừ khi theo sau là ký tự xác định (m),(h),(d)
    #### Example
    ```
    plugin {
      sieve = ~/.dovecot.sieve
      
      sieve_duplicate_default_period = 1h
      sieve_duplicate_max_period = 1d
    }
    ```
### My Config
- Sửa file (thêm user_query) vim /etc/dovecot/dovecot-sql.conf.ext
```
user_query = SELECT CONCAT('/var/mail','/','%d','/','%n') as home, 'vmail' as uid, 'vmail'as gid,CONCAT('/var/mail','/','%d','/','%n') as mail FROM virtual_Users WHERE email= '%u';
```
- vim /etc/dovecot/sieve/before.sieve
```
require ["duplicate","fileinto","mailbox"];
if allof (duplicate :header "Subject")
{fileinto :create "INBOX.AHa";}
```
```
sievec /etc/dovecot/sieve/before.sieve
systemctl restart dovecot
```
