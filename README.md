# ✅ **Week 11: Authentication & Security** 

This week introduces **user authentication** and **API security** in Spring Boot.
Students will learn how to register and authenticate users securely, using:

- **Password hashing** with BCrypt
- **Login & Token-based authentication**
- **JWT (JSON Web Tokens)** for stateless session management
- **Spring Security** to protect API endpoints

By the end of this week, students will build APIs where only authenticated users can access protected routes.

---

## 📚 Learning Objectives

Students will be able to:

- Securely **store user passwords** using hashing (BCrypt)
- Implement **User Registration** & **Login** endpoints
- Generate and validate **JWT tokens**
- Apply **Spring Security filters & configurations**
- Restrict access to API endpoints using **authentication & authorization rules**

---

## 🧠 Key Concepts

### 1. Password Hashing (BCrypt)

- Never store raw passwords — only store **hashed** passwords.
- BCrypt automatically handles salt generation and secure hashing.

### 2. User Registration & Login

- A secure `POST /auth/register` and `POST /auth/login` flow
- Validate credentials and return error messages for invalid input

### 3. JWT Authentication

- JWT tokens store user identity
- Sent with each request in `Authorization: Bearer <token>`
- Eliminates the need for server-side sessions

### 4. Securing Endpoints with Spring Security

- Require authentication to access private routes
- Leave some routes public (e.g., login/register)
- Custom filter to validate JWTs per request

---

## 🏗️ Example Project Structure

```txt
spring-boot-security-demo/
├── controller/
│   └── AuthController.java
├── service/
│   ├── AuthService.java
│   └── JwtService.java
├── repository/
│   └── UserRepository.java
├── dto/
│   ├── RegisterRequestDTO.java
│   ├── LoginRequestDTO.java
│   └── AuthResponseDTO.java
├── model/
│   └── User.java
├── security/
│   ├── JwtAuthenticationFilter.java
│   ├── SecurityConfig.java
│   └── JwtUtils.java
├── exception/
│   ├── GlobalExceptionHandler.java
│   └── AuthExceptions.java
└── resources/
    └── application.properties
```

---

## 🧰 Dependencies (add to `pom.xml`)

```xml
<dependencies>
    <!-- Spring Web -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>

    <!-- Spring Security -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-security</artifactId>
    </dependency>

    <!-- JWT -->
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-api</artifactId>
        <version>0.11.5</version>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-impl</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>io.jsonwebtoken</groupId>
        <artifactId>jjwt-jackson</artifactId>
        <version>0.11.5</version>
        <scope>runtime</scope>
    </dependency>

    <!-- BCrypt password hashing -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-crypto</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- DB Driver -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>
</dependencies>
```

---

## ✅ End of Week Expected Skills

| Skill                        | Outcome                              |
|------------------------------|--------------------------------------|
| Secure password storage      | Using BCrypt hashing                 |
| Login authentication         | Validate credentials and return JWT  |
| Stateless session management | Use JWT for identity across requests |
| Endpoint protection          | Require token for protected routes   |

---




