---
title : "Frontend & Backend Deployment (S3, CloudFront, EC2, WAF, SES)"
date : 2024-01-01 
weight : 3
chapter : false
pre : " <b> 5.3. </b> "
---

In this section, we will deploy the entire application (including the static frontend assets and the Spring Boot backend server) to AWS. The services integrated include: **S3** (Frontend hosting), **CloudFront** (CDN distribution), **EC2** (running the Backend), **WAF** (application protection), and **SES** (sending email notifications).

### Visual Reference

![Create S3 bucket](/images/5-Workshop/5.3-S3-CloudFront-EC2-WAF-SES/image1.png)

![Create CloudFront distribution](/images/5-Workshop/5.3-S3-CloudFront-EC2-WAF-SES/image8.png)

![Launch EC2 backend instance](/images/5-Workshop/5.3-S3-CloudFront-EC2-WAF-SES/image15.png)

![Verify SES sender identity](/images/5-Workshop/5.3-S3-CloudFront-EC2-WAF-SES/image25.png)

---

### A. AMAZON S3 CONFIGURATION (STATIC FRONTEND HOSTING)

1. Log in to the **AWS Console**, select **S3** -> click **Create bucket**.
2. Configure basic details:
   - **Bucket name**: `caulongvui-static`
   - **Region**: Select your desired region (e.g., `ap-southeast-1` - Singapore).
3. Security settings:
   - In the **Block Public Access settings for this bucket** section: **Uncheck all** (Allow public access) to permit public web access to static assets.
4. Click **Create bucket**.
5. Enable **Static website hosting**:
   - Go to your bucket `caulongvui-static` -> select the **Properties** tab.
   - Scroll down to the bottom to find **Static website hosting** -> click **Edit**.
   - Select **Enable**.
   - Input configuration:
     - **Index document**: `index.html`
     - **Error document**: `index.html`
   - Click **Save changes**.
6. Configure the **Bucket Policy** to grant public read permission to all objects:
   - Switch to the **Permissions** tab -> **Bucket policy** section -> click **Edit**.
   - Paste the following policy:
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
   - Click **Save changes**.

---

### B. CLOUDFRONT CONFIGURATION (CDN DISTRIBUTION)

1. Navigate to the **CloudFront** service -> click **Create distribution**.
2. Configure **Origin Settings**:
   - **Origin domain**: Choose the S3 Static Website Hosting endpoint created above (format: `caulongvui-static.s3-website-ap-southeast-1.amazonaws.com`).
     > [!TIP]
     > Use the S3 website endpoint rather than selecting the direct S3 bucket origin to ensure client-side routing functions correctly.
3. Keep Steps 4 & 5 at their default settings -> click **Create distribution**.
4. Wait for the CloudFront distribution status to change to `Deployed` (usually takes 3-5 minutes).
5. Copy the assigned **Distribution domain name** (e.g., `d33wcfjdygo2at.cloudfront.net`).
6. Update application configuration files:
   - **Frontend**: Open `config.js` in your source code and update the configuration variables:
     - `apiBase`: API path (e.g., `https://d33wcfjdygo2at.cloudfront.net/api`)
     - `cdnUrl`: `https://d33wcfjdygo2at.cloudfront.net`
   - **Backend**: Open `application.properties` and add:
     - `app.cdn-url=https://d33wcfjdygo2at.cloudfront.net`
     - `app.frontend-url=https://d33wcfjdygo2at.cloudfront.net`
7. **Upload Frontend Assets to S3**:
   - Open a terminal in the root directory of your project and run the following command to sync static assets to the S3 bucket:
     ```bash
     aws s3 sync src/main/resources/static/ s3://caulongvui-static/ --delete --region ap-southeast-1
     ```
8. **Update Redirect URL in AWS Cognito**:
   - Go to your Cognito User Pool -> App Client -> Modify the **Allowed callback URLs** to use the CloudFront domain:
     `https://d33wcfjdygo2at.cloudfront.net/login/oauth2/code/cognito`
   - Update **Allowed sign-out URLs** to point to the CloudFront domain as well.
9. **Configure CloudFront Behaviors**:
   - To route API calls and logins to the EC2 Backend server, configure additional **Behaviors**:
     - Click **Create behavior**.
     - API Behavior Settings:
       - **Path pattern**: `/api/*`
       - **Origin**: Select the origin pointing to your EC2 backend.
       - **Viewer protocol policy**: Select `Redirect HTTP to HTTPS`.
       - **Allowed HTTP methods**: Select all methods (`GET, HEAD, OPTIONS, PUT, POST, PATCH, DELETE`).
     - Create two more behaviors following the same steps for paths:
       - `/oauth2/*`
       - `/login/*`

---

### C. BACKEND DEPLOYMENT ON EC2

