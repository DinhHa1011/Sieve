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

