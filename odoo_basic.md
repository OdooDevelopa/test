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
	+ `user_id`: m2o đến `res.users` (người nào sở hữu otp này, lưu ý: khi user bị xóa, toàn bộ otp của user cũng bị xóa theo)
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

Bài 2.3
Tạo module `hoang_otp_code_ext`
Kế thừa model `otp.code` thêm trường:
	+ `is_expired`: Bool, compute (mã này đã hết hạn hay chưa)
	+ `otp_code`: thêm thuộc tính `required`
	
Trường `is_expired` sẽ được tính toán dựa trên điều kiện sau:
	+  `is_expired` là `true` nếu mã otp này đã được sử dụng hoặc
							  nếu thời điểm tạo quá 30 phút so với hiện tại
							  nếu tồn tại mã otp khác mới hơn có `user_id` == `user_id` của otp hiện tại (...)

Hiện thị trên `formview` tùy ý đặt vị trí.

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
Tạo `http.route` có tên `/otp-code/validete-otp/`
	+ type `json`, auth `none`
	+ nhận vào tham số `user_id : int`, `otp_code : char`
	+ trả về: 
		.. nếu validate thành công `{'status': 'success'}`
		.. nếu validate thất bại `{'status': 'fail'}`
	+ route này thực hiện các nhiệm vụ sau:
		.. kiểm tra otp truyền vào có hợp lệ thông qua phương thức `check_otp_code` của model `otp.code`
		.. nếu hợp lệ đánh dấu rằng mã otp dã được sử dụng thông qua phương thức `mark_as_used`
		.. sau đó đánh dấu người dùng này đã được xác thực thông qua phương thức `validate_by_otp`
		.. nếu qua tất cả các bước trên validate thành công, ngược lại thì thất bại.
Kiểm sử dụng công cụ `postman` kiểm tra route trên và chụp ảnh gửi kèm bài nộp.

Bài 5
Tạo automated test cho module trên với test case sau:
	- Tạo liên tiếp 4 record otp
	- Với 3 otp đầu, kiểm tra độ dài otp có bằng 6 & chưa được sử dụng
	- với otp  thứ 4, kiểm tra phải đưa ra kết quả đây là otp spam
	- với 3 otp đầu, kiểm tra otp có hợp lệ (expect: 2 cái đầu không hợp lệ, cái thứ 3 hợp lệ)

Bài 6
Tạo `schedule action` model `ir.cron` tự động xóa những bản ghi model `otp.code` đã được sử dụng hoặc đã hết hạn quá 30 phút.
