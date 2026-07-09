---
title : "Cấu hình VPC và RDS SQL Server"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

Trong phần này, chúng ta sẽ thực hiện cấu hình mạng cơ bản trên AWS bằng cách tạo một **VPC riêng** (Virtual Private Cloud), thiết lập các **Security Groups** phù hợp, sau đó khởi tạo cơ sở dữ liệu **Amazon RDS SQL Server** nằm trong VPC này.

### Hình minh họa

<div align="center">

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image1.png" alt="Tạo VPC" style="max-width:100%;height:auto;" />
  <figcaption><em>Tạo VPC riêng cho dự án</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image5.png" alt="Security group ALB" style="max-width:100%;height:auto;" />
  <figcaption><em>Security group cho Application Load Balancer</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image9.png" alt="Cấu hình RDS" style="max-width:100%;height:auto;" />
  <figcaption><em>Các bước tạo cơ sở dữ liệu RDS</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image15_new.png" alt="Cấu hình kết nối" style="max-width:100%;height:auto;" />
  <figcaption><em>Cấu hình kết nối và bảo mật</em></figcaption>
</figure>

</div>

---

### A. CẤU HÌNH VPC VÀ SECURITY GROUPS

#### 1. Tạo VPC riêng biệt
1. Truy cập vào **AWS Console**, tìm kiếm và chọn dịch vụ **VPC**.
2. Nhấp chọn **Create VPC** -> chọn tùy chọn **VPC and more**.
3. Cấu hình các thông số như sau:
   - **Name tag auto-generation**: Nhập `caulongvui`.
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - **Number of Availability Zones (AZs)**: `2`
   - **Number of public subnets**: `2`
   - **Number of private subnets**: `4`
   - **NAT gateways**: Chọn `None` (để tiết kiệm chi phí thử nghiệm).
   - **VPC endpoints**: Chọn `None`.
4. Nhấn nút **Create VPC** ở dưới cùng để hệ thống tự động tạo VPC, Subnets, Route Tables và Internet Gateway.

---

#### 2. Bật tính năng gán Public IP tự động cho Public Subnets
*Vì mặc định các public subnet không tự động cấp địa chỉ IP công cộng cho các máy chủ khi khởi chạy, chúng ta cần cấu hình như sau:*

1. Truy cập menu **VPC** -> **Subnets**.
2. Chọn public subnet đầu tiên: `caulongvui-subnet-public1-ap-southeast-1a`.
3. Nhấp chọn **Actions** -> **Edit subnet settings**.
4. Tick chọn ô **Enable auto-assign public IPv4 address**.
5. Nhấn **Save**.
6. Thực hiện thao tác tương tự cho public subnet còn lại (`caulongvui-subnet-public2-ap-southeast-1b`).

---

#### 3. Tạo các nhóm bảo mật (Security Groups)
*Chúng ta cần tạo 3 Security Groups độc lập nhằm phân quyền truy cập giữa Load Balancer, EC2 Backend, và RDS Database.*
*Nhóm Security Group cho ALB được tạo sẵn ở đây để dùng lại ở các bước triển khai phía sau trong mục 5.3 và 5.4.*

##### 3.1. Security Group cho Application Load Balancer (`caulongvui-alb-sg`)
1. Truy cập **VPC** -> **Security groups** -> Chọn **Create security group**.
2. Cấu hình:
   - **Security group name**: `caulongvui-alb-sg`
   - **Description**: `Security group for caulongvui ALB`
   - **VPC**: Chọn `caulongvui-vpc`.
3. Cấu hình **Inbound rules** (Quy tắc đầu vào):
   - Cho phép **HTTP** (Port `80`) từ nguồn `0.0.0.0/0` (Mọi nơi).
   - Cho phép **HTTPS** (Port `443`) từ nguồn `0.0.0.0/0` (Mọi nơi).
4. Nhấn **Create security group**.

##### 3.2. Security Group cho EC2 Backend (`caulongvui-backend-sg`)
1. Nhấp chọn **Create security group**.
2. Cấu hình:
   - **Security group name**: `caulongvui-backend-sg`
   - **Description**: `Security group for caulongvui backend EC2`
   - **VPC**: Chọn `caulongvui-vpc`.
3. Cấu hình **Inbound rules**:
   - Cho phép **SSH** (Port `22`) từ nguồn `My IP` (Chỉ cho phép IP máy tính cá nhân của bạn để quản trị).
   - Cho phép **Custom TCP** (Port `8080`) từ nguồn Security Group `caulongvui-alb-sg` (Chỉ nhận lưu lượng đi qua ALB).
4. Nhấn **Create security group**.

##### 3.3. Security Group cho Database (`caulongvui-rds-sg`)
1. Nhấp chọn **Create security group**.
2. Cấu hình:
   - **Security group name**: `caulongvui-rds-sg`
   - **Description**: `Security group for caulongvui RDS SQL Server`
   - **VPC**: Chọn `caulongvui-vpc`.
3. Cấu hình **Inbound rules**:
   - Cho phép **MS SQL** (Port `1433`) từ nguồn Security Group `caulongvui-backend-sg` (Chỉ chấp nhận kết nối từ Backend EC2).
4. Nhấn **Create security group**.

---

### B. TẠO CƠ SỞ DỮ LIỆU RDS SQL SERVER

#### Bước 1: Truy cập dịch vụ RDS
1. Đăng nhập vào **AWS Console**.
2. Tìm kiếm từ khóa `RDS` ở thanh tìm kiếm phía trên và chọn dịch vụ **RDS**.

