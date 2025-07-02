---
title: "Coding convention khi code Backend"
date: 2025-07-01 21:38:00  +0700
categories: [Học tập]
tags: [Học tập]
---

# Java Spring Boot + JPA Coding Guideline

## Mục lục

1. Project Structure
2. Naming Conventions
3. Entity & Model Design
4. Repository Patterns
5. Service Layer
6. Controller Layer
7. Configuration
8. Database Design
9. Error Handling
10. Performance
11. Security Best Practices
12. API Development Guide

## Project Structure

**Recommended Folder Structure**

```
src/main/java/co/jp/adminsystem/
├── annotation/             # Định nghĩa các annotations
├── config/                 # Định nghĩa các bean và config toàn hệ thống
├── constants/              # Định nghĩa toàn bộ constant trong hệ thống
├── controller/             # Định nghĩa các controller
├── dto/                    # Data Transfer Objects
│   ├── request/            # DTO cho request đến và param cho repository
│   └── response/           # DTO cho response trả về từ API hoặc repository
├── entity/                 # JPA entities
├── enums/                  # Định nghĩa toàn bộ enum trong hệ thống
├── exception/              # Định nghĩa các exception và nơi xử lý global exception
├── repository/             # Data access layer
├── security/               # Định nghĩa về các thông tin bảo mật cho hệ thống
├── service/                # Nơi xử lý logic chính
│   ├── impl/               # Service implementations
│   └── interfaces/         # Service interfaces
├── util/                   # Các class hỗ trợ
└── Application.java        # Main application class

src/main/resources/
├── message/                # Định nghĩa i18n
│   ├── fields/             # i18n tên fields, tên properties
│   └── messages/           # i18n các messages
├── application.properties  # File properties gốc
├── application_dev.properties # File properties cho môi trường dev
├── application_prod.properties # File properties cho môi trường prod
└── logback-spring.xml      # Định nghĩa logging cho hệ thống
```

**File Naming Conventions**

```
// ✅ Good
UserController.java         // Controllers: EntityController
UserService.java            // Services: EntityService
UserRepository.java         // Repositories: EntityRepository
UserDto.java                // DTOs: EntityDto
UserNotFoundException.java  // Exceptions: DescriptiveException

// ❌ Bad
user_controller.java        // snake_case
usercontroller.java         // no separation
UserCtrl.java               // abbreviated
UserDAO.java                // old DAO naming
```

## Naming Conventions

**Classes and Interfaces**

```
// ✅ Good
public class UserService {

}

public interface PaymentProcessor {

}

public class OrderNotFoundException extends RuntimeException {

}

public enum OrderStatus {PENDING, CONFIRMED, CANCELLED}


// ❌ Bad
public class userService {

} // lowercase

public interface IPaymentProcessor {

} // Hungarian notation

public class ordernotfound {

} // no descriptive naming

public enum orderStatus {} // lowercase enum
```

**Variables and Methods**

```
// ✅ Good
private String userName;
private boolean isActiveUser;
private List<OrderItem> orderItems;

public User findUserById(Long userId) {
}

public List<User> findActiveUsers() {
}

public void updateUserStatus(Long userId, UserStatus status) {
}


// ❌ Bad
private String user_name;       // snake_case
private boolean active;         // not descriptive
private List<OrderItem> list;   // generic naming

public User getUser(Long id) {
} // 'get' implies no business logic

public List<User> users() {
} // not descriptive

public void update(Long id, String s) {
} // unclear parameters
```

**Constants**

```
// ✅ Good
public static final String DEFAULT_PAGE_SIZE = "10";
public static final int MAX_LOGIN_ATTEMPTS = 3;
public static final String USER_CACHE_NAME = "users";

// ❌ Bad
public static final String page_size = "10";
public static final int maxattempts = 3;
public static final String Cache = "users";
```

## Entity && Model Design

**DTO (Data Transfer Objects)**

