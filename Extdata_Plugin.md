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
