- `editheader` extension RFC 5293 cho phép Sieve script xóa và thêm fields message header, do đó cho phép tương tác với các thành phần khác sử dụng hoặc tạo ra các deader fields
### Configuration
- `editheader` extension không khả dụng theo mặc định và cần được bật rõ ràng bằng cách thêm extension này vào cài đặt sieve_extensions
- Các cài đặt sau có thể được định cấu hình cho `editheader` extension (các giá trị mặc định được chỉ định):
`sieve_editheader_max_header_size = 2048`
  - Kích thước tối đa bằng byte của giá trị trường header được truyền cho lệnh `addheader`
  - Giá trị tối thiểu cho cài đặt này là 1024 byte
  - Giá trị tính bằng byte, trừ khi theo sau là k(ilo)
`sieve_editheader_forbid_add = `
  - Một danh sách các header được phân tách bằng dấu cách không thể thêm vào message header
  - Bổ sung `Subject`: không thể cấm header, theo yêu cầu của đặc tả RFC
  - Do đó, việc thêm header này vào cài đặt này không có hiệu lực
`sieve_editheader_forbid_delete = `
  - Danh sách các header không thể xóa từ message header
  - Xóa trường `Received:` và `Auto-Submitted:` luôn bị cấm, trong khi xóa header `Subject:` không thể bị cấm, theo yêu cầu của đặc tả RFC. 
  - Do đó, việc thêm một trong các header này vào cài đặt không có hiệu lực
`sieve_editheader_protected = `
  - Danh sách các header không thể thêm hoặc xóa từ message header
  - Cài đặt này cung cấp để tương thích ngược
  - Nó là một sự kết hợp của setting `sieve_editheader_forbid_add` và `sieve_editheader_forbid_delete`
  - Những hạn chế tương tự được áp dụng
- Các giá trị không hợp lệ cho các cài đặt ở trên sẽ khiến trình thông dịch Sieve ghi nhật ký cảnh báo và hoàn nguyên về các giá trị mặc định
### Example
  ```
  plugin {
    # Use aditheader
    sieve_extensions = +editheader
    
    # Header fields must not exceed one kilobyte
    sieve_editheader_max_header_size = 1k
    
    # Protected special headers
    sieve_editheader_forbid_add = X-Verified
    sieve_editheader_forbid_delete = X-Verified X-Seen
  }
  ```