```
// ✅ Good
public class UserDto {

    private Long id;
    private String email;
    private String firstName;
    private String lastName;
    private UserStatus status;
    private LocalDateTime createdAt;

    // Constructors
    public UserDto() {
    }

    public UserDto(Long id, String email, String firstName, String lastName,
                   UserStatus status, LocalDateTime createdAt) {
        this.id = id;
        this.email = email;
        this.firstName = firstName;
        this.lastName = lastName;
        this.status = status;
        this.createdAt = createdAt;
    }

    // Static factory method
    public static UserDto fromEntity(UserEntity entity) {
        return new UserDto(
                entity.getId(),
                entity.getEmail(),
                entity.getFirstName(),
                entity.getLastName(),
                entity.getStatus(),
                entity.getCreatedAt()
        );
    }

    // Getters and setters
}

public class CreateUserRequest {

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    private String email;

    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
    private String firstName;

    @NotNull(message = "Status is required")
    private UserStatus status;

// Getters and setters
}


// ❌ Bad
public class UserDto {

    public Long id;                 // Public fields
    public String email;            // No validation
    public String firstName;
    public String status;           // String instead of enum
    public Date createdAt;          // Old Date type

    // No constructors
    // No factory methods
    // No validation annotations
}

public class CreateUserRequest {

    private String email;           // No validation
    private String firstName;       // No constraints
    private String status;          // String instead of enum

    // Direct entity exposure instead of DTO
}
```

## Repository Patterns

**Spring Data JPA Repositories**

```
// ✅ Good
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {

    Optional<UserEntity> findByEmail(String email);

    List<UserEntity> findByStatusAndCreatedAtAfter(UserStatus status, LocalDateTime date);

    @Query("SELECT v FROM UserEntity v WHERE v.firstName LIKE %:name% OR v.lastName LIKE %:name%")
    List<UserEntity> findByNameContaining(@Param("name") String name);

    @Query(value = "SELECT * FROM users u WHERE u.status = :status AND u.created_at > :date",
            nativeQuery = true)
    List<UserEntity> findActiveUsersNative(@Param("status") String status,
                                          @Param("date") LocalDateTime date);

    @Modifying
    @Query("UPDATE UserEntity u SET u.status = :status WHERE u.id = :id")
    int updateUserStatus(@Param("id") Long id, @Param("status") UserStatus status);

    // Pagination support
    Page<UserEntity> findByStatus(UserStatus status, Pageable pageable);

    // Projection
    @Query("SELECT new com.company.project.dto.UserSummaryDto(u.id, u.email, u.firstName)" +
            "FROM UserEntity u WHERE u.status = :status")
    List<UserSummaryDto> findUserSummariesByStatus(@Param("status") UserStatus status);
}


// ❌ Bad
public interface UserRepository extends JpaRepository<UserEntity, Long> {

    UserEntity findByEmail(String email); // Returns null instead of Optional
    @Query("SELECT u FROM UserEntity u WHERE u.firstName LIKE '%' + :name + '%'")
    // SQL injection risk
    List<UserEntity> findByName(@Param("name") String name);

    @Query(value = "SELECT * FROM users WHERE status = '" +
            ":status" + "'", nativeQuery = true)
    // String concatenation in query
    List<UserEntity> findByStatus(@Param("status") String status);

    @Modifying
    @Query("UPDATE UserEntity u SET u.status = :status WHERE u.id = :id")
    void updateUserStatus(@Param("id") Long id,
                        @Param("status") UserStatus status); // No @Transactional

    // Using List instead of Page for large datasets
    List<UserEntity> findAll(); // Memory issues with large tables
}
```

## Service Layer

**Service Interface and implementation**

