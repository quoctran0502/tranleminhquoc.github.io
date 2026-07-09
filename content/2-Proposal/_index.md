---
title: "Proposal"
date: 2026-07-09
weight: 2
chapter: false
pre: " <b> 2. </b> "
---

# CauLongVui: Online Badminton Court Rental & Management System on AWS

## 1. Project Goal

**CauLongVui** is an end-to-end platform for badminton court booking and sports center operations. The project is designed to solve both the business workflow problem and the cloud infrastructure problem at the same time.

### Business Goal
- Digitalize the full operation flow of a badminton center.
- Support real-time court lookup and hold-slot booking to reduce double-booking.
- Enable online payment through multiple channels, including an internal wallet and external gateways such as MoMo and VNPay.
- Support membership management, equipment rental, and food and beverage pre-ordering.

### Technical Goal
- Build a cloud architecture that follows **High Availability**, **Defense in Depth**, and **Auto Scalability** principles.
- Deploy the system on **Amazon Web Services (AWS)** to improve stability, security, and operational flexibility.

## 2. Problems and Benefits

### Problems Addressed
- Manual booking workflows can easily create scheduling conflicts and double-booking.
- Revenue data is split across court fees, drinks, rentals, and membership payments.
- Peak-hour traffic may overload the system and interrupt service.

### Expected Benefits
- Real-time booking makes the customer experience faster and more accurate.
- Centralized payment and order data help admins manage the business from one place.
- AWS auto scaling and managed services reduce operational risk and improve resilience.
- CloudWatch, X-Ray, and SNS provide clearer visibility for monitoring and alerting.

## 3. Solution Overview

The system uses a 3-tier AWS architecture:
- **Presentation layer**: Static frontend hosted on Amazon S3 and delivered globally by CloudFront.
- **Application layer**: Spring Boot backend running on Amazon EC2 inside private subnets, protected by ALB, Auto Scaling, and WAF.
- **Data layer**: Amazon RDS stores transactional data in Multi-AZ mode, while uploaded files are stored in Amazon S3.

### 3.1 Architecture Snapshots

![CauLongVui AWS architecture](/images/2-Proposal/caulongvui-architecture.jpg)

### 3.2 Core AWS Services

- **Amazon S3** stores the static frontend and uploaded media files.
- **Amazon CloudFront** accelerates global content delivery.
- **AWS WAF** protects the application from common web attacks.
- **Amazon EC2** runs the Spring Boot backend in private subnets.
- **Application Load Balancer (ALB)** distributes incoming requests across healthy instances.
- **Auto Scaling Group (ASG)** automatically adjusts backend capacity based on traffic.
- **Amazon RDS Multi-AZ** provides a managed and highly available SQL Server database.
- **NAT Gateway** lets backend instances call external services securely.
- **Amazon SES** sends booking confirmations and invoices.
