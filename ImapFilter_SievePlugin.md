- Thông thường, Sieve filter có thể được áp dụng khi gửi mail ban đầu hoặc được kích hoạt bở các sự kiện nhất định trong giao thức IMAP (IMAPSIEVEC;RFC 6785)
- Người dùng có thể config Sieve script nào sẽ chạy trong các trường hợp này, nhưng nó không thể kích hoạt việc thực thi các Sieve scripts theo cách thủ công
- Tuy nhiên, điều này có thể rất hữu ích, ví dụ: để kiểm tra các Sieve rules mới và lọc lại các message đã bị xử lý sai bởi version trước đó của Sieve script có liên quan
- Pigeonhole cung cấp `imap_filter_sieve` plugin, nơi cung cấp IMAP extension gọi là `FILTER=SIEVE`
- Điều này thêm một dòng lệnh `FILTER` cho phép áp dụng mail filter (Sieve script) trên cài đặt của message để match với tiêu chí tìm kiếm hình ảnh được chỉ định
- Bản nháp mới nhất của thông số kỹ thuật cho khả năng IMAP này có sẵn tại đây. Plugin này là thử nghiệm và thông số kỹ thuật có thể thay đổi, sử dụng thông số kỹ thuật có trong bản phát hành hiện tại của bạn dể có được thông số kỹ thuật phù hợp cho bản phát hành của bạn
- Plugin này có sẵn cho Pigeonhole v0.4.24 trở lên (có sẵn cho Doveoct v2.2.36) và v0.5.2 trở lên (có sẵn cho Dovecot v2.3.2)
- Các plugin được bao gồm trong gói Pigeonhole và do đó được biên dịch và cài đặt hoàn toàn với chính Pigeonhole
### Setting
- See https://doc.dovecot.org/settings/plugin/imap-filter-sieve-plugin/#imap-filter-sieve
### Configuration
- IMAP FILTER Sieve plugin được kích hoạt bằng cách thêm nó vào cài đặt `mail_plugins` cho imap protocol:
```
protocol imap {
  mail_plugins = $mail_plugins imap_filter_sieve
}
```
- Lưu ý rằng enable plugin này cho phép người dùng chỉ định nội dung Sieve script như một tham số cho dòng lệnh `FILTER`, không chỉ run các script được lưu trữ hiện có
- Plugin này sử dụng cài đặt config bình thường được sử dụng bởi LDA Sieve plugin lúc delivery
- `sieve_before` và `sieve_after` script hiện đang bị plugin này bỏ qua
