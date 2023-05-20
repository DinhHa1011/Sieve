- Sieve `include` extension RFC 6609 cho phép người dùng đưa một Sieve script vào một script khác
- Điều này có thể làm cho việc quản lý các script lớn hoặc nhiều script trở nên dễ dàng hơn nhiều và cho phép một trang web cũng như user của nó xây dựng các thư viện của script
- Người dùng có thể bao gồm các script các nhân của riêng họ hoặc script site-wide
- Các script được bao gồm có thể bao gồm nhiều script của riêng chúng, tạo ra một cây các script được bao gồm với script chính (thường là script cá nhân của người dùng) ở gốc
### Configuration
- `include` extension có sẵn theo mặc định
- `include` extension có các cài đặt cụ thể của riêng nó
- Các cài đặt sau có thể được định cấu hình cho `include` extension (các giá trị mặc định được chỉ định):

`sieve_include_max_includes = 255`
  - Số tối đa script có thể bao gồm
  - Đây là tổng số script tối đa trong include tree
 
`sieve_include_max_nesting_depth = 10`
  - Độ sâu tối đa của tree
### My config
- Sửa file `/etc/dovecot/conf.d/90-sieve.conf`
```
sieve_global = /sieve
```
- Sửa `file /etc/dovecot/sieve/before.sieve` (để đọc file flag.sieve)
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "flag";
```
- sievec & restart (sau khi config file flag)
```
sievec sieve/before.sieve
systemctl restart dovecot
```
- Tạo folder `/sieve`
```
mkdir /sieve
```
- Thêm file `flag.sieve`
```
require [ "imap4flags","fileinto","mailbox","copy", "body", "regex"];
if allof (header :contains "Subject" "hi")
{setflag ["\\Flagged"];}
```
- sievec & restart
```
sievec sieve/flag.sieve
systemctl restart dovecot
```
