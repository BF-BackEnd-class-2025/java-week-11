# User Registration & Login

Authentication requires two main actions:
- **Registration** → Create a new user account
- **Login** → Verify user identity and start a session (or issue a token)

---

## Registration Flow

1. Client sends: `name`, `email`, `password`.
2. Validate input using DTO + Bean Validation.
3. Hash password using **BCrypt**.
4. Save the user to the database.

---

## Login Flow

1. User submits: `email`, `password`.
2. Lookup user by email.
3. Compare submitted password with stored hashed password:

```java
   encoder.matches(rawPassword, storedHashedPassword)
```

1. If correct → Generate a **JWT token**.
2. Send back token to client so they can access protected endpoints.

---

## API Endpoints (Typical)

| Method | Endpoint            | Description           | Public/Protected |
|--------|---------------------|-----------------------|------------------|
| POST   | `/auth/register`    | Create new user       | Public           |
| POST   | `/auth/login`       | Return JWT token      | Public           |
| GET    | `/api/user/profile` | Get current user data | **Protected**    |

---