#### Bước 2: Bắt đầu tạo Database
1. Trong menu bên trái, chọn **Databases**.
2. Nhấn nút **Create database**.

#### Bước 3: Cấu hình thông số Engine
1. **Choose a database creation method**: Chọn **Full configuration** (Tùy chọn này cho phép tùy chỉnh cài đặt mạng nâng cao, bật Public Access để kết nối từ máy cá nhân).
2. **Engine options**: Chọn **Microsoft SQL Server**.
3. **Database management type**: Chọn **Amazon RDS**.
4. **Edition**: Chọn **SQL Server Express Edition** (Cơ sở dữ liệu gọn nhẹ và nằm trong gói AWS Free Tier).
5. **Engine version**: Chọn **SQL Server 2019** (hoặc phiên bản mới hơn tùy nhu cầu).
6. **Templates**: Chọn **Free tier** nếu tài khoản của bạn còn hiển thị tùy chọn này; nếu console đang dùng giao diện trả phí thì thường sẽ thấy **Sandbox**.

#### Bước 4: Cấu hình thông tin đăng nhập (Credentials Settings)
1. **DB instance identifier**: Đặt tên định danh cho database instance, ví dụ: `caulongvui-db`.
2. **Master username**: Nhập tài khoản quản trị tối cao, ví dụ: `admin`.
3. **Master password**: Nhập mật khẩu bảo mật của bạn.

#### Bước 5: Cấu hình tài nguyên máy chủ và bộ nhớ
1. **DB instance class**: Chọn `db.t3.micro` nếu bạn muốn bám theo Free Tier; nếu chọn lớp lớn hơn như `db.t3.small` thì có thể phát sinh thêm chi phí.

#### Bước 6: Cấu hình kết nối mạng (Connectivity)
1. **Compute resource**: Chọn **Don’t connect to an EC2 compute resource**.
2. **Virtual private cloud (VPC)**: Chọn VPC của bạn vừa tạo: `caulongvui-vpc`.
3. **Public access**: Chọn **Yes** nếu bạn cần kết nối trực tiếp từ máy cá nhân qua SSMS. Nếu chỉ kết nối thông qua EC2/SSH tunnel thì nên chọn **No** để an toàn hơn.
4. **VPC security group**: Chọn **Choose existing** và chọn `caulongvui-rds-sg` (Đồng thời gỡ bỏ Security Group mặc định `default` nếu có).
5. Cuộn xuống dưới cùng và nhấn **Create database**. Quá trình tạo Database sẽ mất khoảng 5-10 phút.

---

### C. CẤU HÌNH THÊM RULES & KẾT NỐI QUA SSMS

#### 1. Cấp thêm quyền cho lập trình viên quản trị database (Optional)
*Nếu lập trình viên cần kết nối trực tiếp từ máy cá nhân tới RDS qua SSMS, hãy cập nhật Inbound rules của `caulongvui-rds-sg`:*
1. Vào **VPC** -> **Security groups** -> Chọn `caulongvui-rds-sg`.
2. Chọn **Edit inbound rules**.
3. Thêm một rule mới:
   - **Type**: `MS SQL` (Port `1433`)
   - **Source**: Chọn `My IP` (Hoặc `Custom` nhập địa chỉ IP cụ thể của lập trình viên).
4. Nhấn **Save rules**.

#### 2. Kết nối vào SQL Server Management Studio (SSMS)
1. Sau khi database có trạng thái `Available`, chọn database `caulongvui-db` và copy địa chỉ **Endpoint** trong phần **Connectivity & security**.
   - *Định dạng Endpoint ví dụ*: `caulongvui-db.xxxxxxxxx.ap-southeast-1.rds.amazonaws.com`
2. Mở ứng dụng **SQL Server Management Studio (SSMS)** trên máy tính cá nhân.
3. Điền thông tin kết nối:
   - **Server name**: Paste địa chỉ *Endpoint* vừa copy ở trên.
   - **Authentication**: Chọn `SQL Server Authentication`.
   - **Login**: `admin`
   - **Password**: Điền mật khẩu bạn đã thiết lập ở Bước 4.
4. Nhấn **Connect** để bắt đầu quản trị cơ sở dữ liệu.

#### 3. Kết nối thông qua SSH Tunnel và SSMS 
Nếu bạn muốn kết nối RDS thông qua EC2 backend bằng SSH tunnel như trong hình minh họa, làm theo các bước sau:

1. Mở **Command Prompt** trên máy tính cá nhân và chạy lệnh tạo tunnel:

```bash
ssh -i "C:\Users\Admin\OneDrive\Documents\caulongvui-key.pem" -N -L 14333:caulongvui-db.c3km8wawc7gx.ap-southeast-1.rds.amazonaws.com:1433 ec2-user@54.255.172.222
```

2. Giữ nguyên cửa sổ terminal này đang mở, vì nó là kết nối tunnel từ máy cá nhân đến RDS.

3. Mở **SQL Server Management Studio (SSMS)** và nhập các thông tin sau:
   - **Server name**: `127.0.0.1,14333`
   - **Authentication**: `SQL Server Authentication`
   - **Login**: `admin`
   - **Password**: Mật khẩu đã tạo ở Bước 4

4. Nhấn **Connect** để đăng nhập vào SSMS.

<div align="center">

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image20.png" alt="Mở SSH tunnel" style="max-width:100%;height:auto;" />
  <figcaption><em>Mở SSH tunnel từ Command Prompt</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image21.png" alt="Kết nối SSMS" style="max-width:100%;height:auto;" />
  <figcaption><em>Kết nối SSMS qua endpoint local</em></figcaption>
</figure>

</div>
