- Sử dụng spamtest và virustest extensions RFC 5235, ngôn ngữ Sieve cung cấp một giao diện lệnh thống nhất và được tiêu chuẩn hóa để đánh giá các bài kiểm tra spam và virus được thực hiện trên message
- Người dùng không còn cần biết những header nào cần được kiểm tra và phán quyết của scanner trong giá trị field header. Họ chỉ cần biết sử dụng extension `spamtest` (`spamtestplus`) và virustest như thế nào
- Điều này cũng cung cấp cho các editors Sieve dựa trên GUI-based để cung cấp giao diện di dộng và dễ cài đặt cho cấu hình bộ lọc spam và virus
### Configuration
- `spamtest`, `spamtestplus`, `virustest` extensions không enable theo mặc định, do đó cần enabled rõ ràng cài đặt sử dụng `sieve_extensions`
- Cho phép setting cần config cho sử dụng `spamtest` và `spamtestplus` extensions
- `virustest` extension có cài đặt cấu hình giống hệt nhau, nhưng với một `sieve_virustest_prefix` thay vì `sieve_spamtest_prefix`:
`sieve_spamtest_status_type = "score" / "strlen" / "text"`
  - Đây là loại cụ thể của trạng thái result mà trình quét spam/virus tạo ra.
  - Điều này có thể là một điểm số (score), một chuỗi các ký tự giống hệt nhau (strlen), ví dụ `*******`, hoặc một văn bản miêu tả (text), ví dụ {{{Spam}}} hoặc Not Spam
`sieve_spamtest_status_header = <header-field> [ ":" <regexp> ]`
  - Field này chỉ định trường tiêu đề chứa thông tin kết quả của trình quét spam và nó có thể diễn đạt syntax nội dung của header. Nó không match header được tìm trong message, dòng lệnh spamtest sẽ match với '0'
  - Đây là một cài đặt structured. Phần đầu tiên chỉ định tên header field
  - Tùy chọn, một biểu thức chính quy posix theo sau tên field header, cách nhau bởi dấu hai chấm ':'. Bất kì khoảng trắng nào ngay sau dấu 2 chấm không phải là một phần của biểu thức chính quy. Nếu biểu thức chính quy bị bỏ qua, mọi nội dung  header sẽ được chấp nhận và giá trị header đầy đủ được sử dụng
  - Khi một biểu thức chính quy được sử dụng, nó phải chỉ định một giá trị match (trong ngoặc) mang lại kết quả test spam mong muốn
  - Nếu header không match với biểu thức chính quy hoặc nó không match với giá trị nào được tìm thấy, test `spamtest` sẽ match với 0 trong quá trình thực thi Sieve script
