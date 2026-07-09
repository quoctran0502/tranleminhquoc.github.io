---
title: "Worklog Tuần 4"
date: 2024-01-01
weight: 1
chapter: false
pre: " <b> 1.4. </b> "
---
### Mục tiêu tuần 4:

* Nghiên cứu và thực hành chuyên sâu về lưu trữ và cấu hình máy chủ ảo trên AWS thông qua các dịch vụ Amazon EBS, Snapshot, Amazon AMI, Elastic IP và cơ chế EC2 User Data.

### Các công việc cần triển khai trong tuần này:
| Thứ | Công việc                                                                                                                                                                                   | Ngày bắt đầu | Ngày hoàn thành | Nguồn tài liệu                            |
| --- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------ | --------------- | ----------------------------------------- |
| 2   | - **Nghiên cứu Amazon EBS** <br>&emsp; + Khái niệm Block Storage. <br>&emsp; + Tạo EBS Volume. <br>&emsp; + Attach Volume về EC2. <br>&emsp; + Kiểm tra trạng thái lưu trữ. | 11/05/2026   | 11/05/2026      | <https://aws.amazon.com/vi/ebs/> |
| 3   | - **Tìm hiểu Snapshot và Backup trên AWS** <br>&emsp; + Tạo EBS Snapshot. <br>&emsp; + Khôi phục Volume từ Snapshot. <br>&emsp; + So sánh Snapshot và AMI. <br>&emsp; + Tìm hiểu ứng dụng sao lưu dữ liệu. | 12/05/2026   | 12/05/2026      | <https://000004.awsstudygroup.com/5-amazonec2basic/5.2-createec2snapshot/> |
| 4   | - **Nghiên cứu Amazon AMI** <br>&emsp; + Tìm hiểu Amazon Machine Image. <br>&emsp; + Tạo AMI từ EC2. <br>&emsp; + Khởi tạo EC2 từ AMI. <br>&emsp; + Kiểm tra Image sau khi tạo. | 13/05/2026   | 13/05/2026      | <https://000004.awsstudygroup.com/5-amazonec2basic/5.3-createcustomami/> |
| 5   | - **Tìm hiểu Elastic IP** <br>&emsp; + Public IP và Elastic IP. <br>&emsp; + Gán Elastic IP cho EC2. <br>&emsp; + Kiểm tra kết nối. | 14/05/2026   | 14/05/2026      | <https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html> |
| 6   | - **Nghiên cứu EC2 User Data** <br>&emsp; + Cơ chế User Data. <br>&emsp; + Tự động cài Apache. <br>&emsp; + Kiểm tra kết quả triển khai. | 15/05/2026   | 15/05/2026      | <https://000004.awsstudygroup.com/5-amazonec2basic/5.6-keypair-user-data-linux/> |


### Kết quả đạt được tuần 4:

* Hiểu rõ khái niệm Block Storage và cách hoạt động của Amazon EBS.

* Thực hành tạo mới, gắn kết (attach) và kiểm tra trạng thái lưu trữ của EBS Volume trên máy chủ EC2.

* Nắm vững cơ chế Snapshot và Backup: tạo thành công EBS Snapshot, khôi phục Volume từ Snapshot và so sánh sự khác biệt giữa Snapshot với AMI.

* Nghiên cứu Amazon AMI: tạo AMI từ một EC2 đang hoạt động và khởi tạo thành công máy chủ mới từ AMI đó.

* Hiểu rõ cơ chế của Elastic IP (Public IP tĩnh), thực hành gán Elastic IP cho EC2 và kiểm tra kết nối.

* Nghiên cứu cơ chế EC2 User Data và thực hành cấu hình tự động cài đặt máy chủ web Apache khi khởi chạy máy chủ.


