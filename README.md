# Đồ án: Triển khai Website Quảng bá Sản phẩm (WordPress) với Docker, Giám sát và Bảo mật

Hệ thống được thiết kế và triển khai trên môi trường **Ubuntu** sử dụng **Docker Compose** và **`nano`**, tích hợp đầy đủ các dịch vụ ứng dụng, cơ sở dữ liệu, quản trị, giám sát hệ thống và bảo mật (Hardening).

---

## 🛠️ 1. Kiến trúc Hệ thống & Các dịch vụ (8 Containers)
Hệ thống chạy trên một mạng bridge chung (`app-net`) gồm các dịch vụ:
1. **`db` (MySQL 8.0):** Cơ sở dữ liệu lưu trữ cho WordPress (có thiết lập `healthcheck`).
2. **`phpmyadmin` (Port `8081`):** Giao diện web quản lý cơ sở dữ liệu MySQL.
3. **`wordpress` (Port `80` nội bộ):** Mã nguồn ứng dụng WordPress chính.
4. **`nginx` (Ports `80`, `443`):** Reverse Proxy, cấu hình SSL tự ký (HTTPS) và tích hợp các Header bảo mật.
5. **`prometheus` (Port `9091`):** Thu thập metrics giám sát hiệu năng hệ thống.
6. **`grafana` (Port `3001`):** Giao diện trực quan hóa Dashboard giám sát.
7. **`loki` (Port `3100`):** Hệ thống lưu trữ log tập trung (phiên bản `2.9.2`).
8. **`promtail`:** Thu thập log từ các container Docker đẩy về Loki.

---

## 🚀 2. Hướng dẫn Triển khai (Installation Guide)

### Yêu cầu tiên quyết
* Hệ điều hành Ubuntu đã cài đặt sẵn **Docker** và **Docker Compose**.

### Các bước thực hiện:
1. **Clone hoặc tải mã nguồn về thư mục làm việc:**
   ```bash
   cd ~/website-wordpress
