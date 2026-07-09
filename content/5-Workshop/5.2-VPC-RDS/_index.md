---
title : "VPC and RDS SQL Server Configuration"
date : 2024-01-01
weight : 2
chapter : false
pre : " <b> 5.2. </b> "
---

In this section, we will configure the basic networking architecture on AWS by creating a **custom VPC** (Virtual Private Cloud), establishing appropriate **Security Groups**, and launching an **Amazon RDS SQL Server** database instance within this network.

### Visual Reference

<div align="center">

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image1.png" alt="Create VPC" style="max-width:100%;height:auto;" />
  <figcaption><em>Create the custom VPC</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image5.png" alt="Application Load Balancer security group" style="max-width:100%;height:auto;" />
  <figcaption><em>Security group for the application load balancer</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image9.png" alt="RDS database configuration" style="max-width:100%;height:auto;" />
  <figcaption><em>RDS database creation settings</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image15_new.png" alt="Connectivity settings" style="max-width:100%;height:auto;" />
  <figcaption><em>Connectivity and security settings</em></figcaption>
</figure>

</div>

---

### A. VPC AND SECURITY GROUPS CONFIGURATION

#### 1. Create a Custom VPC
1. Log in to the **AWS Console**, search for, and select the **VPC** service.
2. Click **Create VPC** and select the **VPC and more** option.
3. Configure the following parameters:
   - **Name tag auto-generation**: Enter `caulongvui`.
   - **IPv4 CIDR block**: `10.0.0.0/16`
   - **Number of Availability Zones (AZs)**: `2`
   - **Number of public subnets**: `2`
   - **Number of private subnets**: `4`
   - **NAT gateways**: Select `None` (to minimize test environment costs).
   - **VPC endpoints**: Select `None`.
4. Click **Create VPC** at the bottom to automatically provision the VPC, Subnets, Route Tables, and Internet Gateway.

---

#### 2. Enable Auto-Assign Public IP for Public Subnets
*By default, public subnets do not automatically assign public IP addresses to launched instances. We need to enable this feature:*

1. In the VPC Console, navigate to **Subnets**.
2. Select the first public subnet: `caulongvui-subnet-public1-ap-southeast-1a`.
3. Click **Actions** -> **Edit subnet settings**.
4. Check the box **Enable auto-assign public IPv4 address**.
5. Click **Save**.
6. Repeat the exact steps for the remaining public subnet (`caulongvui-subnet-public2-ap-southeast-1b`).

---

#### 3. Create Security Groups
*We will create 3 independent Security Groups to isolate access traffic between the Load Balancer, EC2 Backend, and the RDS Database.*
*The ALB security group is prepared here so it can be reused in the later deployment steps in sections 5.3 and 5.4.*

##### 3.1. Security Group for Application Load Balancer (`caulongvui-alb-sg`)
1. Navigate to **VPC** -> **Security groups** -> click **Create security group**.
2. Configure:
   - **Security group name**: `caulongvui-alb-sg`
   - **Description**: `Security group for caulongvui ALB`
   - **VPC**: Select `caulongvui-vpc`.
3. Configure **Inbound rules**:
   - Allow **HTTP** (Port `80`) from source `0.0.0.0/0` (Anywhere).
   - Allow **HTTPS** (Port `443`) from source `0.0.0.0/0` (Anywhere).
4. Click **Create security group**.

##### 3.2. Security Group for EC2 Backend (`caulongvui-backend-sg`)
1. Click **Create security group**.
2. Configure:
   - **Security group name**: `caulongvui-backend-sg`
   - **Description**: `Security group for caulongvui backend EC2`
   - **VPC**: Select `caulongvui-vpc`.
3. Configure **Inbound rules**:
   - Allow **SSH** (Port `22`) from source `My IP` (Restricts administration access to your own PC).
   - Allow **Custom TCP** (Port `8080`) from source Security Group `caulongvui-alb-sg` (Only accepts traffic routed via the ALB).
4. Click **Create security group**.

##### 3.3. Security Group for Database (`caulongvui-rds-sg`)
1. Click **Create security group**.
2. Configure:
   - **Security group name**: `caulongvui-rds-sg`
   - **Description**: `Security group for caulongvui RDS SQL Server`
   - **VPC**: Select `caulongvui-vpc`.
3. Configure **Inbound rules**:
   - Allow **MS SQL** (Port `1433`) from source Security Group `caulongvui-backend-sg` (Only accepts database connections originating from the Backend EC2 instances).
4. Click **Create security group**.

---

### B. CREATE RDS SQL SERVER DATABASE

