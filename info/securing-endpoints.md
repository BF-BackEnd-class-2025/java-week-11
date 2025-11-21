# 🛡️ Securing Endpoints with Spring Security

Spring Security is the core framework used to protect your backend API.
In a secure Spring Boot application, every request should pass through:

1. **Authentication** — Who are you?
2. **Authorization** — What are you allowed to access?

Using **JWT tokens**, we can secure endpoints without server-side sessions.

---

## 🎯 Goals

Students will learn how to:

- Protect private routes using Spring Security
- Allow public access to `/auth/register` and `/auth/login`
- Add a **custom JWT filter** that runs before every request
- Validate tokens and set the authenticated user in the SecurityContext

---

# 🔓 1. Public vs. Private Endpoints

Some endpoints should be available to everyone:

| Endpoint              | Access              |
|-----------------------|---------------------|
| `POST /auth/register` | Public              |
| `POST /auth/login`    | Public              |
| `/swagger-ui/**`      | Public (if enabled) |

Everything else should be protected:

| Endpoint        | Access  |
|-----------------|---------|
| `GET /products` | Private |
| `POST /orders`  | Private |
| `GET /users/me` | Private |

---

# 🧰 2. Spring Security Configuration (Core of the System)

This is the heart of securing endpoints.

---

## 📦 `SecurityConfig.java`

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig 
{
    private final JwtAuthenticationFilter jwtAuthFilter;
    public SecurityConfig(JwtAuthenticationFilter jwtAuthFilter) 
    {
        this.jwtAuthFilter = jwtAuthFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception 
    {

        return http
                .csrf(csrf -> csrf.disable()) // JWT → no CSRF needed
                .sessionManagement(session -> 
                        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
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

## 🔍 What this config does

### ✔️ Disables CSRF

Because JWT is stateless — no server sessions.

### ✔️ Sets session mode to STATELESS

The server does NOT track logged-in users.

### ✔️ Public endpoints

`/auth/login`, `/auth/register` can be accessed without authentication.

### ✔️ Private endpoints

All other endpoints require a valid JWT.

### ✔️ Adds custom JWT filter

Runs **before** Spring Security’s default authentication filter.

---

# 🧪 3. Custom JWT Authentication Filter

This filter runs **on every request** and checks:

- Is there an Authorization header?
- Does it contain a valid JWT?
- Does the token’s username match a real user?
- Is the token expired?

If valid → The user becomes authenticated.

---

## 📦 `JwtAuthenticationFilter.java`

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter 
{

    private final JwtService jwtService;
    private final UserDetailsService userDetailsService;

    public JwtAuthenticationFilter(JwtService jwtService, UserDetailsService userDetailsService) 
    {
        this.jwtService = jwtService;
        this.userDetailsService = userDetailsService;
    }

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain)
            throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        // No token? → Move to next filter (endpoint stays protected)
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
                                userDetails,
                                null,
                                userDetails.getAuthorities()
                        );

                authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }

        filterChain.doFilter(request, response);
    }
}
```

---

## 💡 Responsibilities of the Filter

| Step | Action                                              |
|------|-----------------------------------------------------|
| 1    | Read Authorization header                           |
| 2    | Extract JWT from `Bearer <token>`                   |
| 3    | Extract username from JWT                           |
| 4    | Load user details                                   |
| 5    | Validate token (signature + expiration)             |
| 6    | Set authenticated user into Spring Security context |
| 7    | Continue request                                    |

---

# 🔐 4. Accessing Protected Endpoints

After logging in, the user must include the token in every request:

### Example HTTP Request

```
GET /products
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

If the token is:

- ✔️ Valid → User can access the route 
- ❌ Invalid or expired → Returns `401 Unauthorized`

---

# 🚫 5. What Happens If No Token Is Provided?

Request:

```
GET /users/me
```

No Authorization header?

→ Spring Security blocks access automatically
→ Returns:

```json
{
  "error": "Unauthorized"
}
```

---

# 🧷 Summary

| Concept         | Description                     |
|-----------------|---------------------------------|
| Public routes   | `/auth/login`, `/auth/register` |
| Private routes  | All other endpoints             |
| JWT validation  | Done by custom filter           |
| Stateless auth  | No server sessions              |
| SecurityContext | Holds the authenticated user    |

You now have a fully secure backend architecture using **Spring Security + JWT**.

---

If you’d like, I can also prepare:

- MD file for Authorization (roles & permissions)
- A diagram explaining how Spring Security filters work
- A complete starter-template project for students

