---
title: "Workshop"
date: 2024-01-01
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Triển khai và cấu hình ứng dụng web fullstack trên AWS

#### Tổng quan

Trong chuỗi bài thực hành này, chúng ta sẽ xây dựng và triển khai một hệ thống web **Fullstack** gồm Frontend tĩnh, Backend Spring Boot và cơ sở dữ liệu SQL Server trên nền tảng **Amazon Web Services (AWS)**.

Hệ thống được thiết kế theo các nguyên tắc tốt nhất về hiệu năng, khả năng mở rộng, bảo mật và tính sẵn sàng cao:
- **Identity & Access Management**: Tích hợp **AWS Cognito** để đăng nhập liên kết với Google OAuth.
- **Thiết kế mạng & bảo mật**: Xây dựng **VPC** riêng với public/private subnet tách biệt, kiểm soát truy cập bằng **Security Groups** và **AWS WAF**.
- **Lưu trữ & phân phối CDN**: Hosting Frontend tĩnh trên **Amazon S3** và phân phối toàn cầu bằng **Amazon CloudFront**.
- **Tự động co giãn**: Kết hợp **Application Load Balancer (ALB)**, **Auto Scaling Group (ASG)** và **EC2** để điều chỉnh số lượng backend theo tải thực tế.
- **Cơ sở dữ liệu**: Triển khai **Amazon RDS SQL Server** trong private subnet.
- **Thông báo giao dịch**: Gửi email tự động bằng **Amazon SES**.


#### Nội dung

1. [Cấu hình AWS Cognito](5.1-AWS-Cognito/)
2. [VPC và RDS](5.2-VPC-RDS/)
3. [Triển khai ứng dụng (S3, CloudFront, EC2, WAF, SES)](5.3-S3-CloudFront-EC2-WAF-SES/)
4. [Auto Scaling cho Backend](5.4-Auto-Scaling/)
5. [Ước tính chi phí dịch vụ AWS](5.5-Cost-Estimation/)
