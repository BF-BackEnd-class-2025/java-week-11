# 👤 User Registration & Login (Secure Auth Flow)

User authentication is one of the core building blocks of any backend system.
In Spring Boot, we build this with:

- A registration endpoint → `/auth/register`
- A login endpoint → `/auth/login`
- Secure password hashing (BCrypt)
- Clear error messages for invalid input
- JWT token generation on login

This ensures a safe and professional authentication flow.

---

## 🧠 Goals

Students will learn how to:

- Create secure registration & login endpoints
- Hash passwords before saving
- Validate user input
- Check credentials during login
- Return meaningful error messages
- Generate a JWT token after successful login

---

# 📝 1. Registration (`POST /auth/register`)

The registration endpoint creates a new user account.
Security rules:

- Password --must-- be hashed
- Username must be unique
- Validate fields before saving
- Never return the password (hashed or not)

---

## 📥 Registration Request Body

```json
{
  "username": "sam123",
  "password": "mySecret123"
}
```

---

## 📤 Registration Response (Success)

```json
{
  "message": "User registered successfully"
}
```

---

# 🧪 Input Validation Rules

| Field               | Rule                           |
|---------------------|--------------------------------|
| username            | Required, minimum 3 characters |
| password            | Required, minimum 6 characters |
| username uniqueness | Cannot already exist           |

---

## 🔧 Registration DTO

```java
public class RegisterRequestDTO {
    private String username;
    private String password;
    // getters + setters
}
```

---

## 🧩 Registration Service

```java
@Service
public class AuthService 
{
    private final UserRepository userRepository;
    private final BCryptPasswordEncoder passwordEncoder;

    public AuthService(UserRepository userRepository,
                       BCryptPasswordEncoder passwordEncoder) 
    {
        this.userRepository = userRepository;
        this.passwordEncoder = passwordEncoder;
    }

    public void register(RegisterRequestDTO request) 
    {

        // Validate username
        if (request.getUsername() == null || request.getUsername().length() < 3) 
        {
            throw new RuntimeException("Username must be at least 3 characters");
        }

        // Validate password
        if (request.getPassword() == null || request.getPassword().length() < 6) 
        {
            throw new RuntimeException("Password must be at least 6 characters");
        }

        // Check if username exists
        if (userRepository.findByUsername(request.getUsername()).isPresent()) 
        {
            throw new RuntimeException("Username already exists");
        }

        User user = new User();
        user.setUsername(request.getUsername());

        // 🔐 Hash the password before saving
        user.setPassword(passwordEncoder.encode(request.getPassword()));

        userRepository.save(user);
    }
}
```

---

## 🕹️ Registration Controller

```java
@RestController
@RequestMapping("/auth")
public class AuthController 
{

    private final AuthService authService;

    public AuthController(AuthService authService) 
    {
        this.authService = authService;
    }

    @PostMapping("/register")
    public ResponseEntity<?> register(@RequestBody RegisterRequestDTO request) 
    {
        authService.register(request);
        return ResponseEntity.ok(Map.of("message", "User registered successfully"));
    }
}
```

---

# 🔑 2. Login (`POST /auth/login`)

The login endpoint checks credentials and returns a --JWT token-- when correct.

---

## 📥 Login Request Body

```json
{
  "username": "sam123",
  "password": "mySecret123"
}
```

---

## 📤 Login Response (Success)

```json
{
  "token": "eyJhbGciOiJIUzI1NiJ9..."
}
```

---

## ❌ Error Response (Invalid Credentials)

```json
{
  "error": "Invalid username or password"
}
```

---

# 🚦 Login Flow

1. User enters username/password
2. Backend checks if user exists
3. Verify password using BCrypt
4. If valid → generate JWT token
5. If invalid → send error response

---

## 🧩 Login DTO

```java
public class LoginRequestDTO 
{
    private String username;
    private String password;
}
```

---

## 📦 Login Service

```java
public AuthResponseDTO login(LoginRequestDTO request) 
{

    User user = userRepository.findByUsername(request.getUsername())
            .orElseThrow(() -> new RuntimeException("Invalid username or password"));

    // Check password
    if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) 
    {
        throw new RuntimeException("Invalid username or password");
    }

    // Generate JWT
    String token = jwtService.generateToken(user.getUsername());

    return new AuthResponseDTO(token);
}
```

---

## 📦 Login Response DTO

```java
public class AuthResponseDTO 
{
    private String token;

    public AuthResponseDTO(String token) 
    {
        this.token = token;
    }
}
```

---

## 🍃 Login Controller

```java
@PostMapping("/login")
public ResponseEntity<?> login(@RequestBody LoginRequestDTO request) 
{
    AuthResponseDTO response = authService.login(request);
    return ResponseEntity.ok(response);
}
```

---

# ⚠️ Error Handling (Elegant & Useful)

You should have a global exception handler to send clean error messages:

```java
@RestControllerAdvice
public class GlobalExceptionHandler 
{
    @ExceptionHandler(RuntimeException.class)
    public ResponseEntity<?> handleRuntime(RuntimeException ex) {
        return ResponseEntity
                .badRequest()
                .body(Map.of("error", ex.getMessage()));
    }
}
```

---

# 🧷 Summary

| Feature        | Description                                       |
|----------------|---------------------------------------------------|
| Registration   | Validates user input, hashes password, saves user |
| Login          | Validates credentials, returns JWT                |
| Security       | BCrypt + JWT + Spring Security filters            |
| Error handling | Returns clean and safe error messages             |

---


