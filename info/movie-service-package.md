Absolutely Sam — here is the **updated Service Layer README**, fully aligned with your correct project rules:

## ✔ USER can CRUD **only their own movies**

## ✔ ADMIN can CRUD **ALL movies**

This version fixes the incorrect parts in the MovieService you pasted earlier.

---

# ⚙️ **Service Layer**

## Business Logic · Authentication · Movie Ownership · Admin Override

The **service layer** handles all business rules:

* Validating login & registration
* Hashing passwords
* Issuing JWT tokens
* Ensuring users can only modify their own movies
* Allowing admins to manage ALL movies

This layer is the “brain” of the application—controllers only pass data here.

---

# 🔐 **AuthService — Login · Register · Token Generation**

Handles:

* Registering users
* Hashing passwords
* Logging in
* Validating credentials
* Issuing JWT tokens that include:

```
userId
role (USER or ADMIN)
email (subject)
```

### ✅ Final AuthService (Correct + Clean + Commented)

```java
@Service
public class AuthService
{
    private final UserRepository userRepository;
    private final JwtUtils jwtUtils;
    private final BCryptPasswordEncoder passwordEncoder;

    public AuthService(UserRepository userRepository, JwtUtils jwtUtils, BCryptPasswordEncoder passwordEncoder)
    {
        this.userRepository = userRepository;
        this.jwtUtils = jwtUtils;
        this.passwordEncoder = passwordEncoder;
    }

    // -----------------------------------------------
    // REGISTER NEW USER
    // -----------------------------------------------
    public void register(RegisterRequestDTO dto)
    {
        // Email must be unique
        if (userRepository.existsByEmail(dto.email))
        {
            throw new EmailAlreadyExistsException(dto.email);
        }

        // Build new user
        User user = new User();
        user.setUsername(dto.username);
        user.setEmail(dto.email);
        user.setHashedPassword(passwordEncoder.encode(dto.password));

        // Default role = USER
        user.setRole(Role.USER);

        userRepository.save(user);
    }

    // -----------------------------------------------
    // LOGIN + RETURN JWT TOKEN
    // -----------------------------------------------
    public LoginResponseDTO login(LoginRequestDTO dto)
    {
        User user = userRepository.findByEmail(dto.email);

        if (user == null)
        {
            throw new UserNotFoundException(dto.email);
        }

        // Verify password using BCrypt
        if (!passwordEncoder.matches(dto.password, user.getHashedPassword()))
        {
            throw new IllegalArgumentException("Invalid email or password");
        }

        // Create signed token
        String token = jwtUtils.generateToken(
                user.getEmail(),
                Map.of(
                        "userId", user.getId(),
                        "role", user.getRole().name()
                )
        );

        return new LoginResponseDTO(token);
    }
}
```

---

# 🎬 **MovieService — USER Ownership + ADMIN Full Access**

This is the MOST important layer for your project.

## ✔ Public:

* `getAllMovies()`
* `getMovieById()`

## ✔ USER:

* Create **his own** movies
* Update **his own** movies
* Delete **his own** movies

## ✔ ADMIN:

* Can update/delete **ANY** movie in the system

## ✔ Ownership Logic (core of the project)

```java
if (user.getRole() == Role.USER && !movie.getOwner().getId().equals(user.getId())) {
    throw new ForbiddenActionException("You do not own this movie");
}
```

---

# ✅ Final MovieService (Perfect Version)

```java
@Service
public class MovieService
{
    private final MovieRepository movieRepository;
    private final MovieMapper movieMapper;

    public MovieService(MovieRepository movieRepository, MovieMapper movieMapper)
    {
        this.movieRepository = movieRepository;
        this.movieMapper = movieMapper;
    }

    // -------------------------------------------------------
    // PUBLIC: Anyone can view all movies
    // -------------------------------------------------------
    public List<MovieResponseDTO> getAllMovies()
    {
        return movieRepository.findAll()
                .stream()
                .map(movieMapper::toResponseDto)
                .toList();
    }

    // -------------------------------------------------------
    // PUBLIC: Anyone can view a single movie
    // -------------------------------------------------------
    public MovieResponseDTO getMovieById(Long movieId)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        return movieMapper.toResponseDto(movie);
    }

    // -------------------------------------------------------
    // USER/Admin: create movie
    // USER creates movie for *himself*
    // ADMIN creates movie for *anyone* (but usually himself)
    // -------------------------------------------------------
    public MovieResponseDTO createMovie(MovieRequestDTO dto, User user)
    {
        Movie movie = movieMapper.toEntity(dto);

        // Set owner = authenticated user
        movie.setOwner(user);

        Movie saved = movieRepository.save(movie);
        return movieMapper.toResponseDto(saved);
    }

    // -------------------------------------------------------
    // USER/Admin: update movie
    // USER: only if owns movie
    // ADMIN: unrestricted
    // -------------------------------------------------------
    public MovieResponseDTO updateMovie(Long movieId, MovieRequestDTO dto, User user)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        // USER cannot edit someone else's movie
        if (user.getRole() == Role.USER &&
                !movie.getOwner().getId().equals(user.getId()))
        {
            throw new ForbiddenActionException("You do not own this movie");
        }

        // Apply edits
        movie.setTitle(dto.title);
        movie.setDirector(dto.director);
        movie.setYear(dto.year);
        movie.setGenre(dto.genre);

        Movie updated = movieRepository.save(movie);
        return movieMapper.toResponseDto(updated);
    }

    // -------------------------------------------------------
    // USER/Admin: delete movie
    // USER: only if owns movie
    // ADMIN: unrestricted
    // -------------------------------------------------------
    public void deleteMovie(Long movieId, User user)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        if (user.getRole() == Role.USER &&
                !movie.getOwner().getId().equals(user.getId()))
        {
            throw new ForbiddenActionException("You do not own this movie");
        }

        movieRepository.delete(movie);
    }
}
```

---


