---
title : "Cấu hình Auto Scaling cho Backend"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

Trong phần này, chúng ta sẽ thiết lập hệ thống tự động co giãn (**Auto Scaling**) cho máy chủ Spring Boot Backend của ứng dụng. Điều này giúp đảm bảo tính sẵn sàng cao (High Availability), tự động thay thế khi máy chủ gặp sự cố và giảm thiểu thời gian ngừng hoạt động (Downtime).

---

### A. TẠI SAO CẦN AUTO SCALING?

- **Đáp ứng tải linh hoạt**: Khi lượng người dùng truy cập tăng đột biến, một máy ảo EC2 đơn lẻ sẽ không đủ khả năng xử lý. Auto Scaling Group (ASG) giúp tự động sinh thêm EC2 để phân phối tải.
- **Tự động khôi phục lỗi (Self-Healing)**: Khi một máy chủ EC2 bị lỗi phần cứng hoặc ứng dụng bên trong bị đứng (Unhealthy), ASG sẽ tự động hủy instance lỗi đó và khởi tạo một instance mới khỏe mạnh thay thế.
- **Tiết kiệm chi phí**: Tự động giảm số lượng máy chủ khi lưu lượng truy cập thấp (ví dụ ban đêm).

#### Mô hình triển khai Auto Scaling cho dự án:
- **Backend**: Triển khai trên nhiều máy ảo **EC2** nằm trong **Auto Scaling Group (ASG)** và phân phối tải thông qua **Application Load Balancer (ALB)**.
- **Frontend**: Lưu trữ tĩnh trên **Amazon S3** và phân phối qua **Amazon CloudFront** CDN.
- **Database**: Sử dụng **Amazon RDS** (RDS tự động co giãn dung lượng ổ đĩa, không nằm trong ASG của EC2).

---

### B. CẤU HÌNH HEALTH ENDPOINT TRONG SPRING BOOT

Để Application Load Balancer và Auto Scaling Group có thể kiểm tra xem ứng dụng Spring Boot đang chạy bình thường hay đã bị lỗi (dựa trên cơ chế Health Check), ta cần kích hoạt Spring Boot Actuator.

1. Thêm dependency Actuator vào tệp `pom.xml` của dự án:
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. Thêm các thuộc tính sau vào tệp cấu hình `application.properties`:
   ```properties
   management.endpoints.web.exposure.include=health,info
   management.endpoint.health.probes.enabled=true
   ```
   > [!NOTE]
   > Sau khi cấu hình, ứng dụng sẽ cung cấp một API kiểm tra sức khỏe tại đường dẫn `/actuator/health`. ALB và Auto Scaling Group sẽ sử dụng endpoint này để xác minh tình trạng ứng dụng.

---

### C. CÁC BƯỚC THIẾT LẬP AUTO SCALING TRÊN AWS

#### 1. Tạo Launch Template (Bản mẫu khởi chạy EC2)
*Launch Template định nghĩa cấu hình của máy ảo EC2 sẽ được tự động tạo ra khi có nhu cầu co giãn.*

1. Vào **EC2 Console** -> **Launch Templates** -> Chọn **Create launch template**.
2. Thiết lập cấu hình cơ bản:
   - **Launch template name**: `caulongvui-template`
   - **Auto Scaling Guidance**: Tick chọn ô *Provide guidance to help me set up a template that can be used with EC2 Auto Scaling*.
   - **Application and OS Images (AMI)**: Chọn **Amazon Linux 2023** (không nên chọn Windows cho trường hợp này để tối ưu tốc độ khởi động và chi phí).
   - **Instance type**: Chọn `t3.micro` hoặc `t3.small` (tùy thuộc vào tải trung bình mong muốn).
   - **Key pair**: Chọn `caulongvui-key`.
   - **Security groups**: Chọn `caulongvui-backend-sg`.
3. Cuộn xuống phần **Advanced details** -> mục **User data**, dán đoạn script sau để máy tự động cài đặt môi trường và chạy ứng dụng Spring Boot khi khởi tạo:
   ```bash
   #!/bin/bash
   set -e

   # Cập nhật hệ thống
   yum update -y
   # Cài đặt Java 21 và AWS CLI để tải file JAR từ S3
   yum install -y java-21-amazon-corretto awscli

   # Tạo thư mục và tải tệp JAR ứng dụng từ S3
   mkdir -p /home/ec2-user
   cd /home/ec2-user
   aws s3 cp s3://caulongvui-static/deploy/CauLongVui-0.0.1-SNAPSHOT.jar /home/ec2-user/app.jar

   # Thiết lập Spring Boot như một systemd service chạy ngầm
   cat >/etc/systemd/system/caulongvui.service <<'EOF'
   [Unit]
   Description=CauLongVui Spring Boot App
   After=network.target

   [Service]
   User=ec2-user
   WorkingDirectory=/home/ec2-user
   ExecStart=/usr/bin/java -jar /home/ec2-user/app.jar
   Restart=always
   RestartSec=10

   [Install]
   WantedBy=multi-user.target
   EOF

   # Tải lại cấu hình daemon, kích hoạt và chạy service
   systemctl daemon-reload
   systemctl enable caulongvui
   systemctl start caulongvui
   ```
