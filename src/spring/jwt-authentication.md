# JWT (JSON Web Token) Authentication

JWT provides **stateless authentication** — the server does *not* store user sessions.
The client holds the token and sends it with every request.

---

## What is a JWT?
A JWT is a **digitally signed token** that represents user identity.

The client sends it with requests:

```txt
Authorization: Bearer <token>
```

---

## JWT Typically Contains:
- **User ID**
- **Expiration timestamp**
- **Signature** that verifies it hasn’t been tampered with

---

## JWT Authentication Steps

1. User logs in successfully.
2. Server generates and signs JWT token.
3. Client stores token (localStorage / secure cookie).
4. For every request, client includes token in `Authorization` header.
5. Server:
   - Validates the token
   - Extracts user identity
   - Allows or denies access

---

## Why JWT Instead of Sessions?

| Sessions                             | JWT                                     |
|--------------------------------------|-----------------------------------------|
| Server stores session                | Server stores nothing (stateless)       |
| Harder to scale across servers       | Easy to scale (no shared session store) |
| Works best with traditional web apps | Works best with modern SPAs & APIs      |

---

