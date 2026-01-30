# Custom Authentication System
**Module 23 Assignment - Spring Boot JWT Authentication with Email Verification**

## 📋 Overview
A complete backend authentication system built with Spring Boot 4.0.2 featuring JWT-based authentication, email verification with unique links, and comprehensive security measures.

## ✨ Features

### 🔐 Core Authentication
- **JWT-based Authentication** - Stateless, token-based user sessions
- **Email Verification Required** - Users must verify email before login
- **Secure Password Storage** - BCrypt password encryption
- **Protected API Endpoints** - JWT validation on protected routes

### 📧 Email Verification System
- **Unique Verification Links** - UUID-based tokens
- **10-Minute Expiry** - Time-limited verification tokens
- **One-Time Use Tokens** - Tokens invalidated after verification
- **Automatic Email Sending** - JavaMailSender with Gmail SMTP

### 🛡️ Security Features
- **5-Minute Email Throttling** - Prevents verification email spam
- **Old Token Invalidation** - Previous tokens disabled on new email send
- **Input Validation** - Bean validation on all API requests
- **Database Persistence** - PostgreSQL for production-ready storage

## 🏗️ Architecture

### Technology Stack
- **Framework:** Spring Boot 4.0.2
- **Language:** Java 21
- **Database:** PostgreSQL
- **Authentication:** JWT (jjwt 0.13.0)
- **Email:** JavaMailSender (Gmail SMTP)
- **API Documentation:** Swagger/OpenAPI 3
- **Build Tool:** Gradle (Kotlin DSL)

### Project Structure
```
src/main/java/root/cyb/mhr/CustomAuthentication/
├── config/
│   └── SecurityConfig.java          # Spring Security + JWT configuration
├── controller/
│   └── AuthController.java          # REST API endpoints
├── dto/
│   ├── AuthResponse.java            # API response object
│   ├── LoginRequest.java            # Login request validation
│   └── RegisterRequest.java         # Registration request validation
├── entity/
│   ├── User.java                    # User domain model
│   └── VerificationToken.java       # Token domain model
├── exception/
│   └── GlobalExceptionHandler.java  # Global error handling
├── filter/
│   └── JwtAuthenticationFilter.java # JWT request filter
├── repository/
│   ├── UserRepository.java          # User data access
│   └── VerificationTokenRepository.java
├── service/
│   ├── AuthService.java             # Business logic
│   ├── CustomUserDetailsService.java
│   └── EmailService.java            # Email sending service
└── util/
    └── JwtUtil.java                 # JWT utility functions
```

## 🚀 Setup & Installation

### Prerequisites
- **Java 21** or higher
- **PostgreSQL** database
- **Gmail account** with App Password (for email sending)
- **Gradle** (or use included Gradle wrapper)

### 1. Clone Repository
```bash
git clone <repository-url>
cd CustomAuthentication
```

### 2. Configure Database
Create PostgreSQL database:
```sql
CREATE DATABASE acuthdb;
```

### 3. Configure Application
Edit `src/main/resources/application.properties`:

```properties
# Server Configuration
server.port=8085

# PostgreSQL Database
spring.datasource.url=jdbc:postgresql://localhost:5432/acuthdb
spring.datasource.username=YOUR_DB_USERNAME
spring.datasource.password=YOUR_DB_PASSWORD

# Email Configuration (Gmail)
spring.mail.username=YOUR_EMAIL@gmail.com
spring.mail.password=YOUR_APP_PASSWORD

# JWT Secret (Change in production!)
jwt.secret=YOUR_SECURE_SECRET_KEY_HERE_MINIMUM_256_BITS
jwt.expiration=86400000

# Application Base URL
app.base.url=http://localhost:8085
```

### 4. Generate Gmail App Password
1. Go to Google Account Settings
2. Security → 2-Step Verification → App passwords
3. Generate password for "Mail"
4. Copy 16-character password to `spring.mail.password`

**Reference:** [Gmail App Password Guide](https://medium.com/@AlexanderObregon/sending-emails-from-a-spring-boot-application-3cba9b051dbd)

### 5. Build & Run
```bash
# Using Gradle Wrapper (Windows)
./gradlew bootRun

# Using Gradle Wrapper (Linux/Mac)
./gradlew bootRun

# Or build JAR
./gradlew build
java -jar build/libs/CustomAuthentication-0.0.1-SNAPSHOT.jar
```

Application will start on: `http://localhost:8085`

## 📡 API Endpoints

### Authentication Endpoints

#### 1. Register User
```http
POST /auth/register
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response (200 OK):**
```json
{
  "message": "Registration successful. Please check your email to verify your account.",
  "token": null,
  "email": null
}
```

#### 2. Verify Email
```http
GET /auth/verify?token={uuid-token}
```

**Response (200 OK):**
```json
{
  "message": "Email verified successfully. You can now login.",
  "token": null,
  "email": "user@example.com"
}
```

#### 3. Login
```http
POST /auth/login
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}
```

**Response (200 OK):**
```json
{
  "message": "Login successful",
  "token": "eyJhbGciOiJIUzI1NiJ9...",
  "email": "user@example.com"
}
```

#### 4. Test Protected Endpoint
```http
GET /auth/test
Authorization: Bearer {jwt-token}
```

**Response (200 OK):**
```json
{
  "message": "This is a protected endpoint. You are authenticated!",
  "token": null,
  "email": null
}
```

### API Documentation
Interactive Swagger UI available at:
```
http://localhost:8085/swagger-ui.html
```

## 🔬 Testing

### Using PowerShell
```powershell
# Register
Invoke-WebRequest -Uri "http://localhost:8085/auth/register" `
    -Method POST -ContentType "application/json" `
    -Body '{"email":"test@example.com","password":"password123"}'

