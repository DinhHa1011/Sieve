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
  - Field này chỉ định trường tiêu đề chứa thông tin kết quả của trình quét soam và nó có thể diễn đạt syntax nội dung của header. Nó không match header được tìm trong message, dòng lệnh spamtest sẽ match với '0'
  - Đây là một cài đặt structured. Phần đầu tiên chỉ định tên header field
  - Tùy chọn, một biểu thức chính quy posix theo sau tên field header, cách nhau bởi dấu hai chấm ':'. Bất kì khoảng trắng nào ngay sau dấu 2 chấm không phải là một phần của biể thức chính quy. Nếu biểu thức chính quy bị bỏ qua, mọi nội dung  header sẽ được chấp nhận và giá trị header đầy đủ được sử dụng
  - Khi một biểu thức chính quy được sử dụng, nó phải chỉ định một giá trị match (trong ngoặc) mang lại kết quả test spam mong muốn
  - Nếu header không match với biểu thức chính quy hoặc nó không match với giá trị nào được tìm thấy, test `spamtest` sẽ match với 0 trong quá trình thực thi Sieve script
`sieve_spanmtest_max_value =`
- Điều này chỉ định tĩnh giá trị tối đa mà điểm số spam có thể có
`sieve_spamtest_max_header = <header-field> [":" <regexp> ]
- Một vài spam scanners bao gồm giá trị điểm max trong một trong các status header của chúng ta. Sử dụng setting này, maximum này có thể được trích xúât từ chính thông báo thay vì chird dịnh số tối đa theo cách thử công bằng sử dụng cài đặt `sieve_spamtest_max_value` ở trên
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