```
// ✅ Good
public interface UserService {

    UserDto createUser(CreateUserRequest request);

    UserDto getUserById(Long id);

    UserDto updateUser(Long id, UpdateUserRequest request);

    void deleteUser(Long id);

    Page<UserDto> getUsers(UserSearchCriteria criteria, Pageable pageable);

    List<UserSummaryDto> getActiveUserSummaries();
}

@Service
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;
    private final UserMapper userMapper;
    private final ApplicationEventPublisher eventPublisher;

    public UserServiceImpl(UserRepository userRepository,
                           UserMapper userMapper,
                           ApplicationEventPublisher eventPublisher) {
        this.userRepository = userRepository;
        this.userMapper = userMapper;
        this.eventPublisher = eventPublisher;
    }

    @Override
    @Transactional
    public UserDto createUser(CreateUserRequest request) {
        // Validation
        validateEmailUniqueness(request.getEmail());

        // Create entity
        UserEntity user = new UserEntity(
                request.getEmail(),
                request.getFirstName(),
                request.getStatus()
        );
          // Save
        UserEntity savedUser = userRepository.save(user);

        // Publish event
        eventPublisher.publishEvent(new UserCreatedEvent(savedUser.getId()));

        return userMapper.toDto(savedUser);
    }

    @Override
    public UserDto getUserByById(Long id) {
        UserEntity user = userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
        return userMapper.toDto(user);
    }

    private void validateEmailUniqueness(String email) {
        if (userRepository.findByEmail(email).isPresent()) {
            throw new EmailAlreadyExistsException("Email already exists: " + email);
        }
    }


// ❌ Bad
@Service
public class UserService { // No interface

    @Autowired
    private UserRepository userRepository; // Field injection

    public UserEntity createUser(UserEntity user) { // Direct entity exposure
        return userRepository.save(user); // No validation
    }

    public UserEntity getUser(Long id) {
        return userRepository.findById(id).get(); // No null checking
    }

    public List<UserEntity> getAllUsers() {
        return userRepository.findAll(); // No pagination
    }

    public void updateUser(UserEntity user) { // No validation
        userRepository.save(user); // No existence check
    }

    // No transaction management
    // No error handling
    // No event publishing
    // Direct entity return
}
```

**Transaction Management**

```
// ✅ Good
@Service
@Transactional(readOnly = true)
public class OrderService {

    @Transactional(rollbackFor = Exception.class)
    public OrderDto createOrder(CreateOrderRequest request) {
        // Multiple operations in single transaction
        UserEntity user = getUserByUserId(request.getUserId());
        validateInventory(request.getItems());
        OrderItem order = new OrderItem(user);
    orderRepository.save(order);

    // Update inventory
    updateInventory(request.getItems());

    // Send notification (this could fail)
    notificationService.sendOrderConfirmation(order.getId());

    return orderMapper.toDto(order);
    }

    @Transactional(readOnly = true, timeout = 30)
    public List<OrderDto> getOrdersForUser(Long userId) {
        return orderRepository.findByUserId(userId)
                .stream()
                .map(orderMapper::toDto)
                .collect(Collectors.toList());
    }
}

    @Transactional(propagation = Propagation.REQUIRES_NEW)
    public void logOrderEvent(Long orderId, String event) {
        // This runs in separate transaction
        orderLogRepository.save(new OrderLogEntity(orderId, event));
    }


// ❌ Bad
@Service
public class OrderService {

    public OrderDto createOrder(CreateOrderRequest request) { // No @Transactional
        UserEntity user = getUserByUserId(request.getUserId());
        OrderEntity order = new OrderEntity(user);
        orderRepository.save(order); // Each operation separate transaction
        updateInventory(request.getItems()); // Could fail after order saved
        return orderMapper.toDto(order);
    }

    @Transactional
    public List<OrderDto> getOrdersForUser(Long userId) { // Unnecessary transaction for read
        return orderRepository.findByUserId(userId)
                .stream()
                .map(orderMapper::toDto)
                .collect(Collectors.toList());
    }

    // No transaction timeout
    // No rollback configuration
    // No propagation control
}
```

## Controller Layer

**REST Controllers**

