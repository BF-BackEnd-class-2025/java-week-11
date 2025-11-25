# 🎮 **Controller Layer**

### Authentication · Token Validation · Role Checking · CRUD

---

# 🔐 **AuthController (Login & Registration)**

Handles:

* Register → Public
* Login → Public

```java
@RestController
@RequestMapping("/auth")
public class AuthController {

    private final AuthService authService;

    public AuthController(AuthService authService) {
        this.authService = authService;
    }

    // -----------------------------------------
    // POST /auth/register  → Public
    // -----------------------------------------
    @PostMapping("/register")
    public ResponseEntity<?> register(@Valid @RequestBody RegisterRequestDTO dto) {

        authService.register(dto); // service validates email + creates user
        return ResponseEntity.status(HttpStatus.CREATED)
                .body("User registered successfully");
    }

    // -----------------------------------------
    // POST /auth/login  → Public
    // -----------------------------------------
    @PostMapping("/login")
    public ResponseEntity<LoginResponseDTO> login(
            @Valid @RequestBody LoginRequestDTO dto
    ) {
        LoginResponseDTO response = authService.login(dto);
        return ResponseEntity.ok(response);
    }
}
```

---

# 🎬 **MovieController (Token Validation + Admin Check + CRUD)**

This version checks:

### ✔ User must provide token

### ✔ Token must be valid

### ✔ User must be ADMIN

### ✔ Logged-in user is passed to service

---

```java
@RestController
@RequestMapping("/movies")
public class MovieController 
{
    private final MovieService movieService;

    public MovieController(MovieService movieService) 
    {
        this.movieService = movieService;
    }

    // -------------------------------------------------------
    // GET /movies → PUBLIC (No authentication required)
    // -------------------------------------------------------
    @GetMapping
    public List<MovieResponseDTO> getAllMovies() 
    {
        return movieService.getAllMovies();
    }

    // -------------------------------------------------------
    // GET /movies/{id} → PUBLIC (No authentication required)
    // -------------------------------------------------------
    @GetMapping("/{id}")
    public MovieResponseDTO getMovieById(@PathVariable Long id) {
        return movieService.getMovieById(id);
    }

    // -------------------------------------------------------
    // POST /movies → ADMIN ONLY
    // Token must be valid + user must be ADMIN
    // -------------------------------------------------------
    @PostMapping
    public ResponseEntity<?> createMovie(
            @Valid @RequestBody MovieRequestDTO dto,
            @AuthenticationPrincipal User user
    ) 
    {
        // 1. Check token
        if (user == null) 
        {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body("Missing or invalid token");
        }

        // 2. Check admin role
        if (user.getRole() != Role.ADMIN) 
        {
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body("Only ADMIN users can create movies");
        }

        MovieResponseDTO created = movieService.createMovie(dto, user);
        return ResponseEntity.status(HttpStatus.CREATED).body(created);
    }

    // -------------------------------------------------------
    // PUT /movies/{id} → ADMIN ONLY
    // Token + ADMIN check
    // -------------------------------------------------------
    @PutMapping("/{id}")
    public ResponseEntity<?> updateMovie(
            @PathVariable Long id,
            @Valid @RequestBody MovieRequestDTO dto,
            @AuthenticationPrincipal User user
    ) 
    {
        if (user == null) 
        {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body("Missing or invalid token");
        }

        if (user.getRole() != Role.ADMIN)
        {
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body("Only ADMIN users can update movies");
        }

        MovieResponseDTO updated = movieService.updateMovie(id, dto);
        return ResponseEntity.ok(updated);
    }

    // -------------------------------------------------------
    // DELETE /movies/{id} → ADMIN ONLY
    // Token + ADMIN check
    // -------------------------------------------------------
    @DeleteMapping("/{id}")
    public ResponseEntity<?> deleteMovie(
            @PathVariable Long id,
            @AuthenticationPrincipal User user
    ) 
    {

        if (user == null) {
            return ResponseEntity.status(HttpStatus.UNAUTHORIZED)
                    .body("Missing or invalid token");
        }

        if (user.getRole() != Role.ADMIN) 
        {
            return ResponseEntity.status(HttpStatus.FORBIDDEN)
                    .body("Only ADMIN users can delete movies");
        }

        movieService.deleteMovie(id);
        return ResponseEntity.ok("Movie deleted successfully");
    }
}
```

---

# ✅ **WHAT THIS CONTROLLER DOES**

### ✔ If token is NOT provided → `401 Unauthorized`

```
Missing or invalid token
```

### ✔ If user is NOT admin → `403 Forbidden`

```
Only ADMIN users can create/update/delete movies
```

### ✔ If token is correct AND user is admin → CRUD works

### ✔ Logged-in user is automatically injected via:

```java
@AuthenticationPrincipal User user;
```

---


