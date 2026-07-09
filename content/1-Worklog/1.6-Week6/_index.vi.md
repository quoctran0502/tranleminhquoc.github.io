---
title: "Worklog Tuần 6"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 1.6. </b> "
---
### Mục tiêu tuần 6:

* Tìm hiểu kiến trúc mạng cơ bản trên AWS, thực hành tạo Virtual Private Cloud (VPC), phân chia Subnet, cấu hình Security Group, kết nối Internet Gateway và định tuyến lưu lượng thông qua Route Table.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - **Nghiên cứu tổng quan về Amazon VPC** <br>&emsp; + Tìm hiểu khái niệm Virtual Private Cloud. <br>&emsp; + Tìm hiểu Public Subnet và Private Subnet. <br>&emsp; + Nghiên cứu kiến trúc mạng cơ bản trên AWS. | 25/05/2026   | 25/05/2026      | <https://000003.awsstudygroup.com/1-introduce/> |
| 3   | - **Thực hành tạo VPC** <br>&emsp; + Tạo VPC mới trên AWS. <br>&emsp; + Cấu hình CIDR Block. <br>&emsp; + Kiểm tra thông tin và trạng thái VPC. <br>&emsp; + Làm quen với giao diện quản lý VPC. | 26/05/2026   | 26/05/2026      | <https://000003.awsstudygroup.com/3-prerequisite/3.1-createvpc/> |
| 4   | - **Thực hành tạo Subnet và Security Group** <br>&emsp; + Tạo Public Subnet. <br>&emsp; + Cấu hình Security Group cơ bản. <br>&emsp; + Thiết lập các quy tắc Inbound và Outbound. <br>&emsp; + Kiểm tra khả năng truy cập tài nguyên. | 27/05/2026   | 27/05/2026      | <https://000003.awsstudygroup.com/3-prerequisite/3.2-createsubnet/> <br> <https://000003.awsstudygroup.com/3-prerequisite/3.5-createsecuritygroup/> |
| 5   | - **Tìm hiểu Internet Gateway** <br>&emsp; + Tạo Internet Gateway. <br>&emsp; + Kết nối Internet Gateway với VPC. <br>&emsp; + Kiểm tra khả năng truy cập Internet. <br>&emsp; + Đánh giá kết quả cấu hình. | 28/05/2026   | 28/05/2026      | <https://000003.awsstudygroup.com/3-prerequisite/3.3-createigw/> |
| 6   | - **Tìm hiểu Route Table** <br>&emsp; + Tạo Route Table. <br>&emsp; + Cấu hình định tuyến cho Subnet. <br>&emsp; + Liên kết Route Table với Subnet. <br>&emsp; + Kiểm tra hoạt động của hệ thống mạng. | 29/05/2026   | 29/05/2026      | <https://000003.awsstudygroup.com/3-prerequisite/3.4-createroutetable/> |


### Kết quả đạt được tuần 6:

* Hiểu rõ khái niệm và cấu trúc của Amazon VPC, phân biệt được Public Subnet và Private Subnet.

* Khởi tạo thành công VPC mới trên AWS và cấu hình dải CIDR Block phù hợp.

* Thực hành tạo Public Subnet và thiết lập Security Group với các quy tắc Inbound/Outbound để kiểm soát truy cập tài nguyên.

* Cấu hình Internet Gateway, kết nối thành công với VPC để cho phép tài nguyên trong mạng kết nối với Internet.

* Tạo và cấu hình Route Table, thiết lập các quy tắc định tuyến cho Subnet và kiểm tra thành công hoạt động của hệ thống mạng.



