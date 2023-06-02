- "sieve_extprograms" plugin cung cấp một extension cho ngôn ngữ Sieve filtering thêm dòng lệnh action mới để gọi một bộ chương trình bên ngoài được xác định trước
- Message có thể được chuyển đến hoặc lọc qua các chương trình và dữ liệu chuỗi có thể được nhập và truy xuất từ các chương trình đó
- Để giảm thiểu các lo ngại về bảo mật, các chương trình bên ngoài không được chọn tùy ý; các chương trình khả dụng bị hạn chế thông qua cấu hình admin
- Plugin này chỉ có sẵn cho Pigeonhole v0.3 trở lên và cao hơn (có sẵn cho Dovecot v2.1) 
- Đối với Pigeonhole v0.4. plugin này là một phần của bản phát hành. Đây là một sự phát triển của Pipe plugin cho Pigeonhole v0.2 và hiện cung cấp các `filter` và dòng lệnh `execute` (và các phần mở rộng tương ứng) ngoài lệnh `pipe` đã được cung cấp trước đó bởi pipe
### Configuration
- Plugin được hoạt động bằng cách thêm nó vào setting `sieve_plugins`
```
sieve_plugin = sieve_extprograms
```
- Plugin này đăng ký `vnd.dovecot.pipe`, `vnd.dovecot.filter` và `vnd.dovecot.execute` extension với thông dịch viên Sieve. 
- Tuy nhiên, extension đó thì không enabled theo mặc định và do đó cần enabled rõ ràng. Nó thì được recommend để hạn chế việc sử dụng các extension này trong ngữ cảnh global bằng cách thêm chúng vào cài đặt `sieve_global_extensions`
- Nếu các user script cá nhân cũng cần truy cập trực tiếp vào các chương trình bên ngoài, extension cần được thêm vào cài đặt `sieve_extensions`
- Các lệnh được giới thiệu bởi extension ngôn ngữ Sievetrong plugin này có thể chuyển trược tiếp một message hoặc chuỗi dữ liệu sang 1 chương trình bên ngoài (thường là shell script) bằng cách rẽ nhánh một process mới
- Cách khác, chúng ta có thể kết nối một socket unix mà dịch vụ tập lệnh dovecot đang lắng nghe để bắt đầu chương trình bên ngoài, ví dụ: để thực thi với tư cách người dùng khác hoặc để tăng cường bảo mật
- Tên chương trình được chỉ định cho các lệnh pipe, filter, execute mới được sử dụng để tìm chương trình hoặc socket trong một thư mục được định cấu hình
- Các thư mục riêng biệt được chỉ định cho các socket và các file nhị phân được thực thi trực tiếp, thư mục socket được tìm kiếm đầu tiên. Vì sử dụng "/" trong tên chương trình bị cấm nên không thể xây dựng cấu trúc phân cấp
- Các chương trình rẽ nhánh trực tiếp được thực thi với một tập hợp giới hạn cacsbieesn môi trường: `HOME`, `USE`, `SENDER`, `RECIPIENT` và `ORIG_RECIPIENT`
- Các chương trình được thực thi thông qua dịch vụ socket script-pipe hiện không có môi trường nào được cài đặt
- Nếu một shell script được thực thi để đọc một message họăc chuỗi dữ liệu, nó phải đọc đầy đủ đầu vào được cung cấp cho đến khi dữ liệu kết thúc bằng EOF, nếu không hành động sieve gọi chương trình sẽ thất bại. Hoạt động cũng sẽ fail khi shell script trả về một nonzero exit code
- Đầu ra tiêu chuẩn có sẵn để trả về một thông báo (đỗi với lệnh filter) hoặc chuỗi dữ liệu (đối với lệnh thực thi) cho trình thông dịch Sieve. Lỗi tiêu chuẩn được ghi vào file log LDA
- Ba extension được giới thiệu trong plugin này - `vnd.dovecot.pipe`, `vnd.dovecot.filter` và `vnd.dovecot.execute` - mỗi cái cấu hình riêng biệt nhưng giống nhau
- Các cài đặt cấu hình sau được sử dụng, "<extension>" trong cài đặt tên được thay thế bởi `pipe`, `filter`, `execute` tùy thuộc vào extension nào đang được config:
  
