---
title : "Ước tính chi phí dịch vụ AWS"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---
### BẢNG ƯỚC TÍNH CHI PHÍ CÁC DỊCH VỤ AWS

#### 1. Nhóm Mạng và Phân phối (Network & Content Delivery)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **Amazon CloudFront** | - Miễn phí 1 TB truyền dữ liệu ra internet mỗi tháng.<br>- Miễn phí 10,000,000 yêu cầu HTTP/HTTPS mỗi tháng. | Truyền tải vượt mức: **$0.085/GB** |
| **Amazon VPC** | - Môi trường mạng riêng ảo chứa toàn bộ hệ thống (gồm 2 Availability Zones). | **Miễn phí** (Free) |
| **Internet Gateway (IGW)** | - Cổng giao tiếp cho phép VPC kết nối ra Internet công cộng. | **Miễn phí** (Free) |
| **Application Load Balancer (ALB)** | - Phân phối tải lưu lượng truy cập xuống các máy chủ EC2 backend. | ~**$0.0225/giờ** (khoảng $16.20/tháng) |
| **NAT Gateway** | - 1 NAT Gateway đặt tại Public Subnet để EC2 trong mạng riêng gọi API ra ngoài. | ~**$0.045/giờ** (khoảng $32.40/tháng) + **$0.045/GB** dữ liệu truyền qua |

---

#### 2. Nhóm Bảo mật và Định danh (Security & Identity)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **AWS WAF (Web Application Firewall)**| - Tường lửa bảo vệ web để lọc traffic xấu và chống DDoS. | **$10.60/tháng** (mức cơ bản) |
| **AWS Certificate Manager (ACM)** | - Quản lý, cấp phát chứng chỉ SSL/TLS phục vụ kết nối HTTPS. | **Miễn phí** (khi sử dụng kèm CloudFront) |

---

#### 3. Nhóm Máy chủ tính toán (Compute)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **Amazon EC2 (Web Server)** | - Các máy chủ backend Spring Boot chạy trong Private Subnet (loại instance `t3.micro`, chạy liên tục 24/7). | **$20.81/tháng** (mỗi instance) |
| **Auto Scaling Group (ASG)** | - Tự động quản lý số lượng và trạng thái của các máy chủ EC2 backend. | **Miễn phí** (chỉ tính tiền tài nguyên EC2 sinh ra) |

---

#### 4. Nhóm Lưu trữ và Cơ sở dữ liệu (Storage & Database)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **Amazon S3** | - Lưu trữ mã nguồn tĩnh Frontend (HTML, CSS, JS, hình ảnh). | **$0.023/GB** (cho 50 GB đầu tiên) |
| **Amazon RDS (SQL Server)** | - Cơ sở dữ liệu quan hệ, chạy chế độ Multi-AZ (1 DB Primary và 1 DB Standby dự phòng). | **$163.81/tháng** |

---

#### 5. Nhóm Tích hợp ứng dụng (Application Integration)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **Amazon SES (Simple Email Service)** | - Gửi email thông báo tự động cho người dùng. | **$0.0001/email** ($0.10 cho 1,000 email) |

---

#### 6. Nhóm Giám sát và Xác thực (Observability & Identity)

| Dịch vụ | Mô tả cấu hình / Tài nguyên | Đơn giá ước tính (Tháng/Đơn vị) |
| :--- | :--- | :--- |
| **Amazon Cognito** | - Quản lý định danh người dùng và xác thực tài khoản. | - **Miễn phí 50,000 MAU** (người dùng hoạt động hàng tháng) đầu tiên.<br>- Vượt hạn mức: **$0.0055/MAU** |

---

### TỔNG CHI PHÍ ƯỚC TÍNH HÀNG THÁNG

> [!IMPORTANT]
> **Tổng chi phí ước tính tối thiểu**: Từ **$245.28/tháng** trở lên (tùy thuộc vào lưu lượng truyền tải thực tế, số lượng instance EC2 tự động tăng lên bởi ASG và lượng dữ liệu truyền qua NAT Gateway).
