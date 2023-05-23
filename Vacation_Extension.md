- Sieve vacation extension RFC 5230 xác định một cơ chế để tạo trả lời tự động cho các email đến
- Cần có nhiều biện pháp phòng ngừa khác nhau để đảm bảo rằng các câu trả lời chỉ được gửi khi thích hợp
- Tác giả script có thể chỉ định tần suất trả lời có thể được gửi đến một liên hệ cụ thể
- Trong vacation extension nguyên bản, khoảng thời gian này được chỉ định theo ngày với tối thiểu là một ngày
- Khi cần thiết hơn và đặc biệt là khi trả lời phải được gửi thường xuyên hơn 1 ngày, có thể sử dụng vacation-seconds extension RFC 6131
- Điều này cho phép chỉ định khoảng thời gian trả lời tối thiểu tính bằng s với khoảng thời gian trả lời tối thiểu bằng 0 (trả lời sau đó luôn được gửi), tuỳ thuộc vào cấu hình của admin
### Configuration
- Vacation extension là giá trị mặc định
- Ngược lại, `vacation-seconds` extension cần bật với cài đặt `sieve_extensions`
- Config cũng cần được điều chỉnh cho phù hợp để cho phép ktg không trả lời dưới 1 ngày
- .... (nào rảnh viết tiếp)
### My Config
#### 90-sieve.conf
- Enable extension
`sieve_extensions = +vacation-seconds`
- Minium period send vacation response per sender address 0 is no limit
`sieve_vacation_min_period = 0`
- Default period send vacation response per sender address
`sieve_vacation_default_period = 1d`
- Max period send vacation response per sender address
`sieve_vacation_max_period = 2d`
#### Script Sieve
`vim /sieve/va2.sieve`
```
 require ["vacation-seconds"];
   vacation :seconds 2
            :subject "Hi vacation roiiiiiiiiiiiii"
            :addresses ["..."]
            "Nhận mail rồi nhưng ko làm đâu ";
```
```
sievec /sieve/va2.sieve
```
`vim /etc/dovecot/sieve/before.sieve`
```
require ["enotify", "fileinto", "variables", "mailbox", "envelope", "copy", "body", "regex", "imap4flags","duplicate","include"];
include :global "va2";
```
```
sievec /etc/dovecot/sieve/before.sieve
systemctl restart dovecot
```
#### Test
- Send mail from outsite to ah@dinhha.online
- Check log
```
tail -f /var/log/mail.log | grep vacation
```
- Result Ok
```

root@mail:~# tail -f /var/log/mail.log | grep vacation
May 23 02:35:22 mail dovecot: lmtp(ah@dinhha.online)<1170861><Kye7IfrDa2St3REA8GMZgg>: sieve: msgid=<3c7c369855757d5801ef7781db49480e@anthanh264.site>: sent vacation response to <test2@anthanh264.site>
May 23 02:35:45 mail dovecot: lmtp(ah@dinhha.online)<1170861><EA22ExHEa2St3REA8GMZgg>: sieve: msgid=<9f76806ec3618b31f4d1aab71bc28cd6@anthanh264.site>: sent vacation response to <test2@anthanh264.site>

```
