# Password Hashing with BCrypt

## Why Hash Passwords?
Storing plain-text passwords is dangerous. If your database is leaked, all user accounts are exposed.

Instead, we store a **hashed** version of the password — this process is **one-way**, meaning we can **verify** a password without ever needing to reverse the hash.

BCrypt:
- Automatically generates a **salt** (extra protection)
- Designed to be **slow** on purpose to resist brute-force attacks

---

## Using BCrypt in Spring Boot

```java
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;

BCryptPasswordEncoder encoder = new BCryptPasswordEncoder();

String hashedPassword = encoder.encode(rawPassword);

// To verify during login:
boolean matches = encoder.matches(rawPassword, hashedPassword);
```

### ✅ Key Points

* Each `encode()` call creates a **unique** hash (even for the same password!)
* Always store the **hashed** password, never the plain one
* Use `.matches()` during login — **do not compare strings manually**

---