`sieve_spanmtest_max_value =`
- Điều này chỉ định tĩnh giá trị tối đa mà điểm số spam có thể có
`sieve_spamtest_max_header = <header-field> [":" <regexp> ]
- Một vài spam scanners bao gồm giá trị điểm max trong một trong các status header của chúng ta. Sử dụng setting này, maximum này có thể được trích xúât từ chính thông báo thay vì chỉ định số tối đa theo cách thủ công bằng sử dụng cài đặt `sieve_spamtest_max_value` ở trên
- Cú pháp này giống với cài đặt `sieve_spamtest_status_header`
`sieve_spamtest_text_valueX =`
- Khi cài đặt `sieve_spamtest_status_type` được cài là `text`, các cài đặt này chỉ định rằng `spamtest` sẽ khớp với giá trị `X` khi chuỗi được chỉ định bằng với văn bản (được trích xuất) từ tiêu đề trạng thái. Đối với spamtest và spamtestplus, các giá trị của x từ 0-10 được nhận dạng, trong khi đó virustest chỉ sử dụng giá trị giữa 0 và 5
### Example
- Phần này hiển thị một số ví dụ về config. Mỗi ví dụ hiển thị một mẫu header để kiểm tra virus/spam hợp lệ mà config đã cho sẽ hoạt động trên đó
#### Example 1
`Spam header: X-Spam-Score: No, score=-3.2`
```
plugin {
  sieve_extensions = +spamtest +spamtestplus

  sieve_spamtest_status_type = score
  sieve_spamtest_status_header = \
    X-Spam-Score: [[:alnum:]]+, score=(-?[[:digit:]]+\.[[:digit:]])
  sieve_spamtest_max_value = 5.0
}
```
#### Example 2
`Spam header: X-Spam-Status: Yes`
```
plugin {
  sieve_extensions = +spamtest +spamtestplus

  sieve_spamtest_status_type = text
  sieve_spamtest_status_header = X-Spam-Status
  sieve_spamtest_text_value1 = No
  sieve_spamtest_text_value10 = Yes
}
```
#### Example 3
`Spam header: X-Spam-Score: sssssss`
```
plugin {
  sieve_extensions = +spamtest +spamtestplus

  sieve_spamtest_status_header = X-Spam-Score
  sieve_spamtest_status_type = strlen
  sieve_spamtest_max_value = 5
}
```
#### Example 4
`Spam header: X-Spam-Score: status=3.2 required=5.0`

`Virus header: X-Virus-Scan: Found to be clean`
```
plugin {
  sieve_extensions = +spamtest +spamtestplus +virustest

  sieve_spamtest_status_type = score
  sieve_spamtest_status_header = \
    X-Spam-Score: score=(-?[[:digit:]]+\.[[:digit:]]).*
  sieve_spamtest_max_header = \
   X-Spam-Score: score=-?[[:digit:]]+\.[[:digit:]] required=([[:digit:]]+\.[[:digit:]])

  sieve_virustest_status_type = text
  sieve_virustest_status_header = X-Virus-Scan: Found to be (.+)\.
  sieve_virustest_text_value1 = clean
  sieve_virustest_text_value5 = infected
}
```
### My Config
#### Spamassassin
- Install 
```
sudo apt-get update
sudo apt-get install spamassassin spamc -y
```
- Add a SpamAssassin user and disable the login
```
sudo adduser spamd --disabled-login
```
- Edit file config 
```
sudo nano /etc/default/spamassassin
```
- Add line ENABLED=0
- Find the line: `OPTIONS="--create-prefs .....` Change it to
```
OPTIONS="--create-prefs --max-children 5 --username spamd --helper-home-dir /home/spamd/ -s /home/spamd/spamd.log"
```
- Find the line `CRON=0` change the value from 0 to 1 `CRON=1`
- Backup and create new file
```
sudo mv /etc/spamassassin/local.cf  /etc/spamassassin/local.cf.bk
sudo vim /etc/spamassassin/local.cf 
```
- Add content into file
```
rewrite_header Subject ***** SPAM _SCORE_ *****

report_safe             0

required_score          5.0

use_bayes               1

use_bayes_rules         1

bayes_auto_learn        1

skip_rbl_checks         0

use_razor2              0

use_dcc                 0

use_pyzor               0

ifplugin Mail::SpamAssassin::Plugin::Shortcircuit

endif
```
- Edit file master postfix
```
sudo vim /etc/postfix/master.cf
```
- Locate these entries:
`smtp      inet  n       -       y       -       -       smtpd`
- Add this line below this `-o content_filter=spamassassin`
![](https://hackmd.io/_uploads/BJo_xwuBh.png)
This line is indented by one space compared to the smtp line, o parameter is displayed in blue
- And also add at the end of the file
```
spamassassin unix -     n       n       -       -       pipe

user=spamd argv=/usr/bin/spamc -f -e  

/usr/sbin/sendmail -oi -f ${sender} ${recipient}
```
![](https://hackmd.io/_uploads/HkMpgwOHh.png)
- Restart postfix and enable Spamassassin
```
sudo systemctl restart postfix.service
postfix reload
sudo systemctl enable spamassassin.service
sudo systemctl start spamassassin.service
```
- Test Spamassassin: Send email with content
```
XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X
```
#### Config 
- Thêm plugin
`vim /etc/dovecot/conf.d/90-sieve.conf`
```
   sieve_extensions = +spamtest +spamtestplus

  sieve_spamtest_status_type = score
  sieve_spamtest_status_header = \
    X-Spam-Score: [[:alnum:]]+, score=(-?[[:digit:]]+\.[[:digit:]])
  sieve_spamtest_max_value = 5.0
```
- Create and config file `vim /sieve/spam.sieve`
```
require ["fileinto", "spamtest"];
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
require ["relational", "comparator-i;ascii-numeric"];
if allof (
spamtest :value "le" :comparator "i;ascii-numeric" "2"
 #if x-spam-core <2 => mailbox SPAM (Just for test) 
 ) {
 fileinto :create "INBOX.Spam";
 }
 #
```
- `vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "spam";
```
```
sievec /sieve/spam.sieve
systemctl restart dovecot
```
