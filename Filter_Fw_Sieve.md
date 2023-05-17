```
require [ "imap4flags","fileinto","mailbox","copy",...];
```
### Một vài config sieve thông dụng
#### 1. Subject chứa "hi" => đánh dấu sao
```
if allof (header :contains "Subject" "hi")
{setflag ["\\Flagged"];}
```
#### 2. Subject chứa "sach" => đánh dấu đã xem
```
if allof (header :contains "Subject" "sach")
{setflag ["\\Seen"];}
```
#### 3. Mail gửi từ dinhthiha1011@gmail.com => mail được lưu trong file INBOX.AHa
```
if allof (header :contains "from" "dinhthiha1011@gmail.com")
{fileinto :create "INBOX.AHa";}
```
#### 4. auto fw sang test2@anthanh264.site
```
if true {redirect:copy "test2@anthanh264.site";}
```
#### 5. Mail gửi từ anthanh264@gmail.com => fw sang hadt@pikab.in
```
if allof (header :contains "from" "anthanh264@gmail.com")
 {redirect :copy "hadt@pikab.in";}`
```