#### Step 1: Navigate to RDS Service
1. Log in to the **AWS Console**.
2. Search for `RDS` in the search bar and select the **RDS** service.

#### Step 2: Initiate Database Creation
1. From the left navigation menu, select **Databases**.
2. Click the **Create database** button.

#### Step 3: Configure Database Engine Settings
1. **Choose a database creation method**: Select **Full configuration** (This allows advanced networking customizations, enabling Public Access for testing and SSMS).
2. **Engine options**: Select **Microsoft SQL Server**.
3. **Database management type**: Select **Amazon RDS**.
4. **Edition**: Select **SQL Server Express Edition** (Lightweight database covered under the AWS Free Tier).
5. **Engine version**: Select **SQL Server 2019** (or a newer release as required).
6. **Templates**: Select **Free tier** if your console still shows that option; if you are on a paid-plan console, the equivalent option is usually **Sandbox**.

#### Step 4: Configure Credentials
1. **DB instance identifier**: Name your database instance, e.g., `caulongvui-db`.
2. **Master username**: Create the master admin user, e.g., `admin`.
3. **Master password**: Enter your secure password.

#### Step 5: Configure Instance & Storage
1. **DB instance class**: Select `db.t3.micro` if you want to stay aligned with Free Tier; choosing a larger class such as `db.t3.small` may incur additional charges.

#### Step 6: Configure Connectivity Settings
1. **Compute resource**: Select **Don’t connect to an EC2 compute resource**.
2. **Virtual private cloud (VPC)**: Choose your newly created custom VPC: `caulongvui-vpc`.
3. **Public access**: Select **Yes** if you need to connect directly from your local machine using SSMS. If you only plan to connect through EC2/SSH tunneling, choose **No** for better security.
4. **VPC security group**: Choose **Choose existing** and select `caulongvui-rds-sg` (Make sure to remove the `default` security group from the list).
5. Scroll down to the bottom and click **Create database**. The provisioning process takes approximately 5-10 minutes.

---

### C. ADDITIONAL RULES & CONNECTION VIA SSMS

#### 1. Grant Database Management Access to Developers (Optional)
*If developers need to connect directly to the RDS instance using SSMS from their personal machines, update the Inbound rules of `caulongvui-rds-sg`:*
1. Go to **VPC** -> **Security groups** -> select `caulongvui-rds-sg`.
2. Click **Edit inbound rules**.
3. Add a new rule:
   - **Type**: `MS SQL` (Port `1433`)
   - **Source**: Select `My IP` (or `Custom` to input a specific developer IP address).
4. Click **Save rules**.

#### 2. Connect to Database using SQL Server Management Studio (SSMS)
1. Once the database status turns to `Available`, select `caulongvui-db` and copy the **Endpoint** under **Connectivity & security**.
   - *Example Endpoint*: `caulongvui-db.xxxxxxxxx.ap-southeast-1.rds.amazonaws.com`
2. Open **SQL Server Management Studio (SSMS)** on your computer.
3. Fill in the connection properties:
   - **Server name**: Paste the copied *Endpoint*.
   - **Authentication**: Select `SQL Server Authentication`.
   - **Login**: `admin`
   - **Password**: Enter the password configured in Step 4.
4. Click **Connect** to begin managing the database.

#### 3. Connect through SSH Tunnel and SSMS
If you want to connect to RDS through the EC2 backend using an SSH tunnel, follow these steps:

1. Open **Command Prompt** on your computer and run the tunnel command:

```bash
ssh -i "C:\Users\Admin\OneDrive\Documents\caulongvui-key.pem" -N -L 14333:caulongvui-db.c3km8wawc7gx.ap-southeast-1.rds.amazonaws.com:1433 ec2-user@54.255.172.222
```

2. Keep that terminal window open because it maintains the tunnel between your local machine and RDS.

3. Open **SQL Server Management Studio (SSMS)** and use these values:
   - **Server name**: `127.0.0.1,14333`
   - **Authentication**: `SQL Server Authentication`
   - **Login**: `admin`
   - **Password**: the password created in Step 4

4. Click **Connect** to sign in to SSMS.

<div align="center">

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image20.png" alt="Open SSH tunnel" style="max-width:100%;height:auto;" />
  <figcaption><em>Open the SSH tunnel from Command Prompt</em></figcaption>
</figure>

<figure>
  <img src="/images/5-Workshop/5.2-VPC-RDS/image21.png" alt="Connect in SSMS" style="max-width:100%;height:auto;" />
  <figcaption><em>Connect to the local tunnel endpoint in SSMS</em></figcaption>
</figure>

</div>
