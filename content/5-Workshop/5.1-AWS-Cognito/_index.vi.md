---
title : "Cấu hình AWS Cognito đăng nhập với Google"
date : 2024-01-01 
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

Trong phần này, chúng ta sẽ thực hiện tích hợp **AWS Cognito** và **Google Identity Provider** để cho phép người dùng đăng nhập bằng tài khoản Google vào ứng dụng Spring Boot.

> [!IMPORTANT]
> Toàn bộ các đường dẫn `http://localhost:8080/` trong cấu hình cục bộ sẽ được thay thế bằng đường dẫn của ứng dụng web đã triển khai trên AWS (ví dụ: `https://caulong.awsfcaj.com/`).

---

### Bước 1: Tạo User Pool trên AWS Cognito

1. Đăng nhập vào **AWS Console**.
2. Tìm kiếm và chọn dịch vụ **Amazon Cognito** -> **User pools** -> chọn **Create user pool**.
3. Cấu hình các thông tin cơ bản:
   - **Application type**: Chọn `Traditional web application`.
   - **Name**: `CauLong`
   - **Sign-in identifier**: Chọn `Email`.
4. Cấu hình thuộc tính đăng ký:
   - **Self-registration**: `Enable`
   - **Required attributes**: Chọn `email` và `name`.
   - **Return URL**: Thiết lập đường dẫn callback của ứng dụng (sau này đổi thành link web đã đưa lên AWS).
     - Link chạy local: `http://localhost:8080/login/oauth2/code/cognito`
     - Link triển khai AWS: `https://caulong.awsfcaj.com/login/oauth2/code/cognito`

---

### Bước 2: Tạo Cognito Domain

1. Truy cập vào **Amazon Cognito** -> **User pools** -> Chọn User Pool vừa tạo ở Bước 1.
2. Điều hướng tới menu **Branding** -> **Domain**.
3. Đăng ký/Tạo domain cho Cognito.
   - *Ví dụ Domain được cấp*: `https://ap-southeast-182kj6alis.auth.ap-southeast-1.amazoncognito.com`

---

### Bước 3: Tạo Google OAuth Client ID

1. Truy cập vào [Google Cloud Console](https://console.cloud.google.com/).
2. Chọn **APIs & Services** -> **Credentials (Identifiants)**.
3. Chọn **Create Credentials** -> **OAuth client ID**.
4. Cấu hình các thông số sau:
   - **Application type**: Chọn `Web application`.
   - **Name**: `CauLong`
   - **Allowed redirect URIs**: Nhập Cognito domain đã tạo ở Bước 2 kèm hậu tố `/oauth2/idpresponse`:
     `https://ap-southeast-182kj6alis.auth.ap-southeast-1.amazoncognito.com/oauth2/idpresponse`
     > [!WARNING]
     > Nhớ là phải thêm phần `/oauth2/idpresponse` vào cuối đường dẫn Redirect URI.

5. Sau khi tạo thành công, Google sẽ cấp:
   - **Google Client ID**
   - **Google Client Secret**
   *(Lưu lại thông tin này để cấu hình vào Cognito ở bước tiếp theo)*

---

### Bước 4: Thêm Google Identity Provider vào Cognito

1. Quay lại giao diện **Cognito** -> **User pools** -> Chọn User Pool của bạn.
2. Chọn tab **Authentication** -> mục **Social and external providers**.
3. Chọn **Add identity provider** -> Chọn **Google**.
4. Nhập các thông tin đã lấy từ Google Cloud Console ở Bước 3:
   - **Client ID**: Nhập `Google Client ID`.
   - **Client secret**: Nhập `Google Client Secret`.
   - **Authorized scopes**: Điền `openid email profile`.
   - **Map attributes**:
     - `email` ➔ `email`
     - `name` ➔ `name`
     - *(Có thể map thêm trường `picture` nếu cần).*
5. Nhấp nút **Add identity provider** để hoàn tất liên kết.

---

### Bước 5: Cấu hình App Client trong Cognito

1. Tại giao diện User Pool -> Chọn tab **Applications** -> **App clients** -> Chọn app **CauLong**.
2. Tìm đến mục **Login pages** -> **Managed login pages configuration** -> Chọn **Edit**.
3. Cấu hình các mục như sau:
   - **Allowed callback URLs**:
     - `https://caulong.awsfcaj.com/login/oauth2/code/cognito`
   - **Default redirect URL**:
     - `https://caulong.awsfcaj.com/login/oauth2/code/cognito`
     *(Thêm `/login/oauth2/code/cognito` vào phía sau link web của bạn)*
   - **Allowed sign-out URLs**:
     - Thêm: `https://caulong.awsfcaj.com/`
     - Thêm: `https://caulong.awsfcaj.com/auth/google-switch.html`
   - **Identity providers**:
     - Chọn `Cognito user pool` và `Google`.
   - **OAuth 2.0 grant types**:
     - Chọn `Authorization code grant`.
   - **OpenID Connect scopes**:
     - Chọn `OpenID`, `Email`, và `Profile`.
4. Bấm **Save changes** để hoàn tất.

---

### Bước 6: Cấu hình ứng dụng Spring Boot

1. Lấy các thông số bảo mật từ Cognito:
   - Tại User Pool -> **Applications** -> **App clients** -> Chọn app **CauLong** để lấy:
     - **App client ID**
     - **App client secret**
   - Tại trang **Overview** của User Pool, lấy:
     - **User pool ID** *(Dạng: `ap-southeast-1_xxxxxxxx`)*

2. Cập nhật tệp cấu hình `application.properties` của ứng dụng Spring Boot như sau:

```properties
spring.security.oauth2.client.registration.cognito.client-id=xxxxxxxxxxxxxx
spring.security.oauth2.client.registration.cognito.client-secret=xxxxxxxxxxxx
spring.security.oauth2.client.registration.cognito.scope=openid,email,profile
spring.security.oauth2.client.registration.cognito.client-name=Cognito Google
spring.security.oauth2.client.registration.cognito.provider=cognito
spring.security.oauth2.client.provider.cognito.issuer-uri=https://cognito-idp.ap-southeast-1.amazonaws.com/ap-southeast-1_82kj6ALIs
```

> [!NOTE]
> Thay đổi giá trị `client-id`, `client-secret` và phần ID User Pool trong `issuer-uri` tương ứng với thông tin trên tài khoản AWS của bạn.
