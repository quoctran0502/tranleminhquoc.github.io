---
title : "AWS Services Cost Estimation"
date : 2024-01-01 
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---
### AWS SERVICES COST ESTIMATION TABLE

#### 1. Network & Content Delivery Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **Amazon CloudFront** | - 1 TB free data transfer out to internet per month.<br>- 10,000,000 HTTP/HTTPS requests free per month. | Excess transfer: **$0.085/GB** |
| **Amazon VPC** | - Virtual Private Cloud hosting the entire infrastructure (encompassing 2 Availability Zones). | **Free** |
| **Internet Gateway (IGW)** | - Gateway allowing the VPC to connect to the public Internet. | **Free** |
| **Application Load Balancer (ALB)** | - Distributes incoming traffic to backend EC2 instances. | ~**$0.0225/hour** (approx. $16.20/month) |
| **NAT Gateway** | - 1 NAT Gateway deployed in a Public Subnet for backend EC2 instances in private networks to query external APIs. | ~**$0.045/hour** (approx. $32.40/month) + **$0.045/GB** data processed |

---

#### 2. Security & Identity Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **AWS WAF (Web Application Firewall)**| - Firewalls traffic and protects against DDoS attacks. | **$10.60/month** (base rate) |
| **AWS Certificate Manager (ACM)** | - Provisions and manages SSL/TLS certificates for HTTPS connections. | **Free** (when integrated with CloudFront) |

---

#### 3. Compute Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **Amazon EC2 (Web Server)** | - Spring Boot backend servers hosted in the Private Subnet (using `t3.micro` instances running 24/7). | **$20.81/month** (per instance) |
| **Auto Scaling Group (ASG)** | - Automatically scales and manages the backend EC2 server instances. | **Free** (only charged for the created EC2 resources) |

---

#### 4. Storage & Database Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **Amazon S3** | - Hosts the static web client Frontend code assets (HTML, CSS, JS, images). | **$0.023/GB** (for the first 50 GB) |
| **Amazon RDS (SQL Server)** | - Relational database deployed in Multi-AZ configuration (1 Active Primary and 1 Active Standby instance). | **$163.81/month** |

---

#### 5. Application Integration Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **Amazon SES (Simple Email Service)** | - Sends automated email notifications to application users. | **$0.0001/email** ($0.10 per 1,000 emails) |

---

#### 6. Observability & Identity Group

| Service | Configuration Description / Resources | Estimated Unit Price (Monthly/Unit) |
| :--- | :--- | :--- |
| **Amazon Cognito** | - Manages user identification, directories, and authentication. | - **First 50,000 MAUs** (Monthly Active Users) are **Free**.<br>- Excess MAUs: **$0.0055/MAU** |

---

### TOTAL ESTIMATED MONTHLY COST

> [!IMPORTANT]
> **Total Estimated Minimum Cost**: Starting from **$245.28/month** (variable depending on actual network data transfer, ASG auto-scaling events, and NAT Gateway data processing).
