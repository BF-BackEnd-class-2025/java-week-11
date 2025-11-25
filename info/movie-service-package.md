# ⚙️ Service Layer

## Business Logic · Authentication · Movie CRUD · Role Support

The **service layer** contains the core business logic of the application.
Controllers receive requests → Services process data → Repositories interact with the database.

In this project, the main service classes are:

* **AuthService** → handles registration, login, password encoding, and JWT issuing
* **MovieService** → handles creation, retrieval, update, deletion of movies

Each service is annotated with `@Service` so Spring Boot can manage it automatically.

---

# 🔐 AuthService — Authentication & JWT Token Logic

Handles:

* Registering new users
* Hashing passwords
* Logging in users
* Validating credentials
* Creating JWT tokens with user role

Here is the improved version with **role support** and **inline comments**:

```java
package com.example.demo.service;

import com.example.demo.dto.LoginRequestDTO;
import com.example.demo.dto.LoginResponseDTO;
import com.example.demo.dto.RegisterRequestDTO;
import com.example.demo.exception.EmailAlreadyExistsException;
import com.example.demo.exception.UserNotFoundException;
import com.example.demo.model.Role;
import com.example.demo.model.User;
import com.example.demo.repository.UserRepository;
import com.example.demo.security.JwtUtils;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.stereotype.Service;

import java.util.Map;

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

    public void register(RegisterRequestDTO dto)
    {
        // Check for duplicate email before registering
        if (userRepository.existsByEmail(dto.email)) 
        {
            throw new EmailAlreadyExistsException(dto.email);
        }

        // Create a new User entity
        User user = new User();
        user.setUsername(dto.username);
        user.setEmail(dto.email);

        // Encrypt password before saving
        user.setHashedPassword(passwordEncoder.encode(dto.password));

        // Default role = USER
        user.setRole(Role.USER);

        userRepository.save(user);
    }

    public LoginResponseDTO login(LoginRequestDTO dto)
    {
        // Find user by email
        User user = userRepository.findByEmail(dto.email);
        if (user == null) 
        {
            throw new UserNotFoundException(dto.email);
        }

        // Check password using BCrypt
        if (!passwordEncoder.matches(dto.password, user.getHashedPassword())) 
        {
            throw new IllegalArgumentException("Invalid email or password");
        }

        // Create JWT token containing the user ID + role
        String token = jwtUtils.generateToken(
                user.getEmail(),
                Map.of(
                        "userId", user.getId(),
                        "role", user.getRole().name()
                )
        );

        // Build login response
        LoginResponseDTO response = new LoginResponseDTO();
        response.token = token;
        return response;
    }
}
```

---

# 🎬 MovieService — Movie CRUD Logic (Admin Only)

Handles:

* Listing all movies (public)
* Getting a movie by ID
* Creating a movie (ADMIN only)
* Updating a movie (ADMIN only)
* Deleting a movie (ADMIN only)

```java
package com.example.demo.service;

import com.example.demo.dto.MovieRequestDTO;
import com.example.demo.dto.MovieResponseDTO;
import com.example.demo.exception.MovieNotFoundException;
import com.example.demo.mapper.MovieMapper;
import com.example.demo.model.Movie;
import com.example.demo.repository.MovieRepository;
import org.springframework.stereotype.Service;

import java.util.List;

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

    // PUBLIC: Anyone can view movies
    public List<MovieResponseDTO> getAllMovies()
    {
        List<Movie> movies = movieRepository.findAll();

        // Convert entity → DTO for returning to client
        return movies.stream()
                .map(movieMapper::toResponseDto)
                .toList();
    }

    // PUBLIC: View single movie
    public MovieResponseDTO getMovieById(Long movieId)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        return movieMapper.toResponseDto(movie);
    }

    // ADMIN ONLY: Create a new movie
    public MovieResponseDTO createMovie(MovieRequestDTO dto)
    {
        Movie movie = movieMapper.toEntity(dto);

        Movie saved = movieRepository.save(movie);

        return movieMapper.toResponseDto(saved);
    }

    // ADMIN ONLY: Update a movie
    public MovieResponseDTO updateMovie(Long movieId, MovieRequestDTO dto)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        // Apply updates
        movie.setTitle(dto.title);
        movie.setDirector(dto.director);
        movie.setYear(dto.year);
        movie.setGenre(dto.genre);

        Movie updated = movieRepository.save(movie);

        return movieMapper.toResponseDto(updated);
    }

    // ADMIN ONLY: Delete a movie
    public void deleteMovie(Long movieId)
    {
        Movie movie = movieRepository.findById(movieId)
                .orElseThrow(() -> new MovieNotFoundException(movieId));

        movieRepository.delete(movie);
    }
}
```

---

# 🎯 Summary

Your service layer now includes:

| Service               | Responsibility                                                   |
|-----------------------|------------------------------------------------------------------|
| **AuthService**       | Register users, hash passwords, login, create JWT                |
| **MovieService**      | Handle all movie CRUD operations                                 |
| **Role Support**      | Auth user gets USER role; ADMIN can modify movies                |
| **Exception Support** | Uses custom exceptions: MovieNotFound, UserNotFound, EmailExists |

Both services are clean, readable, and production-ready.

---