```
// ✅ Good
@RestController
@RequestMapping("/api/v1/users")
@Validated
@Slf4j
public class UserController {
    private final UserService userService;

    public UserController(UserService userService) {
    this.userService = userService;
}

@PostMapping
@ResponseStatus(HttpStatus.CREATED)
public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserRequest request) {
    log.info("Creating user with email: {}", request.getEmail());

    UserDto user = userService.createUser(request);

    URI location = ServletUriComponentsBuilder
            .fromCurrentRequest()
            .path("/{id}")
            .buildAndExpand(user.getId())
            .toUri();
    return ResponseEntity.created(location).body(user);
}

@GetMapping("/{id}")
public ResponseEntity<UserDto> getUser(@PathVariable @Positive Long id) {
    UserDto user = userService.getUserById(id);
    return ResponseEntity.ok(user);
}

@GetMapping
public ResponseEntity<Page<UserDto>> getUsers(
        @Valid UserSearchCriteria criteria,
        @PageableDefault(size = 20, sort = "createdAt", direction = Sort.Direction.DESC)
        Pageable pageable) {
    Page<UserDto> users = userService.getUsers(criteria, pageable);
    return ResponseEntity.ok(users);
}

@PutMapping("/{id}")
public ResponseEntity<UserDto> updateUser(@PathVariable @Positive Long id,
                                          @Valid @RequestBody UpdateUserRequest request) {
    UserDto updatedUser = userService.updateUser(id, request);
    return ResponseEntity.ok(updatedUser);
}

@DeleteMapping("/{id}")
@ResponseStatus(HttpStatus.NO_CONTENT)
public ResponseEntity<Void> deleteUser(@PathVariable @Positive Long id) {
    userService.deleteUser(id);
    return ResponseEntity.noContent().build();
}


// ❌ Bad
@Controller
@RequestMapping("users") // No API versioning
public class UserController {

    @Autowired
    private UserService userService; // Field injection

    @PostMapping
    public UserEntity createUser(@RequestBody UserEntity user) { // Direct entity exposure
        return userService.createUser(user); // No validation
    }
    @GetMapping("/{id}")
    public UserEntity getUser(@PathVariable String id) { // String instead of Long
        return userService.getUser(Long.valueOf(id)); // Manual conversion
    }

    @GetMapping
    public List<UserEntity> getAllUsers() { // No pagination
        return userService.getAllUsers(); // Could return huge list
    }

    @PutMapping("/{id}")
    public UserEntity updateUser(@PathVariable Long id,
                                @RequestBody UserEntity user) { // No validation
        user.setId(id); // Manual ID setting
        return userService.updateUser(user);
    }

    @DeleteMapping("/{id}")
    public String deleteUser(@PathVariable Long id) { // String response
        userService.deleteUser(id);
        return "User deleted"; // Not RESTful
    }

    // No proper HTTP status codes
    // No response entity usage
    // No input validation
    // No logging
}
```

## Configuration

**Application Properties**

```
# ✅ Good - application.yml
spring:
  application:
    name: user-service
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:local}
  datasource:
    primary:
      url: ${DB_URL:jdbc:mysql://localhost:3306/userdb}
      username: ${DB_USERNAME:root}
      password: ${DB_PASSWORD:password}
      driver-class-name: com.mysql.cj.jdbc.Driver
      hikari:
        maximum-pool-size: ${DB_MAX_POOL_SIZE:20}
        minimum-idle: ${DB_MIN_IDLE:5}
        connection-timeout: ${DB_CONNECTION_TIMEOUT:30000}
        idle-timeout: ${DB_IDLE_TIMEOUT:60000}
        max-lifetime: ${DB_MAX_LIFETIME:1800000}
  jpa:
    hibernate:
      ddl-auto: ${JPA_DDL_AUTO:validate}
      show-sql: ${JPA_SHOW_SQL:false}
    properties:
      hibernate:
        dialect: org.hibernate.dialect.MySQL8Dialect
        format_sql: true
        use_sql_comments: true
    jdbc:
     batch_size: 20
      order_inserts: true
      order_updates: true
flyway:
  enabled: ${FLYWAY_ENABLED:true}
  locations: classpath:db/migration
  baseline-on-migrate: true

logging:
  level:
    com.company.project: ${LOG_LEVEL:INFO}
    org.hibernate.sql: ${SQL_LOG_LEVEL:WARN}
    org.hibernate.type.descriptor.sql.BasicBinder: ${SQL_PARAM_LOG_LEVEL:WARN}
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"

app:
  security:
    jwt-secret: ${JWT_SECRET:mySecretKey}
    jwt-expiration-ms: ${JWT_EXPIRATION:86400000}
  database:
    max-pool-size: ${DB_MAX_POOL_SIZE:20}
    min-pool-size: ${DB_MIN_POOL_SIZE:5}


# ❌ Bad - application.properties
spring.datasource.url=jdbc:mysql://localhost:3306/mydb        # Hardcoded credentials
spring.datasource.username=root
spring.datasource.password=password
spring.jpa.hibernate.ddl-auto=create-drop                      # Dangerous for production
spring.jpa.show-sql=true                                       # Always enabled
# No environment variables
# No connection pooling configuration
# No logging configuration
# No profile-specific settings
```