4. Nhấn **Create launch template**.

---

#### 2. Tạo Target Group (Nhóm đích nhận traffic)
*Target Group gom nhóm các EC2 instance lại để Load Balancer chuyển hướng lưu lượng vào.*

1. Vào **EC2 Console** -> **Target Groups** -> Chọn **Create target group**.
2. Cấu hình thông số:
   - **Choose a target type**: Chọn **Instances**.
   - **Target group name**: `caulongvui-backend-tg`
   - **Protocol & Port**: Chọn **HTTP** và Port `8080` (vì Spring Boot đang chạy trực tiếp trên cổng này).
   - **VPC**: Chọn `caulongvui-vpc`.
   - **Health check settings**:
     - **Health check protocol**: `HTTP`
     - **Health check path**: `/actuator/health` (sử dụng Endpoint Actuator đã tạo ở Phần B).
3. Nhấp **Next** -> Nhấp **Create target group** (không cần đăng ký instances thủ công ở bước này, ASG sẽ tự động đăng ký sau).

---

#### 3. Tạo Application Load Balancer (ALB)
*ALB nhận request từ bên ngoài (thông qua CloudFront) và phân phối đều cho các EC2 backend.*

1. Vào **EC2 Console** -> **Load Balancers** -> Chọn **Create load balancer** -> Chọn **Application Load Balancer** -> **Create**.
2. Cấu hình thông số:
   - **Load balancer name**: `caulongvui-alb`
   - **Scheme**: Chọn **Internet-facing** (ALB nhận traffic từ internet chuyển tiếp tới).
   - **IP address type**: `IPv4`
   - **Network mapping**: Chọn VPC `caulongvui-vpc`, và tích chọn tất cả Availability Zones hiển thị (phải chọn tối thiểu 2 AZs).
   - **Security groups**: Chọn `caulongvui-alb-sg` (đã tạo ở Bước 5.2).
   - **Listeners and routing**:
     - **Protocol & Port**: Chọn **HTTP** trên Port `80`.
     - **Default action**: Chuyển tiếp tới Target Group `caulongvui-backend-tg` vừa tạo.
3. Bấm **Create load balancer**.
   > [!NOTE]
   > Trong luồng sản xuất (production), CloudFront là endpoint public chính thức phục vụ người dùng. ALB sẽ nhận request được định tuyến từ CloudFront thông qua Behaviors `/api/*` và chuyển tiếp đến các EC2 backend trong Auto Scaling Group.

---

#### 4. Tạo Auto Scaling Group (ASG)
*ASG quản lý vòng đời và số lượng các máy chủ EC2 backend dựa trên tải CPU thực tế.*

1. Vào **EC2 Console** -> **Auto Scaling Groups** -> Chọn **Create Auto Scaling group**.
2. Các bước cấu hình:
   - **Bước 1: Choose Launch Template**: Đặt tên ASG (ví dụ: `caulongvui-asg`) và chọn Launch Template `caulongvui-template` đã tạo ở mục 1. Nhấn **Next**.
   - **Bước 2: Configure settings**: Chọn VPC `caulongvui-vpc` và tích chọn các Private Subnets trong VPC. Nhấn **Next**.
   - **Bước 3: Configure advanced options**:
     - Tại mục **Load balancing**: Chọn **Attach to an existing load balancer** -> Chọn Target Group `caulongvui-backend-tg`.
     - Tại mục **Health checks**: Chọn **ELB** (để ASG tự động thay thế instance nếu Load Balancer báo Unhealthy). Nhấn **Next**.
   - **Bước 4: Configure group size and scaling policies**:
     - **Desired capacity (Số lượng mong muốn)**: `2`
     - **Minimum capacity (Số lượng tối thiểu)**: `1`
     - **Maximum capacity (Số lượng tối đa)**: `4`
     - **Scaling policies**: Chọn **Target tracking scaling policy**.
       - **Metric type**: Chọn `Average CPU utilization`.
       - **Target value**: Điền `50` hoặc `70` (khi lượng sử dụng CPU trung bình vượt ngưỡng này, ASG sẽ tự động khởi chạy thêm EC2).
   - **Bước 5, 6, 7**: Nhấn **Next** qua các bước cấu hình thông báo/thẻ mặc định và nhấn **Create Auto Scaling group**.
