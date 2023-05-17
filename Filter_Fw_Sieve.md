```
require [ "imap4flags","fileinto","mailbox","copy",...];
```
### Một vài config sieve thông dụng
#### 1. Subject chứa "hi" => đánh dấu sao
```
require [ "imap4flags"]
if allof (header :contains "Subject" "hi")
{setflag ["\\Flagged"];}
```
#### 2. Subject chứa "sach" => đánh dấu đã xem
```
require ["imap4flags"]
if allof (header :contains "Subject" "sach")
{setflag ["\\Seen"];}
```
#### 3. Mail gửi từ dinhthiha1011@gmail.com => mail được lưu trong file INBOX.AHa
```
require ["fileinto","mailbox"]
if allof (header :contains "from" "dinhthiha1011@gmail.com")
{fileinto :create "INBOX.AHa";}
```
#### 4. auto fw sang test2@anthanh264.site
```
require ["copy"]
if true {redirect:copy "test2@anthanh264.site";}
```
#### 5. Mail gửi từ anthanh264@gmail.com => fw sang hadt@pikab.in
```
require ["copy"]
if allof (header :contains "from" "anthanh264@gmail.com")
 {redirect :copy "hadt@pikab.in";}`
```
#### 6. Mail có chứa file => đánh dấu sao
```
require ["imap4flag","body","regex"]
if allof (anyof (body :raw :regex ["X-Attachment-Id:"], body :raw :regex ["Content-Disposition: attachment;"]))
{setflag ["\\Flagged"];}
```
#### 7. Mail gửi từ dinhthiha1011@gmail.com và chứa file => fw sang tbm35@dinhha.online
```
require ["copy","body","regex"]
if allof (header :contain "from" "dinhthiha1011@gmail.com", anyof (body :raw :regex ["X-Attachment-Id:"], body :raw :regex ["Content-Disposition: attachment;"]))
{redirect :copy "tbm35@dinhha.online";}
```
#### 8. Mail gửi từ dinhtrongniem@gmail.com, subject chứa "HeaderN", body chứa "bodyNiem", body không chứa "is", chứa file => mail được lưu trong file INBOX.ANiem
```
require ["fileinto", "body", "regex"]
if allof (header :contains "from" "dinhtrongniem@gmail.com", header :contains "subject" "HeaderN", body :text :contains "bodyNiem", not body :text :contains "is", anyof (body :raw :regex ["X-Attachment-Id:"], body :raw :regex ["Content-Disposition: attachment;"])) 
{fileinto :create "INBOX.ANiem";} 
```
#### 9. Gửi mail có size trên 1M => đánh dấu sao
```
require ["imap4flag"]
if allof (size :over 1M)
{setflag ["\\Flagged"];}
```
#### 10. Mail có size dưới 4M => loại bỏ mail
```
if allof (size :under 4M)
{discard;}
```
