# 🔐 Password Hashing with BCrypt

Storing user passwords securely is one of the most important responsibilities in any backend application.
Instead of saving plain-text passwords — which would be disastrous if your database is exposed — you **hash** them.

This ensures that even if someone gains access to the database, they cannot recover the original passwords.

---

## 🧠 What Is Password Hashing?

**Hashing** is a one-way mathematical transformation:

```
plain password  →  hash function  →  hashed password
```

* You **cannot reverse** a hash back into the original password.
* The same input always produces the same output *(for the same salt)*.
* Secure hashing uses **salt** (random data) to prevent attacks like:

    * Rainbow table attacks
    * Precomputed dictionary attacks
    * Identical passwords resulting in identical hashes

---

## 🔒 Why Use BCrypt?

**BCrypt** is one of the most recommended password hashing algorithms because:

- ✔️ It automatically generates a **unique salt** for each password
- ✔️ It’s **slow by design** to resist brute-force attacks
- ✔️ It’s widely supported and audited
- ✔️ Easy to use in Spring Boot

---

## 🧩 BCrypt in Spring Boot

Spring Security provides a built-in class:

```java
org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
```

To hash a password:

```java
String hashed = passwordEncoder.encode("myPassword123");
```

To verify a password during login:

```java
boolean match = passwordEncoder.matches("myPassword123", hashed);
```

---

## 🏗️ Full Working Example

### 1️⃣ Add BCrypt Bean (Spring Boot)

```java
@Configuration
public class AppConfig 
{
    @Bean
    public BCryptPasswordEncoder passwordEncoder() 
    {
        return new BCryptPasswordEncoder();
    }
}
```

### 2️⃣ User Entity (store only hashed passwords)

```java
@Entity
public class User 
{
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;

    // store hashed password, never plain text!
    private String password;

    // getters & setters
}
```

### 3️⃣ Registration Service (Hash Before Save)

```java
@Service
public class AuthService 
{
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    public AuthService(UserRepository userRepository, BCryptPasswordEncoder passwordEncoder) 
    {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public void register(String username, String rawPassword) 
    {
        User user = new User();
        user.setUsername(username);

        // 🔐 Hash the password before saving
        user.setPassword(passwordEncoder.encode(rawPassword));

        userRepository.save(user);
    }
}
```

### 4️⃣ Login: Compare Raw vs Hashed Password

```java
public boolean login(String username, String rawPassword) 
{
    User user = userRepository.findByUsername(username)
            .orElseThrow(() -> new RuntimeException("User not found"));

    // Compare raw password with hashed password
    boolean isValid = passwordEncoder.matches(rawPassword, user.getPassword());

    return isValid;
}
```

---

## 🧪 Example Input/Output

### User registers:

```
Password entered:   mySecret123
BCrypt hash stored: $2a$10$J9p5EiRYoMc.7u8Iikf8YutTD...
```

### User logs in:

```
Raw password:       mySecret123
Database hash:      $2a$10$J9p5EiRYoMc.7u8Iikf8YutTD...
Matches?            ✔️ Yes
```

Even if two users choose the same password, BCrypt generates **different hashes**:

```
$2a$10$Jp0r... 
$2a$10$9UvE...
```

That’s the salt at work.

---

## 🛡️ Key Rules for Password Storage

| Rule                                      | Explanation                              |
|-------------------------------------------|------------------------------------------|
| ❌ Never store plain passwords             | Always hash first                        |
| ❌ Never create your own hashing algorithm | Use battle-tested algorithms like BCrypt |
| ❌ Never compare raw strings               | Use `passwordEncoder.matches()`          |
| ✔️ Always hash **before** saving to DB    | Store only the hashed string             |

---

## 🧷 Summary

BCrypt gives you:

- Secure password hashing
- Automatic salting
- Resistance to brute-force
- Easy integration with Spring Security

It's the industry-standard approach for safe authentication systems — and a core skill your students should master.

---
