# Fail2ban_Linux

** Fail2ban là một công cụ bảo mật theo dõi các tệp log (nhật ký hệ thống) và tự động cập nhật tường lửa (UFW/iptables) để cấm các địa chỉ IP có dấu hiệu tấn công rò rỉ mật khẩu (brute-force) hoặc các hành vi khả nghi khác. Dưới đây là quy trình chuẩn để cài đặt và cấu hình Fail2ban bảo vệ dịch vụ SSH trên máy chủ Ubuntu Server.

1. Cập nhật hệ thống và cài đặt Fail2ban: Đầu tiên, hãy đảm bảo danh sách các gói phần mềm trên máy chủ được cập nhật, sau đó tiến hành cài đặt Fail2ban.

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install fail2ban -y
```

2. Bật dịch vụ Fail2ban tự động chạy, đảm bảo phần mềm luôn hoạt động sau khi khởi động lại máy chủ. Khởi động service và thiết lập cho phép Fail2ban tự động chạy cùng hệ thống:

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

3. Tạo file cấu hình cục bộ (jail.local) đây là quy tắc quan trọng: Không bao giờ chỉnh sửa trực tiếp file jail.conf. Khi cập nhật phần mềm, tệp jail.conf mặc định có thể bị ghi đè. Bạn cần sao chép nó ra một tệp .local để lưu giữ cấu hình cá nhân an toàn:

```bash
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
```
4. Cấu hình các thông số bảo vệ cơ bản: Mở tệp cấu hình vừa tạo bằng trình soạn thảo (ví dụ Nano):
```bash
sudo nano /etc/fail2ban/jail.local
```
Tìm đến phần [DEFAULT] và điều chỉnh các thông số sau theo nhu cầu của bạn:

ignoreip = 127.0.0.1/8 ::1: Thêm IP tĩnh cá nhân hoặc IP nội bộ của bạn vào đây (cách nhau bằng dấu cách) để không bao giờ bị khóa nhầm.


bantime = 1h: Thời gian một IP bị cấm. Có thể dùng 1h (1 giờ), 1d (1 ngày), hoặc để số giây (ví dụ 3600).


findtime = 10m: Khoảng thời gian theo dõi log để đếm số lần lỗi.


maxretry = 5: Số lần thử thất bại tối đa được phép trong khoảng thời gian findtime trước khi IP bị cấm.


5. Kích hoạt bộ lọc bảo vệ SSH (SSHD Jail): Tiếp tục cuộn xuống trong tệp jail.local để tìm cấu hình cho dịch vụ SSH (nằm dưới mục [sshd]). Hãy thêm dòng enabled = true để bật:
```bash
[sshd]
enabled = true
port    = ssh
logpath = %(sshd_log)s
backend = %(sshd_backend)s
```

(Lưu ý: Nếu trước đó bạn đổi port SSH sang số port khác, ví dụ 2222, hãy sửa port = 2222). Lưu tệp và thoát (Ctrl+O -> Enter -> Ctrl+X trong Nano).

6. Khởi động lại dịch vụ và kiểm tra:Áp dụng cấu hình mới bằng cách khởi động lại Fail2ban:
```bash
sudo systemctl restart fail2ban
```

Kiểm tra trạng thái của các Jail (bộ lọc) đang hoạt động:
```bash
sudo fail2ban-client status
```
Để xem chi tiết danh sách các IP đang bị cấm bởi dịch vụ SSH:
```bash
sudo fail2ban-client status sshd
```

---

Để gỡ bỏ lệnh cấm (unban) một địa chỉ IP, bạn sử dụng công cụ fail2ban-client. Dưới đây là hai cách thực hiện tùy thuộc vào nhu cầu của bạn.

1. Gỡ cấm IP khỏi tất cả các bộ lọc (Jail)
Nếu bạn đang sử dụng bản Fail2ban tương đối mới (v0.10.2 trở lên) và muốn mở khóa IP trên toàn bộ hệ thống một cách nhanh chóng, hãy chạy lệnh sau:
```bash
sudo fail2ban-client status sshd
sudo fail2ban-client unban 192.168.1.100
```
2. Gỡ cấm IP khỏi một bộ lọc (Jail) cụ thể. Nếu bạn chỉ muốn mở khóa IP khỏi một dịch vụ nhất định (ví dụ như sshd), bạn cần chỉ định tên của jail đó trong lệnh, sau đó kiểm tra lại danh sách địa chỉ IP:
```bash
sudo fail2ban-client set sshd unbanip 192.168.1.100
sudo fail2ban-client status sshd
```
