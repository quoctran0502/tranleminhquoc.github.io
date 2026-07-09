---
title : "AWS Cognito Configuration for Google Login"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

In this section, we will integrate **AWS Cognito** and **Google Identity Provider** to allow users to log into the Spring Boot application using their Google accounts.

> [!IMPORTANT]
> All local URLs (`http://localhost:8080/`) in the configuration must be replaced with the URL of the web application deployed on AWS (e.g., `https://caulong.awsfcaj.com/`).

### Visual Reference

![Create Cognito user pool](/images/5-Workshop/5.1-AWS-Cognito/image1.png)

![Choose application type](/images/5-Workshop/5.1-AWS-Cognito/image2.png)

![Basic user pool settings](/images/5-Workshop/5.1-AWS-Cognito/image3.png)

![Registration settings](/images/5-Workshop/5.1-AWS-Cognito/image4.png)

---

### Step 1: Create a User Pool in AWS Cognito

1. Log in to the **AWS Console**.
2. Search for and select **Amazon Cognito** -> **User pools** -> click **Create user pool**.
3. Configure the basic details:
   - **Application type**: Select `Traditional web application`.
   - **Name**: `CauLong`
   - **Sign-in identifier**: Select `Email`.
4. Configure self-registration properties:
   - **Self-registration**: `Enable`
   - **Required attributes**: Select `email` and `name`.
   - **Return URL**: Set the callback URL of the application (to be updated to the AWS deployment URL later).
     - Local URL: `http://localhost:8080/login/oauth2/code/cognito`
     - AWS URL: `https://caulong.awsfcaj.com/login/oauth2/code/cognito`

---

### Step 2: Create a Cognito Domain

1. Go to **Amazon Cognito** -> **User pools** -> Select the User Pool created in Step 1.
2. Navigate to **Branding** -> **Domain**.
3. Register/Create a domain for Cognito.
   - *Example assigned domain*: `https://ap-southeast-182kj6alis.auth.ap-southeast-1.amazoncognito.com`

---

### Step 3: Create Google OAuth Client ID

1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Select **APIs & Services** -> **Credentials**.
3. Click **Create Credentials** -> **OAuth client ID**.
4. Configure the following parameters:
   - **Application type**: Select `Web application`.
   - **Name**: `CauLong`
   - **Allowed redirect URIs**: Enter the Cognito domain created in Step 2 appended with `/oauth2/idpresponse`:
     `https://ap-southeast-182kj6alis.auth.ap-southeast-1.amazoncognito.com/oauth2/idpresponse`
     > [!WARNING]
     > Remember to append `/oauth2/idpresponse` at the end of the Redirect URI path.

5. After successful creation, Google will provide:
   - **Google Client ID**
   - **Google Client Secret**
   *(Save these credentials to input into Cognito in the next step)*

---

### Step 4: Add Google Identity Provider to Cognito

1. Go back to the **Cognito Console** -> **User pools** -> Select your User Pool.
2. Select the **Authentication** tab -> **Social and external providers** section.
3. Click **Add identity provider** -> Select **Google**.
4. Enter the credentials obtained from the Google Cloud Console in Step 3:
   - **Client ID**: Enter `Google Client ID`.
   - **Client secret**: Enter `Google Client Secret`.
   - **Authorized scopes**: Enter `openid email profile`.
   - **Map attributes**:
     - `email` -> `email`
     - `name` -> `name`
     - *(Optional: map `picture` if needed)*.
5. Click **Add identity provider** to complete the linkage.

---

### Step 5: Configure App Client in Cognito

1. In the User Pool details -> Select the **Applications** tab -> **App clients** -> Select the **CauLong** app client.
2. Go to the **Login pages** section -> **Managed login pages configuration** -> Click **Edit**.
3. Configure the settings as follows:
   - **Allowed callback URLs**:
     - `https://caulong.awsfcaj.com/login/oauth2/code/cognito`
   - **Default redirect URL**:
     - `https://caulong.awsfcaj.com/login/oauth2/code/cognito`
     *(Append `/login/oauth2/code/cognito` to your web app's URL)*
   - **Allowed sign-out URLs**:
     - Add: `https://caulong.awsfcaj.com/`
     - Add: `https://caulong.awsfcaj.com/auth/google-switch.html`
   - **Identity providers**:
     - Select `Cognito user pool` and `Google`.
   - **OAuth 2.0 grant types**:
     - Select `Authorization code grant`.
   - **OpenID Connect scopes**:
     - Select `OpenID`, `Email`, and `Profile`.
4. Click **Save changes** to apply.

---

### Step 6: Configure Spring Boot Application

1. Retrieve the security parameters from Cognito:
   - In User Pool -> **Applications** -> **App clients** -> Select the **CauLong** app client to retrieve:
     - **App client ID**
     - **App client secret**
   - In the User Pool **Overview** page, retrieve:
     - **User pool ID** *(Format: `ap-southeast-1_xxxxxxxx`)*

2. Update the `application.properties` configuration file of your Spring Boot application as follows:

```properties
spring.security.oauth2.client.registration.cognito.client-id=xxxxxxxxxxxxxx
spring.security.oauth2.client.registration.cognito.client-secret=xxxxxxxxxxxx
spring.security.oauth2.client.registration.cognito.scope=openid,email,profile
spring.security.oauth2.client.registration.cognito.client-name=Cognito Google
spring.security.oauth2.client.registration.cognito.provider=cognito
spring.security.oauth2.client.provider.cognito.issuer-uri=https://cognito-idp.ap-southeast-1.amazonaws.com/ap-southeast-1_82kj6ALIs
```

> [!NOTE]
> Replace the values of `client-id`, `client-secret` and the User Pool ID in `issuer-uri` with your actual AWS Cognito settings.
