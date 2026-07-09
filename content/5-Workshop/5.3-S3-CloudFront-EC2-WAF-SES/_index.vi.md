---
title : "Triển khai Frontend & Backend (S3, CloudFront, EC2, WAF, SES)"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

Trong phần này, chúng ta sẽ thực hiện triển khai toàn bộ ứng dụng (bao gồm giao diện Frontend tĩnh và máy chủ Backend Spring Boot) lên AWS. Các dịch vụ được tích hợp bao gồm: **S3** (lưu trữ Frontend), **CloudFront** (phân phối nội dung CDN), **EC2** (chạy Backend), **WAF** (bảo vệ ứng dụng) và **SES** (gửi email thông báo).

---

### A. CẤU HÌNH AMAZON S3 (LƯU TRỮ FRONTEND TĨNH)

1. Đăng nhập vào **AWS Console**, chọn dịch vụ **S3** -> chọn **Create bucket**.
2. Cấu hình thông tin cơ bản:
   - **Bucket name**: `caulongvui-static`
   - **Region**: Lựa chọn khu vực bạn muốn (ví dụ: `ap-southeast-1` - Singapore).
3. Cấu hình bảo mật:
   - Tại mục **Block Public Access settings for this bucket**: **Tắt tất cả** (Uncheck "Block all public access") để cho phép truy cập công cộng vào các tệp tĩnh.
4. Bấm **Create bucket**.
5. Kích hoạt tính năng **Static website hosting**:
   - Truy cập vào bucket `caulongvui-static` -> Chọn tab **Properties**.
   - Cuộn xuống dưới cùng tìm mục **Static website hosting** -> Chọn **Edit**.
   - Chọn **Enable**.
   - Điền cấu hình:
     - **Index document**: `index.html`
     - **Error document**: `index.html`
   - Nhấn **Save changes**.
