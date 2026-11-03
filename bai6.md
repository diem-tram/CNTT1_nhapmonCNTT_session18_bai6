###### **Bài 6 : Kiểm tra và tối ưu hiệu năng Portal**

**Phần 1**

Nút thắt cổ chai chính gây giật/độ trễ 20 giây

Lỗi Render-Blocking do File tĩnh quá lớn: Các file hình ảnh chưa nén (đặc biệt là ảnh bản đồ lộ trình cho Shipper) dung lượng lên tới vài Megabyte (MB). Khi nạp đồng thời trên biểu đồ Waterfall, các request này chiếm trọn băng thông kết nối, làm kéo dài thời gian kết thúc tải trang (Load / DOMContentLoaded).



Lỗi 404 (Not Found) làm đứt chuỗi nạp: Các yêu cầu gọi file CSS hoặc JavaScript quan trọng bị tô đỏ với Status 404. Việc thiếu CSS/JS khiến trình duyệt (Rendering Engine) không thể dựng xong giao diện hoàn chỉnh hoặc không khởi chạy được thư viện bản đồ tĩnh/động.

**Phần 2**

Bước 1: Sửa lỗi đứt link (Status 404)

Kiểm tra chính xác đường dẫn tương đối/tuyệt đối của các file CSS và JS bị tô đỏ trong tab Network.



Cập nhật lại đường dẫn trong thẻ <link rel="stylesheet" href="..."> và <script src="..."> cho chính xác với cấu trúc thư mục trên server.



Sử dụng các thuộc tính defer hoặc async phù hợp cho file JS không bắt buộc blocking để tối ưu tiến trình parse HTML.



Bước 2: Nén và tối ưu hóa tài nguyên hình ảnh

Chuyển đổi định dạng: Chuyển đổi các hình ảnh bản đồ/lộ trình từ PNG/JPG gốc sang các định dạng hiện đại có nén cao như WebP hoặc AVIF.



Resize \& Responsive: Hạ độ phân giải ảnh về đúng kích thước hiển thị tối đa trên thiết bị di động của Shipper.



Lazy Loading: Thêm thuộc tính loading="lazy" vào các thẻ <img> chưa cần hiển thị ngay ở màn hình đầu (below-the-fold).



Bước 3: Bật cơ chế Browser Cache cho tài nguyên tĩnh

Phía Web Server (Nginx / Apache), cấu hình HTTP Response Headers cho các file tĩnh (.css, .js, .webp, .png, .jpg):



Khai báo header Cache-Control: public, max-age=31536000, immutable.



Cấu hình validator ETag hoặc Last-Modified.

**Phần 3**

Nguyên nhân: Trình duyệt gửi request kèm header xác thực If-None-Match (chứa ETag) hoặc If-Modified-Since. Server kiểm tra thấy tài nguyên tĩnh (CSS, JS, Image) trên server không có thay đổi gì so với bản lưu ở máy client.



Tín hiệu tốt cho băng thông hệ thống:



Server chỉ trả về header HTTP nhỏ nhẹ (khoảng vài trăm bytes) chứa mã 304 thay vì gửi lại toàn bộ dung lượng file (vài MB).



Giảm tải tối đa tài nguyên xử lý cho CPU/RAM của Web Server và tiết kiệm băng thông mạng.



Màn hình phía Shipper hiển thị ngay lập tức từ bộ nhớ đệm local, giải quyết triệt để tình trạng tải chậm.

