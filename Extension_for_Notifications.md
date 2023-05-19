- Extension sieve `enotify` RFC 5435 adds hành động notify vào ngôn ngữ Sieve
### Configuration
`sieve_notify_mailto_envelope_from setting`
  - Điều này cho phép config source của địa chỉ notification người gửi cho e-mail notifications
  - Điều này tương tự như cài đặt `sieve_redirect_envelope_from` để chuyển hướng
### Example
#### 1. Send notifications với importance levels khác
```
require ["enotify", "fileinto", "variables"];
if header :contains "from" "boss@example.org" {
  notify: importance "1"
    :message "This is probably very important" "mailto:alm@example.com";
  # Don't send any further notifications
  stop;
}

if header :contains "to" "sievemailinglist@example.org" {
  # :matches is used to get the value of the Subject header
  if header :matches "Subject" "*" {
    set "subject" "${1}";
  }
  
  # :matches is used to get the value of the From header
  if header :matches "From" "*" {
    set "from" "${1}";
  }
  
  notify :importance "3"
    :message "[SIEVE] ${from}: ${subject}"
    "mailto:alm@example.com";
  fileinto "INBOX.sieve";
}
```
#### 2. Send notification if we receive mail from domain
```
require ["enotify", "fileinto", "variables", "envelope"];
if header :matches "from" "*@*.examole.org" {
  # :matches is used to get the MAIL FROM address
  if envelope :all :matches "from" "*" {
    set "env_from" "[really: ${1}]";
  }
  
  # :matches is used to get the value of the Subject header
  if header :matches "Subject" "*" {
    set "from_addr" "${1}";
  }
  notify :message "${from_addr}$(env_from): ${subject}" "mailto:alm@example.com";
}
```
### My Config
```
vim /etc/dovecot/sieve/before.sieve
```
#### 1. Set mail quan trọng nhất
- Gửi từ hadt@bizflycloud.vn tới domain dinhha.online=> set thông báo quan trọng nhất, auto fw quan trọng sang ha@dinhha.online
```
require ["enotify","imap4flags", "fileinto","mailbox", "variables"];
#if header :contains "from" "hadt@bizflycloud.vn" {
# notify :importance "1"
#  :message "This is probably very important"
#   "mailto:ha@dinhha.online";
# stop;
#}
```
#### 2. Set thứ tự quan trọng (thứ 3)
- Gửi tới ah@dinhha.online => ah@dinhha.online set flags, ha@dinhha.online nhận một message theo định dạng
```
require ["enotify","imap4flags", "fileinto","mailbox", "variables"];
if header :contains "to" "ah@dinhha.online" {
 if header :matches "Subject" "*" {
  set "subject" "${1}";
 }
 if header :matches "From" "*" {
  set "from" "${1}";
 }
 notify :importance "3"
  :message "[SIEVE] ${from}: ${subject}"
  "mailto:ha@dinhha.online";
 setflag ["\\Flagged"];
}
```
#### 3. Set từ domain
```
require ["enotify","envelope","imap4flags", "fileinto","mailbox", "variables"];
if header :matches "from" "*@*bizflycloud.vn" {
 if envelope :all :matches "from" "*" {
  set "env_from" "[really: ${1}]";
 }
if header :matches "Subject" "*" {
 set "subject" "${1}";
}
if address :matches :all "from" "*" {
 set "from_addr" "${1}";
}
notify :message "${from_addr}${env_from}: ${subject}"
 "mailto:ah@dinhha.online";
}
```
