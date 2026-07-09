---
title: "Blog 1 - Tìm hiểu kiến trúc lưu trữ Binary Assets với Lore trên AWS"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

Trong quá trình phát triển game hoặc các dự án truyền thông số, việc quản lý các **Binary Assets** như hình ảnh, âm thanh, video, mô hình 3D hay animation luôn là một thách thức lớn. Những tệp này thường có kích thước lớn, được cập nhật liên tục và yêu cầu hệ thống lưu trữ vừa có khả năng mở rộng, vừa tối ưu chi phí.

Bài viết này giới thiệu cách **Lore** kết hợp các dịch vụ của AWS để xây dựng một hệ thống lưu trữ binary assets hiệu quả hơn.

---

# Vấn đề của phương pháp lưu trữ truyền thống

Đối với mã nguồn, các hệ thống quản lý phiên bản như Git hoạt động rất hiệu quả. Tuy nhiên, khi áp dụng cho các tệp nhị phân (Binary Assets), mỗi lần chỉnh sửa hệ thống thường phải lưu lại toàn bộ tệp mới thay vì chỉ lưu phần thay đổi.

Điều này dẫn đến nhiều hạn chế:

- Dung lượng lưu trữ tăng nhanh.
- Chi phí lưu trữ ngày càng lớn.
- Quá trình đồng bộ dữ liệu mất nhiều thời gian.
- Khó mở rộng khi số lượng asset tăng lên.

Đối với các dự án game hoặc media có hàng nghìn asset được cập nhật mỗi ngày, đây là một vấn đề rất đáng quan tâm.

---

# Lore giải quyết vấn đề như thế nào?

Lore sử dụng mô hình **Content-Addressed Storage (CAS)**.

Thay vì lưu toàn bộ file sau mỗi lần thay đổi, Lore chia dữ liệu thành nhiều phần nhỏ (Fragments). Mỗi fragment sẽ được gán một mã băm (Hash) duy nhất.

Khi người dùng chỉnh sửa asset, hệ thống chỉ lưu những fragment mới thay đổi, còn các fragment đã tồn tại sẽ được tái sử dụng.

Nhờ đó Lore mang lại nhiều lợi ích:

- Giảm dữ liệu trùng lặp.
- Tiết kiệm đáng kể dung lượng lưu trữ.
- Đồng bộ dữ liệu nhanh hơn.
- Quản lý nhiều phiên bản asset hiệu quả.

---

# Các dịch vụ AWS trong kiến trúc Lore

Kiến trúc tham chiếu của Lore được xây dựng từ nhiều dịch vụ AWS, mỗi dịch vụ đảm nhiệm một vai trò riêng.

## Amazon S3

Amazon S3 được sử dụng làm nơi lưu trữ chính cho toàn bộ Binary Assets.

Ưu điểm:

- Độ bền dữ liệu rất cao.
- Khả năng mở rộng gần như không giới hạn.
- Chi phí lưu trữ tối ưu.
- Phù hợp với Object Storage.

---

## Amazon EC2

Amazon EC2 triển khai các **Edge Pods**, có nhiệm vụ:

- Tiếp nhận yêu cầu từ người dùng.
- Cache dữ liệu được truy cập thường xuyên.
- Tăng tốc độ đọc và ghi dữ liệu.

---

## Amazon ECS

Amazon ECS vận hành Backend của Lore.

Các chức năng chính gồm:

- Xử lý dữ liệu mới được tải lên.
- Phát hiện dữ liệu trùng lặp.
- Điều phối việc ghi dữ liệu vào hệ thống lưu trữ.

---

## Amazon DynamoDB

Amazon DynamoDB lưu trữ Metadata của Repository.

Bao gồm:

- Thông tin Asset.
- Phiên bản dữ liệu.
- Danh sách Fragment.
- Thông tin Repository và Branch.

Nhờ khả năng mở rộng tự động cùng độ trễ thấp, DynamoDB đáp ứng tốt việc truy cập metadata với tần suất lớn.

---

## AWS Cloud Map

AWS Cloud Map giúp các thành phần trong hệ thống có thể tự động khám phá (Service Discovery) và giao tiếp với nhau trong môi trường phân tán.

---

# Lợi ích của kiến trúc

Việc kết hợp các dịch vụ AWS giúp Lore mang lại nhiều lợi ích:

- Giảm đáng kể chi phí lưu trữ nhờ loại bỏ dữ liệu trùng lặp.
- Tăng tốc độ đồng bộ dữ liệu giữa các thành viên.
- Dễ dàng mở rộng khi số lượng Binary Assets tăng lên.
- Tận dụng độ tin cậy và khả năng mở rộng của AWS.
- Giảm công sức quản trị hạ tầng nhờ sử dụng các dịch vụ được AWS quản lý.

---

# Bài học rút ra

Qua bài viết này, mình hiểu rõ hơn rằng việc lựa chọn kiến trúc lưu trữ phù hợp ảnh hưởng rất lớn đến hiệu năng cũng như chi phí vận hành của hệ thống.

Lore không chỉ là một giải pháp lưu trữ Binary Assets mà còn là ví dụ điển hình về cách kết hợp nhiều dịch vụ AWS như **Amazon S3**, **Amazon EC2**, **Amazon ECS**, **Amazon DynamoDB** và **AWS Cloud Map** để giải quyết một bài toán thực tế trong ngành công nghiệp game.

Đồng thời, bài viết cũng giúp mình hiểu rõ hơn vai trò của từng dịch vụ AWS trong việc xây dựng một hệ thống lưu trữ có khả năng mở rộng và hiệu quả.

---

# Kết luận

Lore mang đến một cách tiếp cận mới trong việc quản lý Binary Assets bằng cách chỉ lưu trữ những phần dữ liệu thực sự thay đổi thay vì lưu toàn bộ tệp sau mỗi lần cập nhật.

Khi kết hợp với các dịch vụ của AWS như **Amazon S3**, **Amazon EC2**, **Amazon ECS**, **Amazon DynamoDB** và **AWS Cloud Map**, hệ thống có thể đạt được khả năng mở rộng cao, tối ưu chi phí lưu trữ và cải thiện hiệu suất làm việc của các nhóm phát triển.

Đây là một kiến trúc đáng tham khảo cho các doanh nghiệp trong lĩnh vực game, truyền thông số hoặc bất kỳ tổ chức nào cần quản lý khối lượng lớn Binary Assets trên nền tảng đám mây.

---

## Kiến trúc tham khảo

Kiến trúc dưới đây minh họa cách **Lore** triển khai hệ thống lưu trữ binary assets trên AWS. Dữ liệu được tiếp nhận thông qua các Edge Pods, định tuyến bằng AWS Cloud Map, xử lý bởi Amazon ECS và lưu trữ trên Amazon S3 cùng Amazon DynamoDB. Kiến trúc này giúp tối ưu hiệu năng, giảm dữ liệu trùng lặp và hỗ trợ khả năng mở rộng cho các dự án có khối lượng binary assets lớn.

![AWS Reference Architecture for Binary Asset Storage with Lore](/images/3-Blog/aws-reference-architecture-for-binary-asset-storage-with-lore.jpg)

*Hình 1. AWS Reference Architecture for Binary Asset Storage with Lore.*

---

# Tham khảo

https://aws.amazon.com/vi/blogs/gametech/how-lore-rethinks-binary-asset-storage-on-aws/