---
title: "Các bài blog đã đăng"
date: 2024-01-01
weight: 3
chapter: false
pre: " <b> 3. </b> "
---

Phần này giới thiệu các bài blog kỹ thuật mà mình đã thực hiện sau khi tìm hiểu tài liệu AWS và các bài viết chính thức trên AWS Blog. Các bài blog tập trung tóm tắt những khái niệm quan trọng, các dịch vụ AWS, kiến trúc tham chiếu và những ứng dụng thực tế liên quan đến điện toán đám mây và bảo mật trên nền tảng AWS.

### [Blog 1 - Tìm hiểu kiến trúc lưu trữ Binary Assets với Lore trên AWS](3.1-Blog1/)

Bài blog này giới thiệu cách **Lore** xây dựng kiến trúc lưu trữ binary assets dựa trên mô hình lưu trữ theo nội dung (Content-Addressed Storage) trên AWS. Bài viết phân tích những hạn chế của phương pháp lưu trữ truyền thống và trình bày cách các dịch vụ như Amazon S3, Amazon EC2, Amazon ECS, AWS Cloud Map và Amazon DynamoDB phối hợp với nhau để giảm dữ liệu trùng lặp, tối ưu chi phí lưu trữ và nâng cao khả năng mở rộng cho các dự án game và truyền thông số quy mô lớn.

### [Blog 2 - Ngăn chặn rò rỉ dữ liệu trên AWS bằng Egress Controls cho Cloud Workloads](3.2-Blog2/)

Bài blog này giới thiệu khái niệm **Data Exfiltration (Rò rỉ dữ liệu)** và giải thích tầm quan trọng của việc kiểm soát lưu lượng truy cập đi ra (Egress Traffic) trong bảo mật đám mây. Đồng thời, bài viết trình bày cách kết hợp các dịch vụ như AWS Network Firewall, Amazon Route 53 Resolver DNS Firewall, Amazon GuardDuty, AWS Security Hub và Data Perimeter để xây dựng kiến trúc bảo mật nhiều lớp, giúp phát hiện và ngăn chặn các hành vi rò rỉ dữ liệu trái phép.

### [Blog 3 - Tự động hóa quản lý danh tính và tăng cường bảo mật với AWS Directory Service APIs](3.3-Blog3/)

Bài blog này tìm hiểu về **AWS Directory Service Data APIs** dành cho AWS Managed Microsoft Active Directory. Nội dung trình bày cách doanh nghiệp có thể tự động hóa toàn bộ vòng đời quản lý danh tính (Identity Lifecycle) và quy trình phản ứng bảo mật bằng cách kết hợp các dịch vụ như Amazon GuardDuty, Amazon EventBridge, AWS Step Functions, AWS Systems Manager và Amazon SNS. Bài viết cũng làm nổi bật những lợi ích của việc tự động hóa trong việc nâng cao hiệu quả quản trị và tăng cường bảo mật hệ thống.