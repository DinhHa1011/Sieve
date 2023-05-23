- Sieve variables extension RFC 5229 thêm concept của variables to the Sieve language
### Configuration
- Variables extension thì mặc định có sẵn
- Variables extension có cài đặt cụ thể của riêng nó
- Cài đặt dưới đây có thể config cho variables extension (giá trị mặc định được biểu thị):
`sieve_variables_max_scope_size=255`
  - Số max của variables có thể được khai báo trong một phạm vi
  - Hiện tại có 2 phạm vi biến: phạm vi script thông thường và phạm vi global được tạo bởi "include" extension
  - Giá trị min của cài đặt này là 128
`sieve_variables_max_variable_size=4k`
  - Max cho phép size cho giá trị của một biến
  - Nếu vượt quá trong thời gian chạy, giá trị luôn được cắt bớt ở mức tố đa được config
  - Giá trị tối thiểu cho cài đặt này là 4000 byte
  - Giá trị tính bằng byte (trừ khi theo sau là k(ilo))
### My Config
`vim /sieve/variables.sieve`
```
require ["fileinto","variables", "mailbox", "encoded-character"];
set "i" "ID";
set "s" "sj";
set "b" "bd";
if allof (header :matches "List-ID" "*<*@*")
{fileinto :create "INBOX.${i}";}
if allof (header :contains "Subject" "hi")
{fileinto :create "INBOX.${s}";}
if allof (header :contains "body" "hehe")
{fileinto :create "INBOX.${b}";}
```
```
sievec /sieve/variables.sieve
```
`vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "variables";
```
