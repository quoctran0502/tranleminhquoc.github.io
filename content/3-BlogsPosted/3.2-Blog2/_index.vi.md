---
title: "Blog 2 - Ngăn chặn rò rỉ dữ liệu trên AWS bằng Egress Controls cho Cloud Workloads"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
---

## Giới thiệu

Khi triển khai hệ thống trên AWS, nhiều doanh nghiệp tập trung bảo vệ lưu lượng truy cập đi vào (Ingress Traffic) thông qua Security Groups, AWS WAF hoặc IAM. Tuy nhiên, lưu lượng đi ra khỏi hệ thống (Egress Traffic) cũng là một thành phần quan trọng trong chiến lược bảo mật.

Nếu một máy chủ hoặc ứng dụng bị xâm nhập, kẻ tấn công có thể lợi dụng các kết nối đi ra để đánh cắp dữ liệu, tải thêm mã độc hoặc thiết lập kết nối điều khiển từ xa. Đây chính là hành vi **Data Exfiltration (Rò rỉ dữ liệu)**.

Trong bài viết này, chúng ta sẽ tìm hiểu cách AWS sử dụng các cơ chế **Egress Controls** để giúp doanh nghiệp ngăn chặn và phát hiện các hành vi rò rỉ dữ liệu trên môi trường đám mây.

---

# Data Exfiltration là gì?

Data Exfiltration là hành vi truyền dữ liệu ra khỏi hệ thống mà không được phép.

Một số ví dụ phổ biến:

- Máy chủ Amazon EC2 bị khai thác lỗ hổng bảo mật.
- Hacker cài đặt mã độc và gửi dữ liệu khách hàng ra máy chủ bên ngoài.
- AI Agent bị Prompt Injection và gửi dữ liệu nhạy cảm tới dịch vụ của bên thứ ba.

Nếu không có cơ chế kiểm soát Egress Traffic, các hành vi này rất khó được phát hiện trong giai đoạn đầu.

---

# Vì sao Egress Control lại quan trọng?

Sau khi xâm nhập thành công vào hệ thống, mục tiêu tiếp theo của kẻ tấn công thường là:

- Đánh cắp dữ liệu.
- Thiết lập Command and Control (C2).
- Tải thêm mã độc.
- Truyền dữ liệu ra ngoài tổ chức.

Kiểm soát lưu lượng đi ra giúp giảm thiểu thiệt hại ngay cả khi hệ thống đã bị xâm nhập.

---

# Các dịch vụ AWS hỗ trợ Egress Control

## AWS Network Firewall

AWS Network Firewall là dịch vụ tường lửa được quản lý hoàn toàn bởi AWS.

Các tính năng nổi bật:

- Chặn các tên miền không được phép.
- Kiểm soát địa chỉ IP và cổng kết nối.
- Phát hiện IDS/IPS.
- Chặn truy cập theo khu vực địa lý.
- TLS Inspection.
- Threat Intelligence.

---

## Amazon Route 53 Resolver DNS Firewall

DNS Firewall giúp ngăn chặn các kỹ thuật DNS Tunneling.

Các chức năng:

- Chặn domain độc hại.
- Allow List.
- Phát hiện Domain Generation Algorithm (DGA).
- Phát hiện DNS Tunneling bằng AI và Machine Learning.

---

## Data Perimeter

Data Perimeter giúp bảo vệ dữ liệu ở cấp độ API.

Bao gồm:

- Service Control Policies (SCPs)
- Resource Control Policies (RCPs)
- IAM Policies
- Resource Policies
- VPC Endpoint Policies

Mục tiêu là chỉ cho phép người dùng, tài nguyên và mạng đáng tin cậy truy cập dữ liệu.

---

# Các dịch vụ phát hiện rò rỉ dữ liệu

## Amazon GuardDuty

GuardDuty sử dụng Machine Learning và Threat Intelligence để phát hiện:

- DNS Data Exfiltration.
- Kết nối tới tên miền độc hại.
- Command and Control.
- Truy cập bất thường tới Amazon S3.
- Hoạt động từ địa chỉ IP độc hại.

---

## AWS Security Hub

AWS Security Hub tập trung cảnh báo từ:

- Amazon GuardDuty
- IAM Access Analyzer
- AWS Config
- AWS Firewall Manager

Giúp đội ngũ bảo mật theo dõi toàn bộ hệ thống từ một giao diện duy nhất.

---

## IAM Access Analyzer

IAM Access Analyzer giúp phát hiện:

- Tài nguyên bị public ngoài ý muốn.
- Quyền truy cập vượt mức cần thiết.
- Chính sách chia sẻ dữ liệu không an toàn.

---

# Liên hệ với các hệ thống AI

Các AI Agent hiện đại có khả năng:

- Truy cập cơ sở dữ liệu.
- Gọi API.
- Thực thi mã nguồn.
- Thực hiện quy trình tự động.

Nếu bị Prompt Injection hoặc Goal Hijacking, AI Agent hoàn toàn có thể trở thành công cụ đánh cắp dữ liệu.

AWS khuyến nghị áp dụng các cơ chế Egress Control cho cả ứng dụng AI và các ứng dụng truyền thống.

---

# Bài học rút ra

Qua bài viết, mình nhận thấy rằng bảo mật không chỉ là ngăn chặn truy cập từ bên ngoài mà còn phải kiểm soát dữ liệu rời khỏi hệ thống.

Một chiến lược bảo mật hiệu quả nên bao gồm:

- Kiểm soát Egress Traffic.
- Giới hạn Domain và IP được phép.
- Áp dụng nguyên tắc Least Privilege.
- Giám sát liên tục bằng GuardDuty và Security Hub.
- Thiết lập Data Perimeter.

---

# Kết luận

Data Exfiltration là một trong những rủi ro bảo mật quan trọng trên môi trường Cloud.

Bằng cách kết hợp AWS Network Firewall, Route 53 Resolver DNS Firewall, GuardDuty, Security Hub và Data Perimeter, doanh nghiệp có thể xây dựng kiến trúc phòng thủ nhiều lớp nhằm giảm thiểu nguy cơ rò rỉ dữ liệu.

Đây là chủ đề rất hữu ích đối với những ai đang tìm hiểu AWS Security, DevSecOps và các hệ thống AI trên AWS.

---

## Kiến trúc tham khảo

Kiến trúc dưới đây minh họa mô hình nhiều lớp mà AWS đề xuất để ngăn chặn và phát hiện các hành vi **Data Exfiltration**. Kiến trúc kết hợp các dịch vụ như AWS Network Firewall, Amazon Route 53 Resolver DNS Firewall, Data Perimeter, Amazon GuardDuty, AWS Security Hub và Amazon CloudWatch nhằm tăng cường khả năng bảo vệ lưu lượng đi ra (Egress Traffic).

![AWS Reference Architecture for Preventing Data Exfiltration](/images/3-Blog/aws-reference-architecture-for-preventing-data-exfiltration.jpg)

*Hình 2. AWS Reference Architecture for Preventing Data Exfiltration.*

---

# Tham khảo

https://aws.amazon.com/vi/blogs/security/prevent-data-exfiltration-aws-egress-controls-for-cloud-workloads/