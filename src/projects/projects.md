# 🚀 **Full Spring Boot Projects (Security + CRUD + Real Features)**

All projects follow this folder structure:

```txt
spring-boot-project/
├── controller/
├── service/
├── repository/
├── dto/
├── model/
├── security/
├── mapper/
├── exception/
├── SpringBootProjectApplication.java
```

---

# 📦 **Dependencies for ALL Projects**

Same dependencies for every project:

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

    <!-- BCrypt hashing -->
    <dependency>
        <groupId>org.springframework.security</groupId>
        <artifactId>spring-security-crypto</artifactId>
    </dependency>

    <!-- JPA -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>

    <!-- PostgreSQL -->
    <dependency>
        <groupId>org.postgresql</groupId>
        <artifactId>postgresql</artifactId>
        <scope>runtime</scope>
    </dependency>

    <!-- Validation -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-validation</artifactId>
    </dependency>
</dependencies>
```

---

# 🧩 **PROJECT 1 — Secure Book Management API**

### **Features**

* User Registration / Login (JWT + BCrypt)
* Users can manage their own books
* CRUD:

    * `POST /books`
    * `GET /books`
    * `GET /books/{id}`
    * `PUT /books/{id}`
    * `DELETE /books/{id}`

### **Rules**

* Only authenticated users can access book routes
* A user can **only modify their own books**
* Optional: Add pagination

### **Models**

```txt
User
Book (user_id, title, author, publishedYear)
```

---

# 🧩 **PROJECT 2 — Secure Movie Library + Admin Role**

### **Features**

* JWT Auth + Roles: ADMIN, USER
* CRUD:

    * `/movies` CRUD for ADMIN
    * USERS can only view movies

### **Routes**

* ADMIN:

    * `POST /movies`
    * `PUT /movies/{id}`
    * `DELETE /movies/{id}`
* PUBLIC:

    * `GET /movies`

### **Security**

* Protect admin routes using `ROLE_ADMIN`
* JWT includes "role" claim

---

# 🧩 **PROJECT 3 — Product Inventory System (User Ownership)**

### **Features**

* Users have personal inventory
* CRUD:

    * Create product
    * Edit product
    * Delete product
    * List my products

### **Rules**

* You cannot edit/delete someone else’s product
* Add request validation

### **Models**

```txt
Product (id, userId, name, price, quantity)
```

---

# 🧩 **PROJECT 4 — Secure Task Management (Todo App)**

### **Features**

* JWT Login / Register
* CRUD tasks:

    * `POST /tasks`
    * `GET /tasks`
    * `PUT /tasks/{id}`
    * `DELETE /tasks/{id}`
* Tasks belong to users
* Add "status" (TODO / IN_PROGRESS / DONE)

### **Extensions**

* Add filtering: `/tasks?status=DONE`
* Due dates

---

# 🧩 **PROJECT 5 — Secure Notes App (Encrypted Notes Optional)**

### **Features**

* JWT Auth + BCrypt
* CRUD notes:

    * Users manage personal notes
* Notes include:

    * title
    * content
    * createdAt

### **Security Bonus**

* Add server-side encryption using AES (optional)

---

# 🧩 **PROJECT 6 — Online Courses API (Teacher + Student Roles)**

### **Features**

* Teachers can create/update/delete courses
* Students can view courses only
* CRUD:

    * `/courses` — Teacher only
    * `/courses/{id}` — Teacher only
    * `/courses/public` — Anyone

### **Models**

```
Course (id, teacherId, name, description)
```

### **Authorization**

* Only course owner teacher can edit/delete

---

# 🧩 **PROJECT 7 — Comments System with Post Ownership**

### **Features**

* Users can create posts
* Other users can comment
* CRUD:

    * Posts CRUD (owner only)
    * Comments CRUD (comment owner only)

### **Routes**

```
POST /posts
GET /posts
PUT /posts/{id}
DELETE /posts/{id}

POST /posts/{id}/comments
PUT /comments/{id}
DELETE /comments/{id}
```

### **Security**

* Posts & Comments ownership checks
* JWT required for everything except GET posts

---

# 🧩 **PROJECT 8 — User Profile + Avatar Upload (Protected Files)**

### **Features**

* Authenticated users can upload avatar images
* Secure file upload
* Limit size & types
* CRUD User profiles:

    * update name, bio
    * update profile picture
    * get profile

### **Security**

* Only owner can update profile
* Only authenticated users can upload files

---

# 🧩 **PROJECT 9 — Order + Item E-commerce Backend**

### **Features**

* JWT authentication
* CRUD Products (ADMIN)
* CRUD Orders (USER)
* Users can:

    * Add items to order
    * Remove items
    * Submit order

### **Routes**

```
POST /orders
POST /orders/{id}/items
DELETE /orders/{orderId}/items/{itemId}
GET /orders
```

### **Security**

* Users can only access their own orders
* Admin can see all orders

---

# 🧩 **PROJECT 10 — Secure Event Management (User + Admin + Organizer)**

### **Roles**

* USER: can register for events
* ORGANIZER: can create/update events
* ADMIN: can delete any event

### **Features**

* CRUD Events
* Register / unregister for events
* List attendees

### **Models**

```
Event
Registration
User
```

### **Security Rules**

* Organizer can only edit their events
* User can only register themselves
* Admin can delete anything

---
Happy coding! 🚀
