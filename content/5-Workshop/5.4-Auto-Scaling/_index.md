---
title : "Backend Auto Scaling Configuration"
date : 2024-01-01 
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

In this section, we will set up the automatic scaling system (**Auto Scaling**) for the Spring Boot Backend server. This configuration ensures High Availability, automatic recovery from hardware/application failures, and minimal service downtime.

### Visual Reference

![Create launch template](/images/5-Workshop/5.4-Auto-Scaling/image1.png)

![Create target group](/images/5-Workshop/5.4-Auto-Scaling/image6.png)

![Create Application Load Balancer](/images/5-Workshop/5.4-Auto-Scaling/image9.png)

![Create Auto Scaling Group](/images/5-Workshop/5.4-Auto-Scaling/image16.png)

---

### A. WHY USE AUTO SCALING?

- **Dynamic Load Adaptation**: When user traffic surges, a single EC2 virtual machine will not be able to handle the load. The Auto Scaling Group (ASG) automatically provisions additional EC2 instances to distribute the traffic.
- **Self-Healing (Auto Recovery)**: If an EC2 instance experiences hardware failure or the application freezes (becomes Unhealthy), the ASG automatically terminates the faulty instance and spins up a healthy replacement instance.
- **Cost Optimization**: Automatically scales down the number of servers during low traffic periods (e.g., nighttime).

#### Auto Scaling Architecture for the Project:
- **Backend**: Deployed on multiple **EC2** instances managed by an **Auto Scaling Group (ASG)** and distributed via an **Application Load Balancer (ALB)**.
- **Frontend**: Hosted statically on **Amazon S3** and distributed through the **Amazon CloudFront** CDN.
- **Database**: Uses **Amazon RDS** (RDS dynamically manages storage scaling; it does not belong in the EC2 Auto Scaling Group).

---

### B. SPRING BOOT HEALTH ENDPOINT CONFIGURATION

To allow the Application Load Balancer and Auto Scaling Group to check the operational health of the Spring Boot application (using Health Checks), we must enable Spring Boot Actuator.

1. Add the Actuator dependency to your project's `pom.xml`:
   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-actuator</artifactId>
   </dependency>
   ```

2. Add the following parameters to your configuration file `application.properties`:
   ```properties
   management.endpoints.web.exposure.include=health,info
   management.endpoint.health.probes.enabled=true
   ```
   > [!NOTE]
   > Once configured, the application exposes a health check API at the endpoint path `/actuator/health`. The ALB and Auto Scaling Group will query this endpoint to verify instance health.

---

### C. AWS AUTO SCALING SETUPS

#### 1. Create a Launch Template
*A Launch Template defines the virtual machine configuration used to provision new EC2 instances during scaling events.*

1. Go to the **EC2 Console** -> **Launch Templates** -> click **Create launch template**.
2. Configure basic settings:
   - **Launch template name**: `caulongvui-template`
   - **Auto Scaling Guidance**: Check the box *Provide guidance to help me set up a template that can be used with EC2 Auto Scaling*.
   - **Application and OS Images (AMI)**: Choose **Amazon Linux 2023** (Windows is not recommended for this setup due to slower startup times and higher costs).
   - **Instance type**: Select `t3.micro` or `t3.small` (based on typical workload expectations).
   - **Key pair**: Select `caulongvui-key`.
   - **Security groups**: Select `caulongvui-backend-sg`.
3. Scroll down to **Advanced details** -> **User data** section, and paste the following bash script to automate Java environment setup and Spring Boot execution on instance startup:
   ```bash
   #!/bin/bash
   set -e

   # Update system packages
   yum update -y
   # Install Java 21 and AWS CLI to download the JAR from S3
   yum install -y java-21-amazon-corretto awscli

   # Create deployment directories and download application JAR from S3
   mkdir -p /home/ec2-user
   cd /home/ec2-user
   aws s3 cp s3://caulongvui-static/deploy/CauLongVui-0.0.1-SNAPSHOT.jar /home/ec2-user/app.jar

   # Configure Spring Boot as a background systemd service
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

   # Reload systemd configuration, enable, and start service
   systemctl daemon-reload
   systemctl enable caulongvui
   systemctl start caulongvui
   ```
4. Click **Create launch template**.

---

#### 2. Create a Target Group
*Target Groups represent the destination instances where the Load Balancer routes incoming traffic.*

1. Go to the **EC2 Console** -> **Target Groups** -> click **Create target group**.
2. Configure settings:
   - **Choose a target type**: Select **Instances**.
   - **Target group name**: `caulongvui-backend-tg`
   - **Protocol & Port**: Select **HTTP** and Port `8080` (the port Spring Boot binds to).
   - **VPC**: Choose `caulongvui-vpc`.
   - **Health check settings**:
     - **Health check protocol**: `HTTP`
     - **Health check path**: `/actuator/health` (uses the Actuator endpoint created in Section B).
3. Click **Next** -> click **Create target group** (no need to register instances manually, the ASG will automatically register them later).

---

#### 3. Create an Application Load Balancer (ALB)
*The ALB accepts incoming public web requests (via CloudFront) and distributes them evenly across backend EC2 instances.*

1. Go to the **EC2 Console** -> **Load Balancers** -> click **Create load balancer** -> select **Application Load Balancer** -> **Create**.
2. Configure parameters:
   - **Load balancer name**: `caulongvui-alb`
   - **Scheme**: Select **Internet-facing** (ALB routes public internet traffic to targets).
   - **IP address type**: `IPv4`
   - **Network mapping**: Choose VPC `caulongvui-vpc` and check all listed Availability Zones (minimum of 2 AZs is required).
   - **Security groups**: Select `caulongvui-alb-sg` (created in Step 5.2).
   - **Listeners and routing**:
     - **Protocol & Port**: Select **HTTP** on Port `80`.
     - **Default action**: Forward to Target Group `caulongvui-backend-tg`.
3. Click **Create load balancer**.
   > [!NOTE]
   > In a production environment, CloudFront serves as the primary public entry point. The ALB accepts incoming traffic routed from CloudFront via path behaviors (`/api/*`) and forwards them to the EC2 instances in the Auto Scaling Group.

---

#### 4. Create an Auto Scaling Group (ASG)
*The ASG monitors CPU load and manages the lifecycle and count of backend EC2 instances.*

1. Go to the **EC2 Console** -> **Auto Scaling Groups** -> click **Create Auto Scaling group**.
2. Step-by-step configuration wizard:
   - **Step 1: Choose Launch Template**: Give the ASG a name (e.g., `caulongvui-asg`) and select the launch template `caulongvui-template` created in step 1. Click **Next**.
   - **Step 2: Configure settings**: Choose VPC `caulongvui-vpc` and select the Private Subnets. Click **Next**.
   - **Step 3: Configure advanced options**:
     - **Load balancing**: Choose **Attach to an existing load balancer** -> select Target Group `caulongvui-backend-tg`.
     - **Health checks**: Select **ELB** (enables the ASG to automatically replace unhealthy instances flagged by the Load Balancer). Click **Next**.
   - **Step 4: Configure group size and scaling policies**:
     - **Desired capacity**: `2`
     - **Minimum capacity**: `1`
     - **Maximum capacity**: `4`
     - **Scaling policies**: Select **Target tracking scaling policy**.
       - **Metric type**: Select `Average CPU utilization`.
       - **Target value**: Enter `50` or `70` (whenever the average CPU utilization exceeds this value, the ASG will spin up additional EC2 instances).
   - **Steps 5, 6, and 7**: Advance through the default tag and notification settings, then click **Create Auto Scaling group**.
