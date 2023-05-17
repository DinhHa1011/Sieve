```
require [ "imap4flags","fileinto","mailbox","copy",...];
```
## Một vài config sieve thông dụng
### Subject chứa "hi" => đánh dấu sao
```
if allof (header :contains "Subject" "hi")
{setflag ["\\Flagged"];}
```
### Subject chứa "sach" => đánh dấu đã xem
```
if allof (header :contains "Subject" "sach")
{setflag ["\\Seen"];}
```
### Mail gửi từ dinhthiha1011@gmail.com => mail được lưu trong file INBOX.AHa
```
if allof (header :contains "from" "dinhthiha1011@gmail.com")
{fileinto :create "INBOX.AHa";}
```
### auto fw sang test2@anthanh264.site
```
if true {redirect:copy "test2@anthanh264.site";}
```
### Mail gửi từ anthanh264@gmail.com => fw sang hadt@pikab.in
```
if allof (header :contains "from" "anthanh264@gmail.com")
 {redirect :copy "hadt@pikab.in";}`
```
