# 🔐 JWT Authentication (JSON Web Tokens)

**JWT (JSON Web Token)** is a compact, secure, and stateless way to handle user authentication in modern web applications.

Instead of storing session data on the server, JWT stores user identity inside a signed token that the client sends with every request.

---

## 🧠 What Is a JWT?

A JWT is a **string token** with three parts:

```
xxxxx.yyyyy.zzzzz
```

| Part | Name      | Meaning                                    |
|------|-----------|--------------------------------------------|
| 1️⃣  | Header    | Token type + signing algorithm             |
| 2️⃣  | Payload   | Claims → user data (e.g., username, roles) |
| 3️⃣  | Signature | Ensures the token wasn't tampered with     |

### Example JWT (shortened):

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJzYW0iLCJpYXQiOjE3MzAxODY1MDB9.r8R08j4...
```

---

## 🧩 How JWT Works

### 1️⃣ User logs in

User provides username & password.

### 2️⃣ Server validates password

If valid → server creates JWT.

### 3️⃣ Server sends JWT to user

Client stores token (usually in localStorage).

### 4️⃣ Client sends JWT with each request

Sent in the request header:

```
Authorization: Bearer <token>
```

### 5️⃣ Server checks token

If valid → request proceeds.
If invalid or expired → return 401 Unauthorized.

---

## 🧷 Why JWT?

| Benefit                        | Description                         |
|--------------------------------|-------------------------------------|
| ✔️ Stateless                   | No session storage on server        |
| ✔️ Fast                        | Token validation is lightweight     |
| ✔️ Secure                      | Signed tokens prevent tampering     |
| ✔️ Easy to use                 | Works across languages & frameworks |
| ✔️ Great for mobile & SPA apps | Ideal for React, Vue, mobile apps   |

---

## 📦 Creating JWT in Spring Boot

### 1️⃣ Add the JWT Dependency

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
```

---

## 🛠️ Example: `JwtService.java`

```java
@Service
public class JwtService 
{
    private static final String SECRET_KEY = "your-secret-key-change-me-please";
    public String generateToken(String username) 
    {
        return Jwts.builder()
                .setSubject(username)
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 60)) // 1 hour
                .signWith(getSigningKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public String extractUsername(String token) 
    {
        return Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody()
                .getSubject();
    }

    public boolean isTokenValid(String token, String username) 
    {
        return username.equals(extractUsername(token)) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) 
    {
        Date expiration = Jwts.parserBuilder()
                .setSigningKey(getSigningKey())
                .build()
                .parseClaimsJws(token)
                .getBody()
                .getExpiration();

        return expiration.before(new Date());
    }

    private Key getSigningKey() 
    {
        byte[] keyBytes = SECRET_KEY.getBytes(StandardCharsets.UTF_8);
        return Keys.hmacShaKeyFor(keyBytes);
    }
}
```

---

## 📡 Using JWT in Login Flow

### **AuthService.java**

```java
public AuthResponseDTO login(LoginRequestDTO request) 
{
    User user = userRepository.findByUsername(request.getUsername())
            .orElseThrow(() -> new RuntimeException("User not found"));

    if (!passwordEncoder.matches(request.getPassword(), user.getPassword())) 
    {
        throw new RuntimeException("Invalid password");
    }

    String jwt = jwtService.generateToken(user.getUsername());

    return new AuthResponseDTO(jwt);
}
```

---

## 🛡️ Securing Requests with a JWT Filter

Every request must be inspected to check if the token is valid.

### **JwtAuthenticationFilter.java**

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter 
{

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if (authHeader == null || !authHeader.startsWith("Bearer ")) 
        {
            filterChain.doFilter(request, response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUsername(token);

        if (username != null &&
            SecurityContextHolder.getContext().getAuthentication() == null) 
        {

            UserDetails userDetails = userDetailsService.loadUserByUsername(username);

            if (jwtService.isTokenValid(token, userDetails.getUsername())) 
            {
                UsernamePasswordAuthenticationToken authToken =
                        new UsernamePasswordAuthenticationToken(
                                userDetails, null, userDetails.getAuthorities());

                authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request));

                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

---

## 🧰 Security Configuration

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig 
{
    private final JwtAuthenticationFilter jwtAuthFilter;
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception 
    {

        return http
                .csrf(csrf -> csrf.disable())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/auth/login", "/auth/register").permitAll()
                        .anyRequest().authenticated()
                )
                .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
                .build();
    }
}
```

---

## 🧪 Example Request

### Request (from client):

```
GET /products
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

### Server verifies:

- Signature ✔️
- Expiration ✔️
- Username ✔️

If valid → return data
If invalid → `401 Unauthorized`

---

## 🧷 Summary

**JWT makes authentication:**

- Secure
- Fast
- Stateless
- Scalable
- Easy to integrate with Spring Boot

It is the most widely used method for securing APIs in modern applications, especially with SPA frameworks like **React, Vue, Angular**, and mobile apps.

---

