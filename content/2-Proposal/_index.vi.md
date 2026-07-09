---
title: "Bản đề xuất"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# Hệ thống quản lý và đặt sân cầu lông trực tuyến CauLongVui trên nền tảng AWS

## 1. Mục tiêu dự án

**CauLongVui** là nền tảng quản lý và đặt sân cầu lông toàn diện, được xây dựng để giải quyết đồng thời bài toán nghiệp vụ và bài toán hạ tầng cloud.

### Mục tiêu nghiệp vụ
- Số hóa toàn bộ quy trình vận hành của trung tâm cầu lông.
- Hỗ trợ tra cứu sân trống và giữ chỗ theo thời gian thực để giảm trùng lịch.
- Tích hợp thanh toán online qua ví nội bộ và các cổng bên ngoài như MoMo, VNPay.
- Hỗ trợ quản lý hội viên, cho thuê phụ kiện và đặt trước đồ ăn, thức uống.

### Mục tiêu kỹ thuật
- Xây dựng kiến trúc cloud theo các nguyên tắc **High Availability**, **Defense in Depth** và **Auto Scalability**.
- Triển khai trên **Amazon Web Services (AWS)** để tăng độ ổn định, bảo mật và khả năng mở rộng.

## 2. Vấn đề giải quyết và lợi ích

### Vấn đề hiện tại
- Quy trình đặt sân thủ công dễ dẫn đến trùng lịch.
- Dữ liệu doanh thu bị phân tán giữa tiền sân, đồ uống, phụ kiện và thẻ hội viên.
- Lưu lượng truy cập cao có thể làm hệ thống quá tải và gián đoạn dịch vụ.

### Lợi ích mang lại
- Đặt sân realtime giúp khách hàng thao tác nhanh và chính xác hơn.
- Dữ liệu thanh toán và đơn hàng được tập trung về một nơi để quản trị dễ hơn.
- AWS Auto Scaling và các dịch vụ quản lý giúp tăng độ bền hệ thống.
- CloudWatch, X-Ray và SNS hỗ trợ giám sát, truy vết và cảnh báo rõ ràng.

## 3. Tổng quan giải pháp

Hệ thống sử dụng kiến trúc AWS 3 lớp:
- **Lớp trình bày**: Frontend tĩnh được lưu trên Amazon S3 và phân phối toàn cầu qua CloudFront.
- **Lớp ứng dụng**: Backend Spring Boot chạy trên Amazon EC2 trong private subnet, được bảo vệ bởi ALB, Auto Scaling và WAF.
- **Lớp dữ liệu**: Amazon RDS lưu dữ liệu giao dịch ở chế độ Multi-AZ, còn file upload được lưu trên Amazon S3.

### 3.1. Sơ đồ kiến trúc

![Kiến trúc AWS CauLongVui](/images/2-Proposal/caulongvui-architecture.jpg)

### 3.2. Các dịch vụ AWS chính

- **Amazon S3** lưu frontend tĩnh và các file media upload.
- **Amazon CloudFront** tăng tốc phân phối nội dung toàn cầu.
- **AWS WAF** bảo vệ ứng dụng khỏi các kiểu tấn công web phổ biến.
- **Amazon EC2** chạy backend Spring Boot trong private subnet.
- **Application Load Balancer (ALB)** phân phối request tới các instance khỏe mạnh.
- **Auto Scaling Group (ASG)** tự động điều chỉnh số lượng backend theo tải.
- **Amazon RDS Multi-AZ** cung cấp cơ sở dữ liệu SQL Server được quản lý và sẵn sàng cao.
- **NAT Gateway** cho phép backend gọi dịch vụ bên ngoài an toàn.
- **Amazon SES** gửi email xác nhận đặt sân và hóa đơn.