# Verify (use token from email/logs)
Invoke-WebRequest -Uri "http://localhost:8085/auth/verify?token=YOUR_TOKEN" -Method GET

# Login
$response = Invoke-WebRequest -Uri "http://localhost:8085/auth/login" `
    -Method POST -ContentType "application/json" `
    -Body '{"email":"test@example.com","password":"password123"}' `
    -UseBasicParsing

# Protected endpoint
$token = ($response.Content | ConvertFrom-Json).token
Invoke-WebRequest -Uri "http://localhost:8085/auth/test" `
    -Method GET -Headers @{"Authorization"="Bearer $token"}
```

### Using cURL
```bash
# Register
curl -X POST http://localhost:8085/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Verify
curl http://localhost:8085/auth/verify?token=YOUR_TOKEN

# Login
curl -X POST http://localhost:8085/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"test@example.com","password":"password123"}'

# Protected endpoint
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  http://localhost:8085/auth/test
```

## 📌 Assignment Requirements Compliance

| Requirement | Status | Implementation |
|-------------|--------|----------------|
| ✅ JWT Authentication | Complete | jjwt library with token generation & validation |
| ✅ Unique Verification Link | Complete | UUID-based tokens in GET endpoint |
| ✅ Email Sending (Proper Format) | Complete | JavaMailSender with Gmail SMTP |
| ✅ Login After Verification Only | Complete | User.verified flag check |
| ✅ Resend Link on Unverified Login | Complete | Auto-resend with new token |
| ✅ 5-Min Email Throttling | Complete | Timestamp-based throttling |
| ✅ Old Link Invalidation | Complete | Token.used flag marking |
| ✅ Non-In-Memory Database | Complete | PostgreSQL persistence |
| ✅ Input Validation | Complete | Bean validation on DTOs |

## 🎯 Key Implementation Details

### JWT Token Flow
1. User logs in successfully
2. Server generates JWT with user email and expiration
3. Client stores token (localStorage, sessionStorage, etc.)
4. Client sends token in `Authorization: Bearer {token}` header
5. Server validates token on each protected request

### Email Verification Flow
1. User registers → `verified = false`
2. System generates UUID token → Expiry = 10 minutes
3. Email sent with verification link
4. User clicks link → Token validated → `verified = true`
5. User can now login and receive JWT

### Email Throttling Logic
1. Track `lastEmailSentAt` timestamp per user
2. On unverified login attempt, check time difference
3. If < 5 minutes → Return error with remaining time
4. If ≥ 5 minutes → Send new email, invalidate old token

### Token Invalidation
1. Old token marked `used = true` in database
2. Verification endpoint checks `token.isUsed()`
3. If used → Return error
4. Ensures one-time use and prevents reuse attacks

## 🔒 Security Considerations

- ✅ Passwords hashed with BCrypt
- ✅ JWT tokens signed with HMAC-SHA256
- ✅ Stateless sessions (no server-side session storage)
- ✅ CSRF protection disabled (not needed for JWT)
- ✅ Email verification prevents fake accounts
- ✅ Token expiry prevents indefinite token validity
- ✅ Input validation prevents injection attacks
- ✅ Unique constraints on email and token

## 📚 Dependencies

```kotlin
dependencies {
    implementation("org.springframework.boot:spring-boot-starter-data-jpa")
    implementation("org.springframework.boot:spring-boot-starter-security")
    implementation("org.springframework.boot:spring-boot-starter-webmvc")
    implementation("org.springframework.boot:spring-boot-starter-mail")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    
    runtimeOnly("org.postgresql:postgresql")
    
    // JWT
    implementation("io.jsonwebtoken:jjwt-api:0.13.0")
    runtimeOnly("io.jsonwebtoken:jjwt-impl:0.13.0")
    runtimeOnly("io.jsonwebtoken:jjwt-jackson:0.13.0")
    
    // API Documentation
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.4")
}
```

## 🐛 Troubleshooting

### Port Already in Use
Change port in `application.properties`:
```properties
server.port=8086
app.base.url=http://localhost:8086
```

### Email Not Sending
1. Verify Gmail app password is correct
2. Enable 2-step verification in Google Account
3. Check `spring.mail.username` matches Gmail address
4. Ensure port 587 is not blocked by firewall

### Database Connection Failed
```bash
# Check PostgreSQL is running
sudo systemctl status postgresql  # Linux
brew services list                # Mac
services.msc                      # Windows

# Test connection
psql -U postgres -d acuthdb
```

### JWT Token Invalid
- Check token hasn't expired (24-hour default)
- Verify `jwt.secret` matches between requests
- Ensure `Authorization: Bearer ` prefix is included

## 👨‍💻 Author
**Mahmudul Hasan**  
Module 23 - Spring Boot Custom Authentication Assignment

## 📄 License
This project is for educational purposes as part of a coding assignment.

---

**Note:** For production deployment, update JWT secret, use environment variables, enable HTTPS, and configure proper CORS policies.