`sieve_extension_socket_dir=`
  
  - Trỏ đến một thư mục liên quan tới Dovecot base_dir nơi mà plugin tìm socket dịch vụ script
  
`sieve_extension_bin_dir=`
  
  - Trỏ đến thư mục mà plugin tìm chương trình (shell scripts) để thực thi trực tiếp và chuyển message tới
  
`sieve_extension_exec_timeout = 10s`
  
  - Config thời gian thực thi max sau đó chương trình bị buộc chấm dứt
  
`sieve_extension_input_eol = crlf`
  
  - Xác định chuỗi ký tự end-of-line được sử dụng cho dữ liệu được dẫn đến các chương trình bên ngoài
  - Theo mặc định `crlf` hiện nay, đại diện cho một chuỗi các ký tự carriage return (CR) và line feed (LF)
  - Điều này phù hợp với format tin nhắn Internet Message (RFC 5322) và bản thân Sieve sử dụng làm kết thúc dòng. Đặt cái này thành "lf" để sử dụng một ký tự LF thay thế
  
#### Configuration Example 1: socket service cho "pipe" và "execute"
```
plugin {
  sieve = ~/.dovecot.sieve

  sieve_plugins = sieve_extprograms
  sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.execute

  # pipe sockets in /var/run/dovecot/sieve-pipe
  sieve_pipe_socket_dir = sieve-pipe

  # execute sockets in /var/run/dovecot/sieve-execute
  sieve_execute_socket_dir = sieve-execute
}

service sieve-pipe-script {
  # This script is executed for each service connection
  executable = script /usr/lib/dovecot/sieve-extprograms/sieve-pipe-action.sh

  # use some unprivileged user for execution
  user = dovenull

  # socket name is program-name in Sieve (without sieve-pipe/ prefix)
  unix_listener sieve-pipe/sieve-pipe-script {
  }
}

service sieve-execute-action {
  # This script is executed for each service connection
  executable = script /usr/lib/dovecot/sieve-extprograms/sieve-execute-action.sh

  # use some unprivileged user for execution
  user = dovenull

  # socket name is program-name in Sieve (without sieve-execute/ prefix)
  unix_listener sieve-execute/sieve-execute-action {
  }
}
```
#### Configuration Example 2: direct execution for "pipe" và "filter"
```
plugin {
  sieve = ~/.dovecot.sieve

  sieve_plugins = sieve_extprograms
  sieve_global_extensions = +vnd.dovecot.pipe +vnd.dovecot.filter

  # This directory contains the scripts that are available for the pipe command.
  sieve_pipe_bin_dir = /usr/lib/dovecot/sieve-pipe

  # This directory contains the scripts that are available for the filter
  # command.
  sieve_filter_bin_dir = /usr/lib/dovecot/sieve-filter
}
```
### Usage
### Full Examples
#### Example 1
  - Ví dụ đơn giản này cho thấy cách sử dụng "vnd.dovecot.execute" extension để thực hiện một số loại kiểm tra đối với message đến
  - Relevant configuration:
  ```
  plugin {
    sieve_extensions = +vnd.dovecot.execute

    sieve_plugins = sieve_extprograms
    sieve_execute_bin_dir = /usr/lib/dovecot/sieve-execute
  }
  ```
  - The sieve script:
  ```
  require "vnd.dovecot.execute";
  if not execute :pipe "hasfrop.sh" {
    discard;
    stop;
  }
  ```
  - Tại vị trí `usr/lib/dovecot/sieve-execute`, tạo một executable script `hasfrop.sh`
  - Trong ví dụ này, hasfrop.sh kiểm tra liệu message có chứa văn bản chữ "FROP" ở bất kỳ đâu trong message hay không. 
  - Sieve script hiển thị ở trên sẽ loại bỏ thông báo nếu script này kết thúc bằng mã exit khác 0, điều này xảy ra khi tìm thấy "FROP"
  ```
  # Something that reads the whole message and inspects it for some
  # property. Not that the whole message needs to be read from input!
  N=`cat | grep -i "FROP"` # Check it for the undesirable text "FROP"
  if [ ! -z "$N" ]; then
         # Result: deny
         exit 1;
  fi

  # Result: accept
  exit 0
  ```
  #### Example 2
  - Ví dụ này cho thấy cách sử dụng `vnd.dovecot.execute` extension cho querying/updating một MySQL database
  - Điều này được sử dụng để chuyển hướng message cứ sau 300s cho một người gửi cụ thể. 
  - Chú ý rằng trường hợp sử dụng cụ thể này cũng có thể được triển khai bằng cách sử dụng "duplicate" extension
  - Relavant configuration:
  ```
  plugin {
    sieve_extensions = +vnd.dovecot.execute

    sieve_plugins = sieve_extprograms
    sieve_execute_bin_dir = /usr/lib/dovecot/sieve-execute
  }
  ```
  - The sieve script: 
  ```
  require ["variables", "copy", "envelope", "vnd.dovecot.execute"];

  # put the envelope-from address in a variable
  if envelope :matches "from" "*" { set "from" "${1}"; }

  # execute the vacationcheck.sh program and redirect the message based on its exit code
  if execute :output "vacation_message" "vacationcheck.sh" ["${from}","300"]
  {
  redirect
        :copy "foo@bar.net";
  }
  ```
  - Tại vị trí `/usr/lib/dovecot/sieve-execute`, tạo script executable `vacationcheck.sh`
  - Sript này login vào mysql và thực thi lệnh select giá trị từ database postfixadmin. Đếm giá trị mail trong khoảng thời gian bé hơn 300s, nếu giá trị trả về là 0 thì gửi mail và ngược lại
  - Trong ví dụ này, script `vacationcheck.sh` cần 2 thông số: địa chỉ người gửi và khoảng thời gian được chỉ định tính bằng giây (s). Khoảng thời gian được sử dụng để chỉ định khoảng thời gian tối thiểu cần phải trôi qua kể từ khi người gửi được nhìn thấy lần cuối
  - Nếu script trả về mã exit 0, sau đó message được chuyển hướng trong Sieve script được hiển thị
  ```
  USER=postfixadmin
  PASS=pass
  DATABASE=postfixadmin

  # DB STRUCTURE
  #CREATE TABLE `sieve_count` (
  #  `from_address` varchar(254) NOT NULL,
  #  `date` datetime NOT NULL
  #) ENGINE=InnoDB DEFAULT CHARSET=latin1;
  #
  #ALTER TABLE `sieve_count`
  # ADD KEY `from_address` (`from_address`);

  MAILS=$(mysql -u$USER -p$PASS $DATABASE --batch --silent -e "SELECT count(*) as ile FROM sieve_count WHERE from_address='$1' AND DATE_SUB(now(),INTERVAL $2 SECOND) < date;")
  ADDRESULT=$(mysql -u$USER -p$PASS $DATABASE --batch --silent -e "INSERT INTO sieve_count (from_address, date) VALUES ('$1', NOW());")

  # uncoment below to debug
  # echo User $1 sent $MAILS in last $2 s >> /usr/lib/dovecot/sieve-pipe/output.txt
  # echo Add result : $ADDRESULT >> /usr/lib/dovecot/sieve-pipe/output.txt
  # echo $MAILS

  if [ "$MAILS" = "0" ]
  then
  exit 0
  fi

  exit 1
  ```