## Datbase Design

**Entity Relationships**

```
// ✅ Good
@Entity
@Table(name = "users", indexes = {
    @Index(name = "idx_user_email", columnList = "email"),
    @Index(name = "idx_user_status_created", columnList = "status, created_at")
})
public class UserEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(name = "email", unique = true, nullable = false, length = 255)
    private String email;

    @OneToMany(mappedBy = "user", cascade = CascadeType.ALL, fetch = FetchType.LAZY)
    private List<OrderEntity> orders = new ArrayList<>();

    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<RoleEntity> roles = new HashSet<>();
}

@Entity
@Table(name = "orders")
public class OrderEntity {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", nullable = false, foreignKey = @ForeignKey(name = "fk_order_user"))
    private UserEntity user;

    @OneToMany(mappedBy = "order", cascade = CascadeType.ALL, orphanRemoval = true)
    @OrderBy("id ASC")
    private List<OrderItemEntity> items = new ArrayList<>();

    // Helper methods
    public void addItem(OrderItemEntity item) {
        items.add(item);
        item.setOrder(this);
    }
}


// ❌ Bad
@Entity
public class UserEntity { // No table name

    @Id
    private Long id; // No generation strategy

    private String email; // No constraints

    @OneToMany(cascade = CascadeType.ALL, fetch = FetchType.EAGER) // EAGER fetch
    private List<OrderEntity> orders; // No join column specified

    @ManyToMany(fetch = FetchType.EAGER) // EAGER fetch for collections
    private Set<RoleEntity> roles; // No join table configuration

    // No indexes
    // No foreign key names
    // No helper methods
}
```

## Error Handling

**Custom Exceptions**

```
// ✅ Good
@ResponseStatus(HttpStatus.NOT_FOUND)
public class UserNotFoundException extends RuntimeException {

    private final Long userId;

    public UserNotFoundException(String message) {
        super(message);
        this.userId = null;
    }

    public UserNotFoundException(String message, Long userId) {
        super(message);
        this.userId = userId;
    }

    public UserNotFoundException(String message, Throwable cause) {
        super(message, cause);
        this.userId = null;
    }

    public Long getUserId() {
        return userId;
    }
}

@ResponseStatus(HttpStatus.CONFLICT)
public class EmailAlreadyExistsException extends RuntimeException {

    private final String email;

    public EmailAlreadyExistsException(String message, String email) {
        super(message);
        this.email = email;
    }

    public String getEmail() {
        return email;
    }
}

public class ErrorResponse {

    private String code;
    private String message;
    private LocalDateTime timestamp;

    // Constructors, getters, setters
}

public class ValidationErrorResponse extends ErrorResponse {

    private Map<String, String> fieldErrors;

    // Constructors, getters, setters
}


// ❌ Bad
public class UserException extends Exception { // Checked exception
    // No specific information
    // Generic naming
}

public class CustomException extends RuntimeException {
    // No response status
    // No additional context
    // No structured error response
}
```

**Service Layer Eror Handling**

