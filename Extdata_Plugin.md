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
  
