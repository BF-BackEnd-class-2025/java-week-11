# 📂 Repository Layer

The **repository layer** is responsible for all communication with the database.
Thanks to **Spring Data JPA**, these repositories automatically get all CRUD operations without writing a single line of SQL.

Below are the repositories used in your Movie API:

---

# 🎬 MovieRepository (Movie Data Access)

Handles all database operations for the `Movie` entity.

```java
package com.example.demo.repository;

import com.example.demo.model.Movie;
import org.springframework.data.jpa.repository.JpaRepository;

// Repository responsible for CRUD operations on Movie entities.
public interface MovieRepository extends JpaRepository<Movie, Long>
{
    /*
     * Extending JpaRepository<Movie, Long> gives you:
     *
     * save(movie)       → insert or update a movie
     * findById(id)      → find a movie by its ID
     * findAll()         → return all movies
     * deleteById(id)    → remove a movie
     * existsById(id)    → check if a movie exists
     *
     * Spring generates the implementation behind the scenes.
     * No SQL. No coding.
     *
     * You can add custom queries by naming conventions.
     * Example:
     * List<Movie> findByGenre(String genre);
     */
}
```

---

# 👤 UserRepository (User Lookups & Authentication)

Used to register users, check for duplicate emails, and load users for login/authentication.

```java
package com.example.demo.repository;

import com.example.demo.model.User;
import org.springframework.data.jpa.repository.JpaRepository;

// Repository responsible for all database operations related to users.
public interface UserRepository extends JpaRepository<User, Long>
{
    /*
     * existsByEmail(email)
     * ---------------------
     * Checks if a user with the given email already exists.
     * Used during registration to prevent duplicates.
     */
    boolean existsByEmail(String email);

    /*
     * findByEmail(email)
     * -------------------
     * Retrieves a user by email.
     * Used during:
     *  - Login (AuthService)
     *  - JWT authentication (JwtAuthenticationFilter)
     */
    User findByEmail(String email);
}
```

---

# 🧠 How Spring Data JPA Helps

By extending `JpaRepository`, both repositories automatically gain:

✔ Full CRUD operations
✔ Pagination & sorting support
✔ Ready-to-use transaction handling
✔ Auto-generated custom queries based on method names

No SQL required.

---

# 🎯 Why This Layer Matters

The repository layer keeps your project:

* **Clean** → No SQL in controllers or services
* **Maintainable** → Logic stays separated
* **Scalable** → Easy to add more queries later
* **Secure** → UserRepository integrates with JWT authentication

---

If you'd like, I can also prepare:

✔ README for the **Service layer**
✔ README for the **Controller layer**
✔ README for the **Model (entity) layer**
✔ Full architecture diagram

Just say: **"Write the service layer readme"** or whichever you want.
