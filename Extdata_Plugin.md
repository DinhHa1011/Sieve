- Extdata plugin thêm vnd.dovecot.extdata extension tới Sieve language
- Nó cho phép một Sieve script lookup thông tin từ một datasource bên ngoài từ script
- Điều này sử dụng cơ chế dict của Dovecot trong một read-only manner, điều này có nghĩa là script không update dict data source
### Getting the sources 
- Hiện tại, source của extdata plugin chưa được phát hành, nhưng bạn có thể lấy chúng từ Git repo của họ
```
git clone -b core-0.5 https://github.com/stephanbosch/sieve-extdata-plugin.git
```
### Compiling
- Nếu bạn dowload sources của plugin này sử dụng Git, bạn sẽ cần thực thi ./autogen.sh đầu tiên để build automake struture trong source tree của bạn
- Quy trình này yêu cầu phải cài đặt autotools và libtool để installed
- Nếu bạn install Dovecot từ sources, script configure của plugin sẽ có thể tự động tìm dovecot-config đã cài đặt, cùng với các header phát triển Pigeonhole
```
./configure
make
sudo make install
```
### Configuration 
- Gói này builds và install sieve_extdata plugin cho Pigeonhole Sieve
- Plugin hoạt động bằng cách add nó trong cài đặt sieve_plugins
```
sieve_plugins = sieve_extdata
```
- Các cài đặt cấu hình sau được sử dụng
### Settings
`sieve_extdata_dict_uri`
- default: <empty>
- Values: String
- Chỉ định URI của dict được sử dụng để tra cứu extdata
- Example
```
plugin {
  sieve = ~/.dovecot.sieve
  sieve_plugins = sieve_extdata

  sieve_extdata_dict_uri = file:/etc/dovecot/pigeonhole-sieve.dict
}
```
### Usage
- Sieve scripts có thể sử dụng extension mới `vnd.dovecot.extdata` dưới đây:
```
require ["variables", "vacation", "vnd.dovecot.extdata"];

vacation :days 30 :subject "${extdata.vacation_subject}" "${extdata.vacation_message}";
keep;
```
- `priv/vacation_subject` và `priv/vacation_message` sẽ được tra cứu trong dovecot dict. Xem bên dưới để biết một số ví dụ
#### Dict with flat file backend
- Để sử dụng flat file backend cho ví dụ trên, tạo một dict file với format dưới đây (cho ví dụ /etc/dovecot/sieve-extdata-lookup.dict):
```
priv/vacation_message
Sorry I am out of the office
```
#### Dict với SQL backend
- Để sử dụng SQL backend cho ví dụ trên, đầu tiên set up một dict proxy trong /etc/dovecot.conf:
```
dict {
    sieve = mysql:/etc/dovecot/pigeonhole-sieve.dict
}
```
- Và trong /etc/dovecot/pigeonhole-sieve.dict:
```
connect = host=localhost dbname=dovecot user=dovecot password=password

map {
  pattern = priv/vacation_message   # The dict value to lookup
  table = virtual_users             # The SQL table to perform the lookup in
  username_field = email            # The username field to search on in the table
  value_field = vacation_msg        # The database value to return
}
```
- Cuối cùng config extdata để sử dụng proxy:
```
sieve_extdata_dict_uri = proxy::sieve
```
### My Config
`vim /etc/dovecot/conf.d/90-sieve.conf`
```
plugin {

  sieve = file:%h/sieve;active=%h/.dovecot.sieve
  sieve_global = /sieve
  sieve_before = /etc/dovecot/sieve/before.sieve
  sieve_extensions = +notify +imapflags +editheader
  sieve_extensions = +vacation-seconds
  sieve_vacation_min_period = 0
  sieve_vacation_default_period = 1d
  sieve_vacation_max_period = 2d
  sieve_trace_debug = yes
  sieve_plugins = sieve_extdata
  sieve_pipe_bin_dir = /etc/dovecot/conf.d/custom-pipe
  sieve_extensions = +vnd.dovecot.extdata
  sieve_extdata_dict_uri = proxy::sieve
  sieve_extdata_dict_uri = file:/etc/dovecot/pigeonhole-sieve.dict
}
```
- Git clone
```
git clone -b core-0.5 https://github.com/stephanbosch/sieve-extdata-plugin.git
```
```
apt install autoconf libtool dovecot-dev automake
cd sieve-extdata-plugin/
./autogen.sh
./configure --with-dovecot=/usr/lib/dovecot --with-pigeonhole=/usr/include/dovecot/sieve
make
make install
ln -s /usr/local/lib/dovecot/sieve/lib90_sieve_extdata_plugin.so /usr/lib/dovecot/modules/sieve/lib90_sieve_extdata_plugin.so
```
`vim /etc/dovecot/dovecot.conf`
```
!include_try /usr/share/dovecot/protocols.d/*.protocol
protocols = imap pop3 lmtp
listen = *, ::
#mail_debug = no
#auth_verbose = yes
#auth_debug = no
#auth_debug_passwords = no
#auth_mechanisms = plain cram-md5
#dict {
  #quota = mysql:/etc/dovecot/dovecot-dict-sql.conf.ext
  #expire = sqlite:/etc/dovecot/dovecot-dict-sql.conf.ext
#}

!include_try local.conf
!include conf.d/*.conf
```
`vim /etc/dovecot/sieve-notify.conf`
```
connect = host=127.0.0.1 port=5432 dbname=vmail user=vmail password=пароль
map {
    pattern = priv/notify_email
    table = mailbox
    username_field = username
    value_field = notify_email
}
```
`vim /etc/dovecot/sieve-extdata-lookup.dict`
```
priv/vacation_message
Sorry I am out of the office
```
`vim /etc/dovecot.conf`
```
dict {
    sieve = mysql:/etc/dovecot/pigeonhole-sieve.dict
}
```
`vim /etc/dovecot/pigeonhole-sieve.dict`
```
connect = host=localhost dbname=dovecot user=dovecot password=password

map {
  pattern = priv/vacation_message   # The dict value to lookup
  table = virtual_users             # The SQL table to perform the lookup in
  username_field = email            # The username field to search on in the table
  value_field = vacation_msg        # The database value to return
}
```
`vim /sieve/extdata.sieve`
```
require ["fileinto","mailbox", "variables", "vnd.dovecot.extdata"];

   if allof (header :is "X-Spam-Status" "Yes",
     extdata :is "discard_spam" "yes") {
     discard;
   } else {
     fileinto "INBOX.Spam";
   }
```
```
sievec /sieve/extdata.sieve
```
`vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "extdata";
```
```
sievec /etc/dovecot/sieve/before.sieve
systemctl restart dovecot
```

