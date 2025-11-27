# 📄 **CSRF.md**

**CSRF (Cross-Site Request Forgery)** is a type of web attack where a malicious website tricks a user’s browser into sending
unintended requests to a trusted application **where the user is already logged in**.

### ✔️ CSRF depends on two facts:

1. Browsers automatically send cookies (session IDs, JWT cookies, etc.)
2. The user is already authenticated in the target application

### ✔️ Example:

If your API uses **session cookies** for authentication, and the user visits a malicious website, that website can secretly send:

```
POST https://your-api.com/transfer-money
```

And the browser **automatically adds the session cookie**, making the request look valid.

Result: an attacker can perform actions on behalf of the user.

---

# ⚠️ When is CSRF *possible*?

CSRF happens **only when authentication is stored in cookies**:

| Auth Mechanism              | CSRF Risk |
|-----------------------------|-----------|
| Session Cookies             | ❌ High    |
| JWT stored in cookies       | ❌ High    |
| JWT in Authorization Header | ✅ No CSRF |
| API Keys                    | ✅ No CSRF |
| OAuth Bearer Tokens         | ✅ No CSRF |

Most REST APIs using `Authorization: Bearer <JWT>` are **not vulnerable** because browsers do not attach Authorization header automatically.

---

# 🔥 How CSRF Attacks Work (Example)

A user is logged into your site:

```
https://bank.com
Cookie: SESSIONID=abc123
```

The attacker sends a malicious link:

```
<img src="https://bank.com/send-money?amount=5000&to=hacker" />
```

When the victim loads the page, the browser sends:

```
GET /send-money?amount=5000&to=hacker
Cookie: SESSIONID=abc123
```

The bank thinks it’s the real user!

---

# 🧰 How to Prevent CSRF in Spring Boot

There are **two main strategies**, depending on your authentication model.

---

# ✔️ **1. If your API uses JWT (Authorization Header)**

Most modern APIs use:

```
Authorization: Bearer <token>
```

This is stateless and **NOT vulnerable to CSRF** because browsers do not add this header automatically.

### ✅ Correct Spring Security configuration (CSRF disabled for APIs)

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception
{

    http.csrf(csrf -> csrf.disable())             // Disable CSRF for JWT-based APIs
            .authorizeHttpRequests(auth -> auth
                    .requestMatchers("/auth/**").permitAll()               // Public: register/login
                    .requestMatchers(HttpMethod.GET, "/movies/**").permitAll() // Public: view movies
                    .requestMatchers("/movies/**").hasRole("ADMIN")        // Protected: CRUD (admin only)
                    .anyRequest().authenticated()                          // Everything else requires login
            )
            // Run JWT filter before default UsernamePasswordAuthenticationFilter
            .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);

    return http.build();
}
```

### Why is it safe?

* Your API does not use cookies
* No automatic browser authentication
* Requests must include Authorization header manually

---

# ✔️ **2. If you use cookies for authentication (Sessions or JWT Cookies)**

Then CSRF **must be enabled** and protected.

Spring provides two CSRF protection strategies:

---

# ➤ **A. Synchronizer Token Pattern (STP)**

Spring Security generates a random CSRF token and stores it in the session.

Flow:

1. Server sends a CSRF token to the client
2. Client sends the token back in a header (e.g., `X-CSRF-TOKEN`)
3. Server checks:

    * cookie/session belongs to user
    * CSRF token matches

### Enable CSRF Token:

```java
http
    .csrf()
    .csrfTokenRepository(CookieCsrfTokenRepository.withHttpOnlyFalse());
```

Spring sends:

```
Set-Cookie: XSRF-TOKEN=abc123
```

Client must return:

```
X-XSRF-TOKEN: abc123
```

---

# ➤ **B. SameSite Cookies**

Modern browsers support:

```
Set-Cookie: SESSIONID=123; SameSite=Lax
```

* `Strict`: best security
* `Lax`: prevents most CSRF
* `None`: only when using cross-site cookies (must use HTTPS)

---

# ✔️ **3. Double Submit Cookie Pattern**

The server sends a cookie:

```
Set-Cookie: csrfToken=abc
```

The client copies it to a header:

```
X-CSRF-TOKEN: abc
```

Server verifies both match.

This works even without sessions.

---

# 🛑 When Should You Disable CSRF in Spring?

Disable CSRF **only** when both are true:

### ✔️ You are building a REST API

### ✔️ You use tokens in Authorization header (NOT cookies)

This is exactly how most modern Spring Boot + React/Vue projects work.

**Your movie API falls in this category**, so disabling CSRF is correct.

---

# 🧪 Quick Decision Guide

| Scenario                          | Should Enable CSRF? |
|-----------------------------------|---------------------|
| Spring MVC web app using sessions | ✅ Yes               |
| REST API using cookies            | ✅ Yes               |
| REST API using JWT header         | ❌ No                |
| Mobile app API                    | ❌ No                |
| Microservices communication       | ❌ No                |

---

# 🧩 Example: Proper SecurityConfig for JWT-Based API

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig 
{
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception 
    {
        http
            .csrf().disable()  // not needed for stateless JWT APIs
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .authorizeHttpRequests()
                .anyRequest().authenticated();

        return http.build();
    }
}
```

---

# 📌 Summary

### ✔️ CSRF = Attack that abuses cookie-based authentication

### ✔️ APIs using JWT in headers are **not vulnerable**

### ✔️ Disable CSRF for stateless APIs

### ✔️ Enable CSRF ONLY when using session/cookie login

### ✔️ Spring supports CSRF prevention automatically

### ✔️ Tokens > Cookies (safer for APIs)

---