### My Config
#### Example 1: 
- Chạy 1 bash script đọc nội dung thư, nếu có từ FROP thì discard không thì gửi như bthg
- Thêm bash script
`vim /usr/lib/dovecot/sieve-execute/hasfrop.sh`
```
N=`cat | grep -i "FROP"` # Check it for the undesirable text "FROP"
if [ ! -z "$N" ]; then
               # Result: deny
               exit 1;
fi

# Result: accept
exit 0
```
- config file sieve include
`vim /sieve/extpr.sieve`
```
require "vnd.dovecot.execute";
if not execute :pipe "hasfrop.sh" {
  discard;
  stop;
}
```
`vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "extprogram";
```
```
sievec /sieve/extprogram.sieve
sievec /etc/dovecot/sieve/before.sieve
systemctl restart dovecot
```
#### Example 2
- 1 script fw, ghi dữ liệu vào database người gửi và thời gian. Script sẽ đọc database không cho fw mail liên tục
- vacationcheck.sh cần database để lưu => tạo database, tạo user cho nó
- Tạo database và user
```
mysql
CREATE DATABASE postfixadmin;
CREATE USER 'postfixadmin'@'localhost' IDENTIFIED BY 'pass';
GRANT ALL PRIVILEGES ON postfixadmin.* TO 'postfixadmin'@'localhost';
FLUSH PRIVILEGES;
```
- Tạo bảng lưu trữ data
```
use postfixadmin
```
```
CREATE TABLE `sieve_count` (
 `from_address` varchar(254) NOT NULL,
 `date` datetime NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=latin1;
    
ALTER TABLE `sieve_count`
  ADD KEY `from_address` (`from_address`);
```
- Check: 
```
mysql -u postfixadmin – p
```
Nhập "pass" và login
`vim /usr/lib/dovecot/sieve-execute/vacationcheck.sh`
- Sript này login vào mysql và thực thi lệnh select giá trị từ database postfixadmin. Đếm giá trị mail trong khoảng thời gian bé hơn 50s, nếu giá trị trả về là 0 thì gửi mail và ngược lại
```
USER=postfixadmin
PASS=pass
DATABASE=postfixadmin

# DB STRUCTURE
#CREATE TABLE `sieve_count` (
#  `from_address` varchar(254) NOT NULL,
#  `date` datetime NOT NULL
#) ENGINE=InnoDB DEFAULT CHARSET=latin1;
#
#ALTER TABLE `sieve_count`
# ADD KEY `from_address` (`from_address`);

MAILS=$(mysql -u$USER -p$PASS $DATABASE --batch --silent -e "SELECT count(*) as ile FROM sieve_count WHERE from_address='$1' AND DATE_SUB(now(),INTERVAL $2 SECOND) < date;")
ADDRESULT=$(mysql -u$USER -p$PASS $DATABASE --batch --silent -e "INSERT INTO sieve_count (from_address, date) VALUES ('$1', NOW());")

# uncoment below to debug
# echo User $1 sent $MAILS in last $2 s >> /usr/lib/dovecot/sieve-pipe/output.txt
# echo Add result : $ADDRESULT >> /usr/lib/dovecot/sieve-pipe/output.txt
# echo $MAILS

if [ "$MAILS" = "0" ]
then
exit 0
fi

exit 1
```
`vim /sieve/extprogram.sieve`
```
require ["variables", "copy", "envelope", "vnd.dovecot.execute"];

# put the envelope-from address in a variable
 if envelope :matches "from" "*" { set "from" "${1}"; }
#
# # execute the vacationcheck.sh program and redirect the message based on its exit code
 if execute :output "vacation_message" "vacationcheck.sh" ["${from}","50"]
 {
 redirect
       :copy "hadt@bizflycloud.vn";
       }
```
`vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "extpr";
```
```
sievec /sieve/extprogram.sieve
sievec /etc/dovecot/sieve/before.sieve
systemctl restart dovecot
```
