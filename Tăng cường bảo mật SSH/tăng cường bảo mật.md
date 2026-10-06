# ==========================================
# BÀI TẬP 3: TĂNG CƯỜNG BẢO MẬT SSH
# ==========================================

# 1. THAO TÁC TRÊN MÁY TÍNH CÁ NHÂN (WINDOWS CMD)
# Tạo cặp khóa SSH Key (nhấn Enter 3 lần để bỏ qua mật khẩu)
ssh-keygen -t ed25519 -C "deploy_vps"

# In và copy nội dung Public Key
type "%USERPROFILE%\.ssh\id_ed25519.pub"


# 2. THAO TÁC TRÊN VPS AWS (USER DEPLOY)
# Chuyển sang user deploy (nếu đang ở root)
su - deploy

# Tạo thư mục ẩn .ssh và thiết lập quyền
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Tạo file cấu hình chìa khóa và dán Public Key vào
nano ~/.ssh/authorized_keys

# Thiết lập quyền bảo mật cho file chìa khóa
chmod 600 ~/.ssh/authorized_keys

# Thoát khỏi user deploy để quay lại quyền root
exit


# 3. THAO TÁC TRÊN VPS AWS (QUYỀN ROOT)
# Mở file cấu hình SSH để sửa PermitRootLogin no và PasswordAuthentication no
nano /etc/ssh/sshd_config

# (Xử lý lỗi thiếu thư mục tạm và test lại cú pháp do dịch vụ bị gián đoạn)
mkdir -p /run/sshd
sshd -t

# Khởi động lại dịch vụ SSH để áp dụng cấu hình mới
systemctl restart ssh


# 4. KIỂM TRA KẾT QUẢ
# Thử đăng nhập lại bằng root (sẽ bị báo Permission denied (publickey))
ssh root@13.50.226.4