```
// ✅ Good
@Service
@Transactional(readOnly = true)
public class UserServiceImpl implements UserService {

    @Override
    public UserDto getUserByById(Long id) {
        if (id == null || id <= 0) {
            throw new IllegalArgumentException("User ID must be positive");
        }

        return userRepository.findById(id)
                             .map(userMapper::toDto)
                             .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
    }

    @Override
    @Transactional
    public UserDto createUser(CreateUserRequest request) {
        try {
            validateEmailUniqueness(request.getEmail());

            UserEntity user = userMapper.toEntity(request);
            UserEntity savedUser = userRepository.save(user);

            return userMapper.toDto(savedUser);
        } catch (DataIntegrityViolationException ex) {
            if (ex.getMessage().contains("email")) {
                throw new EmailAlreadyExistsException("Email already exists: " + request.getEmail(),
                                                     request.getEmail());
            }
            throw new UserCreationException("Failed to create user", ex);
        }
    }

    private void validateEmailUniqueness(String email) {
        userRepository.findByEmail(email)
                      .ifPresent(user -> {
                          throw new EmailAlreadyExistsException("Email already exists: " + email, email);
                      });
    }
}


// ❌ Bad
@Service
public class UserService {

    public UserDto getUser(Long id) {
        UserEntity user = userRepository.findById(id).get(); // Can throw NoSuchElementException
        return toDto(user);
    }

    public UserDto createUser(CreateUserRequest request) {
        UserEntity user = toEntity(request);
        UserEntity saved = userRepository.save(user); // No exception handling
        return toDto(saved);
    }

    // No input validation
    // No specific exception handling
    // Using .get() without checking Optional
}
```

## Performance

**Query Optimization**

```
// ✅ Good
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {

    // Use projections for limited data
    @Query("SELECT new com.company.project.dto.UserSummaryDto(u.id, u.email, u.firstName) " +
           "FROM UserEntity u WHERE u.status = :status")
    List<UserSummaryDto> findUserSummariesByStatus(@Param("status") UserStatus status);

    // Fetch joins to avoid N+1 problem
    @Query("SELECT DISTINCT u FROM UserEntity u LEFT JOIN FETCH u.orders WHERE u.id = :id")
    UserEntity findUserWithOrdersById(@Param("id") Long id);

    // Pagination for large datasets
    @Query("SELECT u FROM UserEntity u WHERE u.createdAt BETWEEN :startDate AND :endDate")
    Page<UserEntity> findByDateRange(@Param("startDate") LocalDateTime startDate,
                                     @Param("endDate") LocalDateTime endDate,
                                     Pageable pageable);

    // Batch operations
    @Modifying
    @Query("UPDATE UserEntity u SET u.status = :status WHERE u.id IN :ids")
    int updateStatusForUsers(@Param("ids") List<Long> ids,
                             @Param("status") UserStatus status);
}

@Service
public class UserService {

    // Caching
    @Cacheable(value = "users", key = "#id")
    public UserDto getUserById(Long id) {
        return userRepository.findById(id)
            .map(userMapper::toDto)
            .orElseThrow(() -> new UserNotFoundException("User not found: " + id));
    }

    @CacheEvict(value = "users", key = "#id")
    public void deleteUser(Long id) {
        userRepository.deleteById(id);
    }

    // Bulk operations
    @Transactional
    public void updateUserStatuses(List<Long> userIds, UserStatus status) {
        int batchSize = 1000;
        for (int i = 0; i < userIds.size(); i += batchSize) {
            List<Long> batch = userIds.subList(i, Math.min(i + batchSize, userIds.size()));
            userRepository.updateStatusForUsers(batch, status);
        }
    }
}

// ❌ Bad
@Repository
public interface UserRepository extends JpaRepository<UserEntity, Long> {

    // No projections - fetches all fields
    List<UserEntity> findByStatus(UserStatus status);

    // No fetch joins - causes N+1 problem
    List<UserEntity> findAllByOrdersIsNotEmpty();

    // No pagination - memory issues
    List<UserEntity> findAll();
}

@Service
public class UserService {

    // No caching
    public UserDto getUser(Long id) {
        return userRepository.findById(id)
            .map(this::toDto)
            .orElse(null);
    }

    // Individual operations instead of batch
    public void updateStatuses(List<Long> userIds, UserStatus status) {
        for (Long id : userIds) {
            UserEntity user = userRepository.findById(id).get();
            user.setStatus(status);
            userRepository.save(user);  // N+1 database calls
        }
        // No transaction management for bulk operations
    }
}

```

## Security Practices

**Input Validation & Sanitization**