6. Cấu hình **Bucket Policy** để cấp quyền đọc công cộng cho các đối tượng:
   - Chuyển sang tab **Permissions** -> mục **Bucket policy** -> Chọn **Edit**.
   - Dán đoạn mã Policy dưới đây vào:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "PublicReadGetObject",
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::caulongvui-static/*"
             }
         ]
     }
     ```
   - Nhấn **Save changes**.

---

### B. CẤU HÌNH CLOUDFRONT (PHÂN PHỐI CDN)

1. Truy cập dịch vụ **CloudFront** trên AWS Console -> chọn **Create distribution**.
2. Cấu hình **Origin Settings**:
   - **Origin domain**: Chọn endpoint của S3 Static Website Hosting đã tạo ở trên (dạng: `caulongvui-static.s3-website-ap-southeast-1.amazonaws.com`).
     > [!TIP]
     > Hãy dùng link S3 website endpoint thay vì chọn bucket S3 trực tiếp để hỗ trợ client-side routing chính xác hơn.
3. Các cài đặt còn lại (Bước 4, 5) giữ nguyên mặc định -> nhấn **Create distribution**.
4. Chờ trạng thái của CloudFront chuyển sang `Deployed` (thường mất 3-5 phút).
5. Copy địa chỉ **Distribution domain name** được cấp (ví dụ: `d33wcfjdygo2at.cloudfront.net`).
6. Cập nhật cấu hình ứng dụng:
   - **Frontend**: Mở file `config.js` trong mã nguồn và cập nhật các biến:
     - `apiBase`: Đường dẫn API (ví dụ: `https://d33wcfjdygo2at.cloudfront.net/api`)
     - `cdnUrl`: `https://d33wcfjdygo2at.cloudfront.net`
   - **Backend**: Mở file `application.properties` và cấu hình:
     - `app.cdn-url=https://d33wcfjdygo2at.cloudfront.net`
     - `app.frontend-url=https://d33wcfjdygo2at.cloudfront.net`
7. **Đẩy mã nguồn Frontend lên S3**:
   - Mở terminal tại thư mục gốc của dự án và chạy lệnh sau để đồng bộ mã nguồn tĩnh lên bucket S3:
     ```bash
     aws s3 sync src/main/resources/static/ s3://caulongvui-static/ --delete --region ap-southeast-1
     ```
8. **Cập nhật Redirect URL trong AWS Cognito**:
   - Truy cập Cognito User Pool của bạn -> App Client -> Cấu hình lại mục **Allowed callback URLs** thành địa chỉ CloudFront:
     `https://d33wcfjdygo2at.cloudfront.net/login/oauth2/code/cognito`
   - Cập nhật lại các mục **Allowed sign-out URLs** tương ứng với domain CloudFront.
9. **Cấu hình Behavior cho CloudFront**:
   - Để chuyển tiếp các request API và Login tới máy chủ EC2 Backend, ta cấu hình thêm **Behaviors** trong CloudFront:
     - Nhấp chọn **Create behavior**.
     - Cấu hình Behavior cho API:
       - **Path pattern**: `/api/*`
       - **Origin**: Chọn Origin trỏ tới EC2 Backend của bạn.
       - **Viewer protocol policy**: Chọn `Redirect HTTP to HTTPS`.
       - **Allowed HTTP methods**: Chọn tất cả phương thức (`GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE`).
     - Thực hiện tạo thêm 2 Behaviors tương tự cho các đường dẫn:
       - `/oauth2/*`
       - `/login/*`

---

### C. TRIỂN KHAI BACKEND LÊN MÁY ẢO EC2

#### 1. Khởi tạo máy ảo EC2
1. Truy cập dịch vụ **EC2** -> chọn **Launch instance**.
2. Thiết lập cấu hình:
   - **Name**: `caulongvui-backend`
   - **AMI**: Amazon Linux 2023
   - **Instance type**: `t3.micro` (hoặc `t2.micro` tùy thuộc Free Tier).
   - **Key pair**: Tạo key mới đặt tên là `caulongvui-key` và tải tệp `.pem` về máy cá nhân.
   - **Network settings**: Chọn VPC `caulongvui-vpc` và gán Security Group `caulongvui-backend-sg` đã tạo ở Bước 5.2.
3. Bấm **Launch instance**.

#### 2. Kết nối và cài đặt môi trường trên EC2
1. Mở PowerShell hoặc Terminal trên máy tính cá nhân của bạn, phân quyền cho file key (nếu ở Linux/macOS) và thực hiện kết nối SSH:
   ```bash
   ssh -i "C:\path\to\caulongvui-key.pem" ec2-user@<Elastic_IP_cua_ban>
   ```
2. Cập nhật hệ thống Amazon Linux:
   ```bash
   sudo dnf update -y
   ```
3. Cài đặt Java 17:
   ```bash
   sudo dnf install java-17-amazon-corretto-headless -y
   ```
4. Build ứng dụng Spring Boot ở máy cá nhân ra file JAR:
   ```bash
   # Chạy lệnh tại thư mục gốc dự án ở local
   .\mvnw clean package -DskipTests
   ```
5. Đẩy tệp JAR từ máy cá nhân lên EC2 bằng lệnh SCP:
   ```bash
   scp -i "C:\path\to\caulongvui-key.pem" target/CauLongVui-0.0.1-SNAPSHOT.jar ec2-user@<Elastic_IP_cua_ban>:/home/ec2-user/CauLongVui-0.0.1-SNAPSHOT.jar
   ```

---

### D. TÍCH HỢP BẢO VỆ AWS WAF

Để tăng cường bảo mật cho ứng dụng web mà không làm gián đoạn trải nghiệm người dùng trong quá trình cấu hình ban đầu:
- Liên kết **AWS WAF** với CloudFront Distribution đã tạo.
- Bật WAF ở chế độ **Giám sát (Monitoring/Count mode)** đối với các quy tắc chặn để tránh việc hệ thống vô tình chặn các request kiểm thử của lập trình viên.

---

### E. CẤU HÌNH DỊCH VỤ GỬI EMAIL (AMAZON SES)

*Sau khi deploy Backend lên EC2 thành công, thực hiện cấu hình SES như sau:*

#### Bước 1: Khai báo Region gửi mail
Trong tệp `application.properties` của ứng dụng Spring Boot, khai báo Region sử dụng SES (khuyên dùng Singapore):
```properties
app.mail.ses.region=ap-southeast-1
```

#### Bước 2: Xác thực (Verify) email gửi đi trong SES
1. Trên AWS Console, tìm kiếm dịch vụ **Amazon SES** -> chọn **Identities**.
2. Nhấn nút **Create identity**.
3. Chọn **Identity type** là `Email address`.
4. Nhập địa chỉ email của bạn gửi đi (ví dụ: `ngocchau0845@gmail.com`).
5. Bấm **Create identity**.
6. Truy cập hòm thư Gmail của bạn, tìm email xác nhận từ AWS và nhấn vào link liên kết để hoàn thành xác thực.
7. Trạng thái trong tab Identities sẽ chuyển sang **Verified**.

#### Bước 3: Xác thực email người nhận (Nếu SES ở chế độ Sandbox)
> [!IMPORTANT]
> Khi tài khoản SES mới tạo và nằm trong môi trường thử nghiệm (Sandbox), bạn chỉ có thể gửi email đến các địa chỉ email người nhận đã được xác thực trước.
- Thực hiện lại các thao tác ở Bước 2 cho địa chỉ email người nhận (ví dụ: `test@gmail.com`) để chạy thử tính năng.

#### Bước 4: Tạo Policy phân quyền SES cho IAM User
1. Truy cập dịch vụ **IAM** -> **Users** -> Chọn User đang chạy ứng dụng.
2. Chọn **Add permissions** -> **Create inline policy**.
3. Chọn tab JSON và dán đoạn policy sau:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ses:SendEmail",
                   "ses:SendRawEmail"
               ],
               "Resource": "*"
           }
       ]
   }
   ```
4. Đặt tên cho policy (ví dụ: `caulongvui-ses-policy`) và lưu lại.

#### Bước 5: Cấu hình IAM Role và cấp quyền cho EC2 Backend
1. Truy cập **IAM** -> **Roles** -> Tìm kiếm role đang gắn cho EC2 backend (ví dụ: `AmazonS3FullAccess-caulongvui`).
2. Tại tab **Permissions**, nhấp chọn **Add permissions** -> **Attach policies**.
3. Tìm kiếm policy `AmazonSESFullAccess` và `AmazonS3FullAccess`. Tích chọn cả hai và nhấn **Add permissions**.
4. Quay lại trang quản trị **EC2** -> **Instances** -> Chọn instance `caulongvui-backend`.
5. Chọn **Actions** -> **Security** -> **Modify IAM role**.
6. Chọn role `AmazonS3FullAccess-caulongvui` và nhấn **Update IAM role**.

---

### F. VẬN HÀNH ỨNG DỤNG TRÊN BACKEND EC2

1. Kết nối vào máy ảo backend bằng cách chọn instance `caulongvui-backend` -> Nhấn **Connect** -> Chọn tab **SSM Session Manager** để mở terminal trực tiếp trên trình duyệt.
2. Dừng ứng dụng cũ nếu đang chạy:
   ```bash
   sudo pkill -f CauLongVui-0.0.1-SNAPSHOT.jar
   ```
3. Chạy ứng dụng dưới dạng tiến trình nền (background process) bằng lệnh `nohup` kèm các biến môi trường cấu hình SES:
   ```bash
   nohup java -jar /home/ec2-user/CauLongVui-0.0.1-SNAPSHOT.jar \
     --app.mail.enabled=true \
     --app.mail.from="ngocchau0845@gmail.com" \
     --app.mail.ses.region="ap-southeast-1" \
     --app.mail.booking-reminder-hours=12 \
     > /tmp/caulongvui.log 2>&1 &
   ```
