---
title: "Blog 1 - Understanding Binary Asset Storage Architecture with Lore on AWS"
date: 2026-07-09
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

## Introduction

During game development or digital media projects, managing binary assets such as images, audio files, videos, 3D models, and animations is always a significant challenge. These files are typically large in size and are updated frequently, creating the need for a storage system that is efficient, cost-effective, and highly scalable.

---

# Challenges of Traditional Binary Asset Storage

Traditional version control systems work very well for source code but have limitations when handling binary files. Every time an asset is modified, the system usually stores an entirely new version of the file, even if only a small portion has changed.

This leads to several issues:

- Rapid growth in storage usage.
- Higher operational costs.
- Longer data synchronization times.
- Difficulty scaling as the number of assets increases.

For large game projects where thousands of assets are updated every day, these challenges become even more significant.

---

# Lore's Approach

Lore addresses this problem by using a **Content-Addressed Storage (CAS)** model.

Instead of storing the entire file after every modification, Lore divides the data into multiple smaller fragments and assigns each fragment a unique hash value.

When an asset is updated, the system stores only the fragments that have actually changed while reusing the existing fragments that remain unchanged.

This approach provides several benefits:

- Reduces duplicate data.
- Saves storage space.
- Improves data synchronization speed.
- Supports more efficient version management.

This solution is especially suitable for development environments that manage large volumes of data with frequent updates.

---

# AWS Services Used

To deploy Lore on AWS, the reference architecture combines several AWS services, each responsible for a specific role.

## Amazon S3

Amazon S3 serves as the primary storage location for binary assets.

Its advantages include:

- High data durability.
- Virtually unlimited scalability.
- Cost-effective storage.
- Ideal for object storage workloads.

---

## Amazon EC2

Amazon EC2 is used to deploy **Edge Pods**, which are responsible for:

- Receiving user requests.
- Caching frequently accessed data.
- Improving data read and write performance.

---

## Amazon ECS

Amazon ECS runs the Lore backend.

Its primary responsibilities include:

- Processing newly uploaded data.
- Detecting duplicate data.
- Coordinating the process of writing data into the storage system.

---

## Amazon DynamoDB

Amazon DynamoDB stores repository metadata, including:

- Asset information.
- Data versions.
- References to fragments.
- Repository and branch information.

Thanks to its automatic scalability and low latency, DynamoDB is well suited for systems that require continuous metadata access.

---

## AWS Cloud Map

AWS Cloud Map provides service discovery within the distributed environment, allowing different system components to locate and communicate with one another dynamically.

---

# Benefits of the Architecture

This architecture offers several advantages:

- Significantly reduces storage costs through data deduplication.
- Improves team productivity by accelerating data synchronization.
- Provides excellent support for large-scale game and digital media projects.
- Takes advantage of AWS scalability and reliability.
- Reduces infrastructure management overhead by leveraging fully managed AWS services.

---

# Lessons Learned

From this article, I gained a better understanding of how choosing the right storage architecture can greatly impact both system performance and operational costs.

Lore is more than just a binary asset storage solution—it is an excellent example of how multiple AWS services can be integrated to solve real-world challenges in the game development industry.

This article also helped me better understand the roles of Amazon S3, Amazon EC2, Amazon ECS, Amazon DynamoDB, and AWS Cloud Map in building large-scale storage systems.

---

# Conclusion

Lore introduces a modern approach to binary asset management by storing only the portions of data that have actually changed instead of saving an entirely new copy of every file.

By integrating AWS services such as Amazon S3, Amazon EC2, Amazon ECS, Amazon DynamoDB, and AWS Cloud Map, the system achieves high scalability, optimized storage costs, and improved development efficiency.

This architecture is an excellent reference for organizations in the gaming industry, digital media, or any business that needs to manage large volumes of binary assets on the AWS Cloud.

---

## Reference Architecture

The following architecture illustrates how **Lore** manages binary asset storage on AWS. Binary assets are received through Edge Pods, routed using AWS Cloud Map, processed by Amazon ECS, and stored in Amazon S3 and Amazon DynamoDB. This architecture improves scalability, reduces duplicate data, and optimizes storage efficiency for large-scale game and digital media projects.

![AWS Reference Architecture for Binary Asset Storage with Lore](/images/3-Blog/aws-reference-architecture-for-binary-asset-storage-with-lore.jpg)

*Figure 1. AWS Reference Architecture for Binary Asset Storage with Lore.*

---

# Reference

https://aws.amazon.com/vi/blogs/gametech/how-lore-rethinks-binary-asset-storage-on-aws/