```
// ✅ Good
public class CreateUserRequest {

    @NotBlank(message = "Email is required")
    @Email(message = "Invalid email format")
    @Size(max = 255, message = "Email must not exceed 255 characters")
    private String email;

    @NotBlank(message = "First name is required")
    @Size(min = 2, max = 50, message = "First name must be between 2 and 50 characters")
    @Pattern(regexp = "^[a-zA-Z\\s]*$", message = "First name can only contain letters and spaces")
    private String firstName;

    @NotNull(message = "Birth date is required")
    @Past(message = "Birth date must be in the past")
    private LocalDate birthDate;

    @NotNull(message = "Status is required")
    private UserStatus status;

    // Getters and setters
}

@Component
public class InputSanitizer {

    public String sanitizeHtml(String input) {
        if (input == null)
            return null;

        return input.replaceAll("<script.*?</script>", "")
                    .replaceAll("<.*?>", "")
                    .trim();
    }

    public String sanitizeForDatabase(String input) {
        if (input == null)
            return null;

        return input.replace("'", "''")
                    .replace("\"", "\\\"")
                    .trim();
    }
}

@RestController
@RequestMapping("/users")
public class UserController {

    private final InputSanitizer inputSanitizer;

    public UserController(InputSanitizer inputSanitizer) {
        this.inputSanitizer = inputSanitizer;
    }

    @PostMapping
    public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserRequest request) {
        // Validation + Sanitization logic
        // Mapping to entity, saving to DB, etc.
        return ResponseEntity.ok(new UserDto());
    }
}

@PostMapping("/users")
public ResponseEntity<UserDto> createUser(@Valid @RequestBody CreateUserRequest request) {

    // Sanitize inputs
    request.setFirstName(inputSanitizer.sanitizeHtml(request.getFirstName()));
    request.setLastName(inputSanitizer.sanitizeHtml(request.getLastName()));

    UserDto user = userService.createUser(request);
    return ResponseEntity.status(HttpStatus.CREATED).body(user);
}

// ❌ Bad
public class CreateUserRequest {

    private String email;       // No validation
    private String firstName;   // No size limits
    private String birthDate;   // String instead of LocalDate
    private String status;      // String instead of enum

    // No validation annotations
    // No sanitization
}

@RestController
public class UserController {

    @PostMapping("/users")
    public UserEntity createUser(@RequestBody CreateUserRequest request) { // No @Valid
        // No input sanitization
        // Direct processing of potentially malicious input
        return userService.createUser(request);
    }
}
```

## Best Practices for API development

✅ DO's:

- RESTful design: Proper HTTP methods and status code
- Input Validation: Bean validation with meaningful messages
- DTO pattern: Separate entities from API contracts
- Error handling: Global exception handler with structured responses
- Security: Authentication, authorization, input sanitization
- Documentation: OpenAPI/Swagger documentation
- Logging: Comprehensive loggin with correlation IDs
- Testing: Unit and integration tests

❌ DONT'Ts:

- Entity exposure: Returning entities directly from controllers
- Missing validation: Not validating input data
- Poor error handling: Using generic error responses
- No pagination: Returning large lists without pagination
- Hardcoded values: Using magic numbers or hardcoded strings
- No logging: Missing audit trails or application logs
- Security gaps: No authentication or authorization implemented
  > Following these guidelines will help make your Java Spring Boot applications maintainable, scalable, and production-ready.

## api-developlemt-guide

1. Tạo sẵn các class cho request/response nếu cần

- Request thì tạo trong folder dto/request/{PACKAGE_NAME} {PACKAGE_NAME đặt theo tên của chức năng liên quan}
- Response thì tạo trong folder dto/response/{PACKAGE_NAME} {PACKAGE_NAME đặt theo tên của chức năng liên quan}
- Dùng get/set mặc định của lombok, nếu cần get/set thì tạo thêm

2. Thực hiện validate parameter bằng annotation, có thể là annotation của SpringBoot hoặc các annotation trong folder annotation

- Validate có thể sẽ cần các regex hoặc constant, lấy trong class constants/AppConst.java và util/DateUtils ( nếu validate date time)

3. Tạo class controller trong folder controller
4. Tạo interface cho service trong folder service, sau đó tạo implement trong folder service/imp
5. Tạo repository dùng JpaRepository trong folder repository
6. Trong quá trình coding thì phải làm theo rule ghi log và comment từng phần code theo tài liệu API design
7. Sau khi làm xong coding thì viết Unit Test cho service và controller của mình: yêu cầu coverage 100%
