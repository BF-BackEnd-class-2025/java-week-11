# Securing Endpoints with Spring Security

Spring Security controls **who** can access **which** endpoints.

---

## Public vs Protected Endpoints

| Route            | Description           | Access                         |
|------------------|-----------------------|--------------------------------|
| `/auth/register` | User signup           | Public                         |
| `/auth/login`    | User login            | Public                         |
| `/api/**`        | Application resources | **Protected (requires token)** |

---

## Core Security Steps

1. **Disable** default Spring Security login UI.
2. Add a **JWT authentication filter** that:
   - Reads `Authorization` header
   - Validates token
   - Loads user identity
3. Mark protected endpoints so only authenticated users may access them.

---

## High-level Example (Conceptual)

```java
http
  .csrf().disable()
  .authorizeHttpRequests()
    .requestMatchers("/auth/**").permitAll() // public
    .anyRequest().authenticated()            // protected
  .and()
  .addFilterBefore(jwtFilter, UsernamePasswordAuthenticationFilter.class);
```

---

## Result

✅ Only users with **valid JWT tokens** can call protected API endpoints.
❌ Requests without valid tokens receive `401 Unauthorized`.

---