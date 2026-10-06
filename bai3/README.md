# Bài 3: Cấu hình tường lửa UFW và chẩn đoán cổng mạng

## 1. Mục tiêu

- Sử dụng UFW để bảo vệ máy chủ.
- Thiết lập chính sách mặc định cho tường lửa.
- Chặn các kết nối đi vào không cần thiết.
- Cho phép các kết nối đi ra.
- Mở cổng SSH để quản trị máy chủ.
- Mở cổng 8080 cho ứng dụng Web.
- Sử dụng `ss`, `curl` và `ufw status verbose` để kiểm tra kết nối mạng.

---

## 2. Môi trường thực hành

- Hệ điều hành: Ubuntu Server
- Máy chủ: Cloud VPS
- Web Application: Beta App
- SSH Port: 2222
- Web Application Port: 8080

---

## 3. Cấu hình chính sách mặc định của UFW

Đặt chính sách mặc định:

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing