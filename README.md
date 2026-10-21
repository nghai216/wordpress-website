# Đồ án: Triển khai Hệ thống Website Quảng bá Sản phẩm (WordPress) trên Ubuntu với Docker Compose, Giám sát, Log tập trung và Bảo mật

---

## 📋 Mục lục
1. [Giới thiệu Đề tài](#1-giới-thiệu-đề-tài)
2. [Kiến trúc Hệ thống & Sơ đồ Network](#2-kiến-trúc-hệ-thống--sơ-đồ-network)
3. [Chi tiết Các dịch vụ trong Hệ thống](#3-chi-tiết-các-dịch-vụ-trong-hệ-thống)
4. [Hướng dẫn Cài đặt & Triển khai Từ A-Z](#4-hướng-dẫn-cài-đặt--triển-khai-từ-a-z)
5. [Thông tin Truy cập Dịch vụ](#5-thông-tin-truy-cập-dịch-vụ)
6. [Hệ thống Giám sát & Log tập trung](#6-hệ-thống-giám-sát--log-tập-trung)
7. [Các Biện pháp Hardening & Bảo mật Hệ thống](#7-các-biện-pháp-hardening--bảo-mật-hệ-thống)

---

## 1. Giới thiệu Đề tài
Đề tài tập trung xây dựng một hệ thống website doanh nghiệp/quảng bá sản phẩm sử dụng **WordPress**, được triển khai hoàn toàn bằng **Docker Compose** trên nền tảng **Ubuntu Server**. Hệ thống đáp ứng toàn diện các yêu cầu khắt khe: ứng dụng web kết nối cơ sở dữ liệu ổn định, reverse proxy qua Nginx với chứng chỉ bảo mật SSL/HTTPS, tích hợp hệ thống giám sát hiệu năng (`Prometheus`, `Grafana`), quản lý log tập trung (`Loki`, `Promtail`) và áp dụng các tiêu chuẩn bảo mật hệ thống (`Hardening`).

---

## 2. Kiến trúc Hệ thống & Sơ đồ Network
Hệ thống sử dụng một Docker bridge network riêng biệt mang tên **`app-net`** để cô lập và bảo mật giao tiếp nội bộ:
* **Internet** truy cập vào **Nginx Reverse Proxy** qua các cổng chuẩn `80` (HTTP) và `443` (HTTPS - SSL tự ký).
* **Nginx** điều phối request vào mã nguồn **WordPress** (chạy ngầm bên trong mạng nội bộ).
* **WordPress** kết nối trực tiếp đến **MySQL Database (`db`)**.
* Các công cụ quản trị và giám sát (`phpMyAdmin`, `Prometheus`, `Grafana`, `Loki`) được phân tách rõ ràng qua các cổng dịch vụ định mức.

---

## 3. Chi tiết Các dịch vụ trong Hệ thống (8 Containers)
1. **`db` (MySQL 8.0):** Lưu trữ dữ liệu hệ thống WordPress, tích hợp cơ chế `healthcheck` để đảm bảo container ứng dụng chỉ khởi động khi cơ sở dữ liệu đã thực sự sẵn sàng.
2. **`phpmyadmin` (Port `8081`):** Giao diện quản trị cơ sở dữ liệu trực quan qua web.
3. **`wordpress` (Chạy ngầm):** Mã nguồn ứng dụng website chính.
4. **`nginx` (Ports `80`, `443`):** Đóng vai trò Reverse Proxy, chịu trách nhiệm điều hướng request, ép buộc chuyển hướng bảo mật HTTPS và gắn các Security Headers.
5. **`prometheus` (Port `9091`):** Thu thập metrics định kỳ từ các thành phần trong hệ thống.
6. **`grafana` (Port `3001`):** Nền tảng dashboard trực quan hóa dữ liệu giám sát và log.
7. **`loki` (Port `3100`):** Máy chủ nhận và lưu trữ log tập trung (phiên bản `2.9.2`).
8. **`promtail`:** Tác nhân (agent) tự động đọc log sinh ra từ Docker containers và đẩy về Loki.

---

## 4. Hướng dẫn Cài đặt & Triển khai Từ A-Z

### Bước 1: Chuẩn bị môi trường trên Ubuntu
Cập nhật hệ thống và kiểm tra công cụ Docker:
```bash
sudo apt update && sudo apt upgrade -y
docker --version
docker compose version
```
### Bước 2: Tạo thư mục dự án và cấu trúc cấu hình
Tạo các thư mục lưu trữ mã nguồn và file cấu hình cho Nginx, Prometheus và Loki:

```Bash
mkdir -p ~/website-wordpress/nginx/ssl
mkdir -p ~/website-wordpress/prometheus
mkdir -p ~/website-wordpress/loki
cd ~/website-wordpress
```
### Bước 3: Tạo chứng chỉ SSL tự ký cho Nginx (HTTPS)
Sinh chứng chỉ bảo mật để chạy mã hóa HTTPS trên Nginx:

```Bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout nginx/ssl/selfsigned.key \
  -out nginx/ssl/selfsigned.crt \
  -subj "/CN=localhost"
```
### Bước 4: Khởi chạy hệ thống bằng Docker Compose
Khởi động dịch vụ cơ sở dữ liệu (db) trước để hệ thống chờ trạng thái ổn định:

```Bash
docker compose up -d db
Kiểm tra trạng thái container cho đến khi báo trạng thái healthy:
```
```Bash
docker compose ps
Khi cơ sở dữ liệu đã sẵn sàng, tiến hành khởi chạy toàn bộ các dịch vụ còn lại:
```
```Bash
docker compose up -d --build
```
## 5: Thông tin Truy cập Dịch vụ
Trang web WordPress (HTTPS): https://localhost

phpMyAdmin (Quản lý DB): http://localhost:8081 (User: root / Pass: mật khẩu cấu hình trong compose)

Grafana (Dashboard giám sát): http://localhost:3001 (User: admin / Pass: admin)

Prometheus (Metrics): http://localhost:9091

## 6: Hệ thống Giám sát & Log tập trung
Prometheus & Grafana: Thu thập số liệu tài nguyên (CPU, RAM, Network) của các container và hiển thị biểu đồ trực quan.

Loki + Promtail: Promtail tự động đọc luồng log từ các container Docker đẩy về Loki. Bạn có thể sử dụng ngôn ngữ LogQL trên Grafana Explore để truy vấn:

Lọc log theo container Nginx: {container="wp_nginx"}

Tìm kiếm lỗi toàn hệ thống: {container=~"wp_.*"} |= "error"

## 7: Các Biện pháp Hardening & Bảo mật Hệ thống
Network Isolation: Phân chia mạng riêng app-net, chặn tuyệt đối các kết nối trái phép vào container database từ bên ngoài mạng chung.

Nginx Security Headers: Tích hợp đầy đủ các tiêu chuẩn chống tấn công phổ biến trên web server:

X-Frame-Options: "SAMEORIGIN" (chống Clickjacking)

X-XSS-Protection: "1; mode=block" (chống XSS)

X-Content-Type-Options: "nosniff" (chống MIME-sniffing)

Database Security: Sử dụng mật khẩu bảo mật và phân quyền tài khoản chặt chẽ cho ứng dụng WordPress.

Container Healthcheck & Restart Policy: Đảm bảo tính sẵn sàng cao với chính sách tự động khôi phục (restart: always) và cơ chế kiểm tra sức khỏe dịch vụ định kỳ.