#### 1. Launch EC2 Instance
1. Go to the **EC2** dashboard -> click **Launch instance**.
2. Settings configuration:
   - **Name**: `caulongvui-backend`
   - **AMI**: Amazon Linux 2023
   - **Instance type**: `t3.micro` (or `t2.micro` depending on Free Tier availability).
   - **Key pair**: Create a new key pair named `caulongvui-key` and download the `.pem` file.
   - **Network settings**: Choose VPC `caulongvui-vpc` and associate the security group `caulongvui-backend-sg` created in Step 5.2.
3. Click **Launch instance**.

#### 2. Connect and Configure the EC2 Environment
1. Open PowerShell or Terminal on your local machine, modify file permissions on the key file (if running Linux/macOS), and connect via SSH:
   ```bash
   ssh -i "C:\path\to\caulongvui-key.pem" ec2-user@<Elastic_IP_cua_ban>
   ```
2. Update Amazon Linux system packages:
   ```bash
   sudo dnf update -y
   ```
3. Install Java 17:
   ```bash
   sudo dnf install java-17-amazon-corretto-headless -y
   ```
4. Build the Spring Boot application locally to generate a JAR file:
   ```bash
   # Run this command in the project root directory locally
   .\mvnw clean package -DskipTests
   ```
5. Upload the JAR file to the EC2 instance using SCP:
   ```bash
   scp -i "C:\path\to\caulongvui-key.pem" target/CauLongVui-0.0.1-SNAPSHOT.jar ec2-user@<Elastic_IP_cua_ban>:/home/ec2-user/CauLongVui-0.0.1-SNAPSHOT.jar
   ```

---

### D. AWS WAF INTEGRATION

To secure your web application without interrupting developer activities during testing:
- Associate **AWS WAF** with the created CloudFront Distribution.
- Enable WAF rules in **Monitoring/Count mode** to prevent false positives and block legitimate testing requests.

---

### E. AMAZON SES CONFIGURATION (EMAIL SERVICE)

*Once the Backend is successfully deployed to EC2, configure SES as follows:*

#### Step 1: Declare the SES Email Region
In your Spring Boot `application.properties` configuration file, declare the SES region (Singapore is recommended):
```properties
app.mail.ses.region=ap-southeast-1
```

#### Step 2: Verify Sender Email Address in SES
1. Open the AWS Console, search for **Amazon SES** -> select **Identities**.
2. Click the **Create identity** button.
3. Choose **Identity type** as `Email address`.
4. Enter the email address you wish to send from (e.g., `ngocchau0845@gmail.com`).
5. Click **Create identity**.
6. Check your Gmail inbox, find the confirmation email from AWS, and click the link to verify the address.
7. The status in the Identities tab will update to **Verified**.

#### Step 3: Verify Recipient Email Address (If SES is in Sandbox Mode)
> [!IMPORTANT]
> While your SES account is in Sandbox mode, you can only send emails to verified recipient addresses.
- Repeat the steps in Step 2 for the recipient email address (e.g., `test@gmail.com`) to allow end-to-end testing.

#### Step 4: Create SES Permission Policy for the IAM User
1. Go to the **IAM** dashboard -> **Users** -> select the user running the application.
2. Select **Add permissions** -> **Create inline policy**.
3. Choose the JSON tab and paste the following policy configuration:
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
4. Name the policy (e.g., `caulongvui-ses-policy`) and save it.

#### Step 5: Configure IAM Role for the EC2 Backend Instance
1. Go to **IAM** -> **Roles** -> search for the role associated with the EC2 backend instance (e.g., `AmazonS3FullAccess-caulongvui`).
2. Under the **Permissions** tab, select **Add permissions** -> **Attach policies**.
3. Search for the policies `AmazonSESFullAccess` and `AmazonS3FullAccess`. Check both and click **Add permissions**.
4. Go to **EC2** -> **Instances** -> select `caulongvui-backend`.
5. Select **Actions** -> **Security** -> **Modify IAM role**.
6. Select the role `AmazonS3FullAccess-caulongvui` and click **Update IAM role**.

---

### F. RUNNING THE APPLICATION ON EC2

1. Connect to the backend instance by selecting the instance `caulongvui-backend` -> Click **Connect** -> Select the **SSM Session Manager** tab to open the terminal directly in your web browser.
2. Terminate any pre-existing JAR processes if running:
   ```bash
   sudo pkill -f CauLongVui-0.0.1-SNAPSHOT.jar
   ```
3. Run the Spring Boot application as a background process using `nohup`, passing the SES environment variables:
   ```bash
   nohup java -jar /home/ec2-user/CauLongVui-0.0.1-SNAPSHOT.jar \
     --app.mail.enabled=true \
     --app.mail.from="ngocchau0845@gmail.com" \
     --app.mail.ses.region="ap-southeast-1" \
     --app.mail.booking-reminder-hours=12 \
     > /tmp/caulongvui.log 2>&1 &
   ```
