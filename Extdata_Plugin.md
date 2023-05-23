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
