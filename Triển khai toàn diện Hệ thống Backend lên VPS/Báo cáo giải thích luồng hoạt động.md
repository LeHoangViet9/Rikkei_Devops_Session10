# ========================================================
# BÀI TẬP 6: TRIỂN KHAI TOÀN DIỆN HỆ THỐNG BACKEND LÊN VPS
# ========================================================

## 1. FILE DOCKER-COMPOSE.YML
[Hãy copy toàn bộ nội dung file yaml ở Step 2 dán vào đây]


## 2. BÁO CÁO GIẢI THÍCH LUỒNG HOẠT ĐỘNG CỦA HỆ THỐNG

Hệ thống được thiết kế theo mô hình Microservices cơ bản, giao tiếp qua một mạng nội bộ Docker chung (`app_network`). Kiến trúc đảm bảo tính bảo mật và phân tách rõ ràng trách nhiệm của từng dịch vụ:

1. **API Gateway (Nginx):** 
   - Đóng vai trò là cửa ngõ duy nhất của hệ thống giao tiếp với Internet. 
   - Được publish cổng `80` ra máy host (VPS). 
   - Luồng dữ liệu: Khi người dùng hoặc Client gửi Request HTTP vào cổng 80 của VPS, Nginx sẽ tiếp nhận và đóng vai trò Reverse Proxy, định tuyến (forward) yêu cầu đó vào cổng `8080` của dịch vụ Backend.

2. **Spring Boot Backend (Application):**
   - Chứa logic nghiệp vụ của dự án. 
   - Bảo mật: Dịch vụ này KHÔNG publish port ra máy host (chỉ dùng `expose: 8080`). Do đó, hacker không thể truy cập trực tiếp vào backend từ bên ngoài Internet mà bắt buộc phải đi qua chốt chặn Nginx.
   - Kết nối DB: Thông qua cơ chế DNS nội bộ của Docker, Backend gọi tới Database bằng tên service là `postgres-db`. Biến môi trường `depends_on` đảm bảo Backend chỉ khởi động khi Database đã sẵn sàng.

3. **Database (PostgreSQL):**
   - Lưu trữ toàn bộ dữ liệu của hệ thống.
   - Hoàn toàn cách ly với Internet. Chỉ các container nằm trong cùng mạng `app_network` (như Backend) mới có quyền truy cập vào cổng `5432` của nó. 
   - Dữ liệu được mount ra ngoài thông qua Named Volume (`db_data`) để đảm bảo không bị mất data khi container khởi động lại hoặc bị xóa.

4. **Firewall (UFW & AWS Security Group):**
   - Mặc định chặn tất cả chiều vào (Deny Incoming).
   - Chỉ cho phép mở cổng `22` (để quản trị qua SSH) và cổng `80` (để phục vụ ứng dụng web).