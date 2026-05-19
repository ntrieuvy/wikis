## 4. Biện pháp kỹ thuật - nghiệp vụ của hệ thống thanh toán

### 4.1. Phương thức thanh toán

Hệ thống hỗ trợ thanh toán bằng hình thức chuyển khoản ngân hàng thông qua mã QR theo chuẩn VietQR/NAPAS. Người sử dụng có thể thanh toán bằng các ứng dụng ngân hàng hoặc ví điện tử có hỗ trợ quét mã VietQR tại Việt Nam.

Sau khi người dùng thực hiện thanh toán thành công, hệ thống sẽ tự động tiếp nhận thông tin xác nhận giao dịch và cập nhật trạng thái đơn hàng tương ứng.

---

### 4.2. Quy trình xử lý thanh toán

Quy trình thanh toán được thực hiện theo các bước sau:

1. Người dùng xác nhận sử dụng dịch vụ và lựa chọn thanh toán.

2. Hệ thống kiểm tra trạng thái đơn hàng và tạo thông tin thanh toán tương ứng.

3. Hệ thống sinh mã QR thanh toán dành riêng cho từng giao dịch, bao gồm:

   * Số tiền thanh toán;
   * Nội dung chuyển khoản;
   * Thông tin nhận tiền.

4. Người dùng sử dụng ứng dụng ngân hàng hoặc ví điện tử để quét mã QR và thực hiện chuyển khoản.

5. Sau khi giao dịch được ngân hàng xác nhận thành công, hệ thống nhận thông tin đối soát giao dịch từ đơn vị tích hợp thanh toán.

6. Hệ thống kiểm tra:

   * Tính hợp lệ của giao dịch;
   * Sự khớp đúng về số tiền;
   * Nội dung chuyển khoản;
   * Trạng thái đơn hàng trước khi ghi nhận thanh toán thành công.

7. Sau khi xác nhận hợp lệ, hệ thống:

   * Cập nhật trạng thái thanh toán;
   * Kích hoạt quyền sử dụng dịch vụ;
   * Gửi thông báo xác nhận thanh toán cho người dùng qua email hoặc thông báo điện tử.

---

### 4.3. Biện pháp bảo mật và an toàn giao dịch

Để bảo đảm an toàn cho quá trình thanh toán trực tuyến, hệ thống áp dụng các biện pháp kỹ thuật và nghiệp vụ sau:

#### a) Bảo mật kết nối

* Toàn bộ quá trình trao đổi dữ liệu thanh toán được thực hiện thông qua kết nối bảo mật HTTPS/TLS.
* Các yêu cầu xác nhận giao dịch đều được xác thực trước khi xử lý.

---

#### b) Kiểm tra và đối soát giao dịch

Hệ thống thực hiện kiểm tra tự động đối với từng giao dịch thanh toán, bao gồm:

* Đối chiếu chính xác số tiền thanh toán;
* Kiểm tra nội dung chuyển khoản của giao dịch;
* Kiểm tra trạng thái đơn hàng;
* Từ chối các giao dịch không hợp lệ hoặc không khớp thông tin.

---

#### c) Ngăn chặn xử lý giao dịch trùng lặp

Hệ thống áp dụng cơ chế kiểm tra giao dịch nhằm:

* Ngăn chặn việc ghi nhận thanh toán nhiều lần cho cùng một đơn hàng;
* Đảm bảo mỗi giao dịch chỉ được xử lý thành công một lần duy nhất.

---

#### d) Đảm bảo tính toàn vẹn dữ liệu

Các bước cập nhật trạng thái thanh toán, đơn hàng và quyền sử dụng dịch vụ được xử lý đồng bộ trong cùng một quy trình nghiệp vụ nhằm bảo đảm:

* Không phát sinh trạng thái dữ liệu không đồng nhất;
* Có khả năng khôi phục dữ liệu khi xảy ra lỗi hệ thống.

---

#### e) Ghi nhận lịch sử giao dịch

Hệ thống lưu trữ lịch sử thanh toán và nhật ký xử lý giao dịch để phục vụ:

* Kiểm tra, đối soát;
* Giải quyết khiếu nại;
* Hỗ trợ cơ quan quản lý nhà nước khi cần thiết.

Dữ liệu thanh toán không cho phép chỉnh sửa hoặc xóa trái phép.

---

#### f) Kiểm soát giao dịch hết hạn hoặc hủy bỏ

Các giao dịch chưa hoàn tất trong thời gian quy định sẽ tự động chuyển sang trạng thái hết hạn. Người dùng có thể hủy giao dịch chưa thanh toán và hệ thống sẽ tự động giải phóng các tài nguyên hoặc ưu đãi đã tạm giữ trước đó.

---

### 4.4. Chính sách áp dụng khuyến mãi và mã giảm giá

Hệ thống hỗ trợ áp dụng mã giảm giá hoặc chương trình khuyến mãi trong quá trình thanh toán.

* Giá trị giảm giá được tính trước khi tạo giao dịch thanh toán.
* Trong thời gian chờ thanh toán, hệ thống tạm giữ quyền sử dụng mã giảm giá nhằm tránh việc sử dụng vượt quá số lượng cho phép.
* Sau khi thanh toán thành công, hệ thống xác nhận hoàn tất việc sử dụng mã giảm giá.
* Trường hợp giao dịch bị hủy hoặc hết hạn, quyền sử dụng mã giảm giá sẽ được hoàn trả theo quy định của chương trình khuyến mãi.

---

### 4.5. Công tác quản lý và hỗ trợ giao dịch

Hệ thống có cơ chế quản trị phục vụ việc:

* Theo dõi trạng thái giao dịch;
* Đối soát thanh toán;
* Tra cứu lịch sử thanh toán;
* Hỗ trợ xử lý khiếu nại hoặc sự cố phát sinh.

Các chức năng quản trị thanh toán được phân quyền và kiểm soát truy cập nhằm bảo đảm an toàn dữ liệu giao dịch của người sử dụng.
