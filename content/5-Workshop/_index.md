---
title: "Workshop"
date: 2026-07-09
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Deploying and Configuring a Fullstack Web Application on AWS

#### Overview

In this series of hands-on labs, we will build and deploy a **Fullstack** web application (Static Frontend + Spring Boot Backend + SQL Server Database) on **Amazon Web Services (AWS)**.

* **Demo Website (CauLongVui)**: [https://drive.google.com/drive/u/0/folders/1XI1mu_dUN5vNOfi7s5rXoBk4oP7qKwcz](https://drive.google.com/drive/u/0/folders/1XI1mu_dUN5vNOfi7s5rXoBk4oP7qKwcz)

The system is architected to align with cloud design best practices for performance, scalability, security, and high availability:
- **Identity & Access Management**: Integrate **AWS Cognito** as the identity broker, enabling Google OAuth federated login.
- **Network Design & Security**: Create a custom **VPC** with isolated public and private subnets, securing resource communications using **Security Groups** and **AWS WAF**.
- **Hosting & CDN Distribution**: Host the static web client on **Amazon S3** and cache resources globally using **Amazon CloudFront CDN**.
- **Elasticity & High Availability**: Deploy an **Application Load Balancer (ALB)** coupled with an **Auto Scaling Group (ASG)** of **EC2** backend servers to dynamically adjust instance count based on real-time CPU utilization.
- **Database Management**: Provision a fully managed **Amazon RDS SQL Server** database instance nested inside private subnets.
- **Transactional Notifications**: Trigger automated notification emails using **Amazon SES**.

#### Content

1. [AWS Cognito Configuration](5.1-AWS-Cognito/)
2. [VPC and RDS](5.2-VPC-RDS/)
3. [Application Deployment (S3, CloudFront, EC2, WAF, SES)](5.3-S3-CloudFront-EC2-WAF-SES/)
4. [Backend Auto Scaling](5.4-Auto-Scaling/)
5. [AWS Services Cost Estimation](5.5-Cost-Estimation/)
