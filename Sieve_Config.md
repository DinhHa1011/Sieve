## Mail server running dovecot LMTP
```
vim /etc/dovecot/conf.d/10-master.conf
```
#### edit:
```
service lmtp {
  #unix_listener lmtp {
  unix_listener /var/spool/postfix/private/dovecot-lmtp {
    mode = 0600
    user = postfix
    group = postfix
    #mode = 0666
  }

  # Create inet listener only if you can't use the above UNIX socket
  inet_listener lmtp {
    # Avoid making LMTP visible for the entire internet
    address = 127.0.0.1
    port = 24
  }
}
```
#### restart dovecot
```
systemctl restart dovecot
```
#### check:
```
lsof -i :24
```
#### result OK:
![](https://hackmd.io/_uploads/r1qThRxrn.png)

## Dovecot Install ManageSieved
```
apt-get install dovecot-sieve dovecot-managesieved
```
## Basic config
#### LMTP Config
- Để sử dụng Sieve, cần enable Pigeonhole Sieve plugin trong config LMTP
```
vim /etc/dovecot/conf.d/20-lmtp.conf
```
- Add sieve to mail_plugins trong LMTP protocol section:
```
protocol lmtp {
  mail_plugin = $mail_plugins sieve
}
```
#### sieve config
```
vim /etc/dovecot/conf.d/90-sieve.conf
```
- Change the sieve parameter
from: `sieve = file:~/sieve;active=~/.dovecot.sieve`
to: `sieve = file:%h/sieve;active=%h/.dovecot.sieve`
- Tạo một sieve script 
```
mkdir /etc/dovecot/sieve
```
- edit sieve
```
vim /etc/dovecot/sieve/before.sieve
```
```
#header la hi thi set danh dau sao
require [ "imap4flags","fileinto","mailbox"];
if allof (header :contains "Subject" "hi")
{setflag ["\\Flagged"];}
```
#### Complile Script File
- Tự sinh ra một file before.svbin
```
cd /etc/dovecot/sieve
sievec before.sieve
```
#### Enable Script
```
vim /etc/dovecot/conf.d/90-sieve.conf
```
- Thay đổi:
from: `#sieve_before = /var/lib/dovecot/sieve.d/`
to: `sieve_before = /etc/dovecot/sieve/before.sieve`

#### Restart dovecot
```
systemctl restart dovecot
```

## Test
- Note: Tạo folder (tạo trên sieve rồi vào bật trên webmail roundcube: http://domain/mail/)
