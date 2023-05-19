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
- vim /etc/dovecot/sieve/before.sieve

```# Require
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate"];

# Send notifications with different importance levels
if header :contains "from" "ah@dinhha.online" { # nếu header có from là ah@dinhha.online
 notify :importance "1" # Set thông báo quan trọng = 1 => Cao nhất
  :message "This is probably very important" # Nội dung thông báo: Cái mail này quan trọng vc
   "mailto:test1@anthanh264.site"; # Gửi tới test1@anthanh264.site
 stop; # Dừng 
}
if header :contains "to" "test2@anthanh264.site" { # Nếu header có to là test2@anthanh264.site
 if header :matches "Subject" "*" { # Matches để lấy subject của mail 
  set "subject" "${1}"; # Hiểu câu này kiểu lưu cái subject vừa lấy vào biến subject 
 }
 if header :matches "From" "*" { # Matches để lấy From của mail 
  set "from" "${1}"; # Hiểu câu này kiểu lưu cái from vừa lấy vào biến from 
 }
 notify :importance "3"  # Set thông báo quan trọng = 3 => Thấp nhất
  :message "[SIEVE] ${from}: ${subject}" # Nội dung thông báo: [SIEVE] biến from : biến subject 
  "mailto:test1@anthanh264.site"; # Gửi tới test1@anthanh264.site
 fileinto :create "INBOX.sieve"; # Cho vào mailbox sieve của test2@anthanh264.site 
}

# Send notification if we receive mail from domain
if header :matches "from" "*@*dinhha.online" { # nếu mà header có đuôi sau @ là dinhha.online thì lấy hết phần from 
    # :matches is used to get the MAIL FROM address
    if envelope :all :matches "from" "*" {
        set "env_from" " [really: ${1}]"; #set biến envfrom = really: from vừa lấy 
    }

    # :matches is used to get the value of the Subject header
    if header :matches "Subject" "*" {
        set "subject" "${1}"; #set biến subject = subject vừa lấy
    }

    # :matches is used to get the address from the From header
    if address :matches :all "from" "*" {
        set "from_addr" "${1}";
    }

    notify :message "${from_addr}${env_from}: ${subject}"
                    "mailto:test1@anthanh264.site";
}
```
