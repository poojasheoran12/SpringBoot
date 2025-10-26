# Spring Boot Complete Guide

---

## 1. Introduction to Spring Boot

**Spring Boot** is a framework built on top of the Spring Framework to simplify building production-ready applications. It eliminates boilerplate configuration and integrates an embedded server for quick execution.

**Key Points:**

* **Auto-configuration:** Detects dependencies in your project and configures them automatically.
* **Embedded Server:** Runs with Tomcat, Jetty, or Undertow out-of-the-box.
* **Starter Dependencies:** Simplifies dependency management, e.g., `spring-boot-starter-web`, `spring-boot-starter-data-jpa`.
* **Convention over Configuration:** You follow standard conventions, reducing XML or manual setup.

**Example:**

```java
@SpringBootApplication
public class MyApp {
    public static void main(String[] args) {
        SpringApplication.run(MyApp.class, args);
    }
}
```

---

## 2. Core Concepts

### 2.1 Beans & IoC Container

* **Bean:** An object managed by Spring.
* **IoC (Inversion of Control):** Spring controls object creation, not you.
* **Dependency Injection (DI):** Spring injects required beans automatically.

```java
@Service
public class UserService {
    @Autowired
    private UserRepository repo;
}
```

### 2.2 Component Scanning

* Spring automatically scans packages for annotated classes: `@Component`, `@Service`, `@Repository`, `@Controller`.
* Managed beans are stored in **ApplicationContext**.

### 2.3 Configuration

* `application.properties` or `application.yml` for environment-specific configuration.
* Profiles allow multiple environments (`dev`, `prod`).

---

## 3. Spring Boot Architecture

**Flow of a Request:**

```
Client (Browser/App) -> Embedded Server (Tomcat) -> DispatcherServlet -> Controller -> Service -> Repository -> Database
```

* **Tomcat:** Embedded server that listens on port 8080 and forwards requests.
* **DispatcherServlet:** Routes HTTP requests to the correct controller.
* **Controller:** Handles request, calls service.
* **Service:** Contains business logic.
* **Repository:** Handles database operations.

---

## 4. Annotations in Spring Boot

| Annotation               | Purpose                                                                 |
| ------------------------ | ----------------------------------------------------------------------- |
| `@SpringBootApplication` | Combines `@Configuration`, `@EnableAutoConfiguration`, `@ComponentScan` |
| `@Component`             | Marks a class as a Spring-managed bean                                  |
| `@Service`               | Business logic layer                                                    |
| `@Repository`            | Data access layer                                                       |
| `@Controller`            | Handles web requests                                                    |
| `@RestController`        | REST endpoints                                                          |
| `@Autowired`             | Dependency injection                                                    |
| `@Bean`                  | Explicitly declares a bean                                              |
| `@Value`                 | Inject values from properties                                           |

---

## 5. Spring Boot Auto Configuration

* Detects project dependencies and configures them automatically.
* Controlled via `@EnableAutoConfiguration` and `spring.factories` metadata.
* Reduces manual setup for MVC, JPA, Security, etc.

---

## 6. Spring Boot Starter Dependencies

| Starter                        | Purpose                         |
| ------------------------------ | ------------------------------- |
| `spring-boot-starter-web`      | REST APIs, MVC, embedded Tomcat |
| `spring-boot-starter-data-jpa` | JPA + Hibernate support         |
| `spring-boot-starter-security` | Spring Security for auth        |
| `spring-boot-starter-actuator` | Monitoring endpoints            |
| `spring-boot-starter-test`     | Testing libraries               |

---

## 7. Spring Boot with Databases

**JPA + Hibernate Setup:**

```java
@Entity
public class User {
    @Id @GeneratedValue
    private Long id;
    private String username;
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {}

@Service
public class UserService {
    @Autowired
    private UserRepository repo;
    public List<User> getAllUsers() { return repo.findAll(); }
}

@RestController
@RequestMapping("/users")
public class UserController {
    @Autowired
    private UserService service;
    @GetMapping
    public List<User> getUsers() { return service.getAllUsers(); }
}
```

---

## 8. Authentication & Authorization

### 8.1 Basic JWT Flow

1. User logs in with username/password.
2. Backend validates credentials.
3. Backend generates **JWT token**.
4. Token sent to client.
5. Client includes token in `Authorization: Bearer <token>` header for protected routes.

### 8.2 Spring Security Setup

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .authorizeHttpRequests()
            .requestMatchers("/auth/**").permitAll()
            .anyRequest().authenticated();
        return http.build();
    }
}
```

### 8.3 JWT Token Example

```java
@Service
public class JwtService {
    @Value("${jwt.secret}")
    private String secret;

    public String generateToken(UserDetails user) { ... }
    public Boolean validateToken(String token, UserDetails user) { ... }
}
```

* **UserDetailsService** loads user data from DB.
* **AuthenticationManager** authenticates login.
* **JWT Filter** intercepts requests to validate token.

---

## 9. Spring Boot Actuator & Monitoring

* Adds `/actuator` endpoints.
* `/health` → App health
* `/info` → App info
* `/metrics` → Performance metrics

---

## 10. Exception Handling

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<String> handleNotFound(ResourceNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(ex.getMessage());
    }
}
```

---

## 11. Profiles & Environments

* Use `application-dev.yml` and `application-prod.yml`.
* Activate profile with `--spring.profiles.active=dev`
* Spring loads the corresponding configuration.

---

## 12. DevTools & Hot Reloading

* Automatic restart on code change.
* LiveReload integration for frontend refresh.
* Dependency: `spring-boot-devtools`.

---

## 13. External API Calls

* **RestTemplate (blocking)**

```java
RestTemplate restTemplate = new RestTemplate();
String response = restTemplate.getForObject(url, String.class);
```

* **WebClient (non-blocking)**

```java
WebClient client = WebClient.create();
Mono<String> response = client.get().uri(url).retrieve().bodyToMono(String.class);
```

---

## 14. Deployment

* Package as **JAR**: `mvn clean package`
* Run: `java -jar target/myapp.jar`
* Embedded Tomcat runs automatically.
* Can also deploy as WAR if needed.

---

## 15. Testing

* Unit tests: `@SpringBootTest`, `@WebMvcTest`.
* Mocking: Mockito.
* Example:

```java
@SpringBootTest
class UserServiceTest {
    @Autowired UserService service;

    @Test
    void testGetAllUsers() {
        List<User> users = service.getAllUsers();
        assertNotNull(users);
    }
}
```

---

## 16. Complete Request Flow Summary

```
Client (Browser/App)
  ↓
Embedded Server (Tomcat)
  ↓
DispatcherServlet
  ↓
Controller (@RestController)
  ↓
Service Layer (@Service)
  ↓
Repository (@Repository)
  ↓
Database (MySQL/Postgres)
  ↑
Response sent back up the chain
```

---

## 17. Best Practices

* Keep layers separated (Controller → Service → Repository).
* Use DTOs for request/response.
* Always validate inputs.
* Secure endpoints using Spring Security.
* Use profiles for environment configs.
* Write unit and integration tests.

---

**File Complete: `springboot-complete-guide.md`**

This file now contains a **detailed revision-ready guide** covering Spring Boot architecture, core concepts, beans, DI, Tomcat, database integration, authentication (JWT), actuator, exception handling, testing, deployment, and best practices — with **example code snippets** for each section.
