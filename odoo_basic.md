Module về OTP:

Mô tà về module như sau (chỉ để hiểu)
- Tạo một module về OTP để lưu lại các mã otp được gửi đi bởi hệ thống
- Tự động xóa mã otp đã được sử dụng (định kỳ)
- Module này nhằm mục đích kiểm tra mã OTP thông qua một request http (hoặc json) đến server Odoo
- Khi một người dùng đăng ký và điền số điện thoại, sẽ tự động gửi về mã OTP để người dùng nhập vào, hệ thống sẽ xác nhận mã điền vào có đúng hay không.

Khởi động:
Tạo module tên `hoang_otp_code`


Bài 1.
Kế thừa model `res.users` và thêm vào fields sau:
	+ `user_type`: selection có 2 giá trị (đã xác thực, chưa xác thực) (mặc định chưa xác thực)
Hiện trường `user_type` ra formview và listview của model `res.users`
	+ Đường dẫn đến formview `Setting \  Users & Companies \  Users`
	+ Vị trí hiện trên `formview`: http://bit.ly/2HdHPEe (bật chế độ debug lên để thấy trường `Related partner`)
	+ Vị trí hiện trên `listview`: http://bit.ly/2HgyV91 (sau cột login)

Bài 2.1
Tạo model `otp.code` gồm các trường:
	+ `user_id`: m2o đến `res.users` (người nào sở hữu otp này)
	+ `otp_code`: Char (mã otp là gì)
	+ `is_used`: Bool (đã được sử dụng hay chưa?)
	+ `create_date`: datetime (thời điểm tạo otp) (thực ra trường này ko cần tạo vì khi tạo một model bình thường odoo đã có sẵn)
Thêm các phương thức:
	+ `create_otp_code`: 
		.. tham số nhận vào: `user_id : int`
		.. trả về: object model otp vừa tạo
		.. phương thức này giúp tạo ra một bản ghi của model `otp.code` với:
			* user_id là tham số user_id nhận vào
			* otp_code là mã OTP được tạo ngẫu nhiên là chuỗi gồm 6 ký tự số. Ví dụ: `'323532'`, `'945804'`
			* is_used là false
			* create_date là thời gian hiện tại
	+ `check_otp_code`:
		.. tham số nhận vào là `user_id : int`, `otp_code : char`
		.. trả về `true` | `false`
		.. `true` nếu thỏa mã điều kiện sau:
			* tồn tại bản ghi model `otp.code` thỏa mãn tham số truyền vào
			* mã otp vừa tìm được, vừa được tạo không quá 2 phút
	+ `mark_as_used`:
		.. tham số nhận vào là `user_id : int`, `otp_code : char`
		.. trả về `true | false` nếu thành công / thất bại
		.. phương thức này giúp đánh dấu rằng mã `otp_code` của `user_id` đã được sử dụng

Bài 2.2
Kế thừa model `res.users` và thêm phương thức:
	+ `validate_by_otp`:
		.. phương thức này đánh dấu user đã được xác thực
		.. trả về `true` nếu thực hiện thành công

Bài 3
Tạo `listview` `formview` cho model `otp.code`
	+ Đối với `listview`
		.. hiện như sau http://bit.ly/2Hc0BvP
		.. Nếu mã otp đã được sử dụng (dựa vào trường `is_used`), hiện dòng màu xám
		.. Nếu mã otp chưa được sử dụng, hiện dòng màu xanh làm 
		.. tham khảo https://bit.ly/2EJ9B9U (admin/admin) (cấm tạo dữ liệu linh tinh)
	+ Đối với `formview` hiện thị như sau
		.. http://bit.ly/2HchOoN


Bài 4



