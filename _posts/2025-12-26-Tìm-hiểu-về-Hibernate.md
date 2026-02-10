---
title: "Tìm hiểu về Hibernate"
date: 2025-12-26 01:17:00  +0700
categories: [hibernate, jpa]
tags: [hibernate, jpa]
---


### Hibernate là gì?

Hibernate là một framework ORM (Object-Relational Mapping) mã nguồn mở cho Java, giúp ánh xạ các đối tượng Java với các bảng trong cơ sở dữ liệu quan hệ. Nó là một implementation của JPA (Java Persistence API) specification và được phát triển bởi Red Hat.

### Vấn đề mà Hibernate giải quyết

Trước khi có Hibernate, developers phải đối mặt với nhiều thách thức:

**Impedance Mismatch (Sự không tương thích giữa OOP và Relational Database)**

Mô hình lập trình hướng đối tượng và mô hình cơ sở dữ liệu quan hệ có những khác biệt cơ bản. Ví dụ:
- OOP có kế thừa, đa hình, còn RDBMS thì không
- OOP dùng references, RDBMS dùng foreign keys
- OOP có collections, RDBMS có tables với joins

**Boilerplate Code với JDBC**

Khi sử dụng JDBC thuần, bạn phải viết rất nhiều code lặp đi lặp lại:

```java
// Code JDBC truyền thống - quá nhiều boilerplate!
Connection conn = null;
PreparedStatement stmt = null;
ResultSet rs = null;

try {
    conn = DriverManager.getConnection(url, user, password);
    stmt = conn.prepareStatement("SELECT * FROM users WHERE id = ?");
    stmt.setInt(1, userId);
    rs = stmt.executeQuery();
    
    if (rs.next()) {
        User user = new User();
        user.setId(rs.getInt("id"));
        user.setName(rs.getString("name"));
        user.setEmail(rs.getString("email"));
        // ... nhiều fields khác
        return user;
    }
} catch (SQLException e) {
    e.printStackTrace();
} finally {
    if (rs != null) rs.close();
    if (stmt != null) stmt.close();
    if (conn != null) conn.close();
}
```

**Với Hibernate, code trở nên đơn giản:**

```java
// Hibernate - clean và ngắn gọn!
Session session = sessionFactory.openSession();
User user = session.get(User.class, userId);
session.close();
```

**Các vấn đề khác Hibernate giải quyết:**
- Quản lý connection pool tự động
- Caching để tối ưu performance
- Lazy loading để tránh load dữ liệu không cần thiết
- Transaction management
- Database independence - dễ dàng chuyển đổi giữa các RDBMS

## 2. Các thành phần chính của Hibernate

### 2.1. SessionFactory

SessionFactory là một factory để tạo ra các Session objects. Đây là một thread-safe object và nên được tạo một lần duy nhất cho mỗi database.

```java
Configuration configuration = new Configuration().configure();
SessionFactory sessionFactory = configuration.buildSessionFactory();
```

**Đặc điểm:**
- Heavyweight object, tốn tài nguyên để khởi tạo
- Immutable và thread-safe
- Chứa metadata về mapping
- Chứa second-level cache

### 2.2. Session

Session là interface chính để tương tác với database. Nó đại diện cho một connection đến database và là một short-lived object.

```java
Session session = sessionFactory.openSession();
try {
    // Thực hiện các operations
    User user = session.get(User.class, 1);
} finally {
    session.close();
}
```

**Đặc điểm:**
- Lightweight và short-lived
- NOT thread-safe
- Đại diện cho một unit of work
- Chứa first-level cache (mặc định)

### 2.3. Transaction

Transaction interface cho phép quản lý các giao dịch database.

```java
Session session = sessionFactory.openSession();
Transaction tx = null;

try {
    tx = session.beginTransaction();
    
    User user = new User("John Doe", "john@email.com");
    session.save(user);
    
    tx.commit();
} catch (Exception e) {
    if (tx != null) tx.rollback();
    e.printStackTrace();
} finally {
    session.close();
}
```

### 2.4. Query và Criteria

Hibernate cung cấp nhiều cách để query dữ liệu:

```java
// HQL (Hibernate Query Language)
Query<User> query = session.createQuery("FROM User WHERE name = :name", User.class);
query.setParameter("name", "John");
List<User> users = query.list();

// Criteria API
CriteriaBuilder cb = session.getCriteriaBuilder();
CriteriaQuery<User> cr = cb.createQuery(User.class);
Root<User> root = cr.from(User.class);
cr.select(root).where(cb.equal(root.get("name"), "John"));

List<User> users = session.createQuery(cr).getResultList();
```

### 2.5. Configuration

Configuration object được sử dụng để configure và bootstrap Hibernate.

```java
Configuration config = new Configuration();
config.configure("hibernate.cfg.xml");
config.addAnnotatedClass(User.class);
SessionFactory sessionFactory = config.buildSessionFactory();
```

## 3. Ứng dụng Hibernate trong Java

### 3.1. Định nghĩa Entity

Entity là một POJO (Plain Old Java Object) được map với một table trong database.

```java
import javax.persistence.*;

@Entity
@Table(name = "users")
public class User {
    
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    @Column(name = "name", nullable = false, length = 100)
    private String name;
    
    @Column(name = "email", unique = true, nullable = false)
    private String email;
    
    @Column(name = "age")
    private Integer age;
    
    // Constructors
    public User() {}
    
    public User(String name, String email) {
        this.name = name;
        this.email = email;
    }
    
    // Getters and Setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    
    public String getName() { return name; }
    public void setName(String name) { this.name = name; }
    
    public String getEmail() { return email; }
    public void setEmail(String email) { this.email = email; }
    
    public Integer getAge() { return age; }
    public void setAge(Integer age) { this.age = age; }
}
```

### 3.2. CRUD Operations

**Create (Insert)**

```java
public void createUser(String name, String email) {
    Session session = sessionFactory.openSession();
    Transaction tx = null;
    
    try {
        tx = session.beginTransaction();
        
        User user = new User(name, email);
        session.save(user);
        
        tx.commit();
        System.out.println("User created with ID: " + user.getId());
    } catch (Exception e) {
        if (tx != null) tx.rollback();
        e.printStackTrace();
    } finally {
        session.close();
    }
}
```

**Read (Select)**

```java
public User getUser(Long id) {
    Session session = sessionFactory.openSession();
    User user = session.get(User.class, id);
    session.close();
    return user;
}

public List<User> getAllUsers() {
    Session session = sessionFactory.openSession();
    List<User> users = session.createQuery("FROM User", User.class).list();
    session.close();
    return users;
}
```

**Update**

```java
public void updateUser(Long id, String newEmail) {
    Session session = sessionFactory.openSession();
    Transaction tx = null;
    
    try {
        tx = session.beginTransaction();
        
        User user = session.get(User.class, id);
        if (user != null) {
            user.setEmail(newEmail);
            session.update(user);
        }
        
        tx.commit();
    } catch (Exception e) {
        if (tx != null) tx.rollback();
        e.printStackTrace();
    } finally {
        session.close();
    }
}
```

**Delete**

```java
public void deleteUser(Long id) {
    Session session = sessionFactory.openSession();
    Transaction tx = null;
    
    try {
        tx = session.beginTransaction();
        
        User user = session.get(User.class, id);
        if (user != null) {
            session.delete(user);
        }
        
        tx.commit();
    } catch (Exception e) {
        if (tx != null) tx.rollback();
        e.printStackTrace();
    } finally {
        session.close();
    }
}
```

### 3.3. Relationships trong Hibernate

**One-to-Many Relationship**

```java
@Entity
@Table(name = "departments")
public class Department {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @OneToMany(mappedBy = "department", cascade = CascadeType.ALL)
    private List<Employee> employees = new ArrayList<>();
    
    // Getters, setters, constructors
}

@Entity
@Table(name = "employees")
public class Employee {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToOne
    @JoinColumn(name = "department_id")
    private Department department;
    
    // Getters, setters, constructors
}
```

**Many-to-Many Relationship**

```java
@Entity
@Table(name = "students")
public class Student {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String name;
    
    @ManyToMany(cascade = CascadeType.ALL)
    @JoinTable(
        name = "student_course",
        joinColumns = @JoinColumn(name = "student_id"),
        inverseJoinColumns = @JoinColumn(name = "course_id")
    )
    private List<Course> courses = new ArrayList<>();
    
    // Getters, setters, constructors
}

@Entity
@Table(name = "courses")
public class Course {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;
    
    private String title;
    
    @ManyToMany(mappedBy = "courses")
    private List<Student> students = new ArrayList<>();
    
    // Getters, setters, constructors
}
```

## 4. Object-Relational Mapping (ORM)

### ORM là gì?

ORM là một kỹ thuật lập trình cho phép chuyển đổi dữ liệu giữa hệ thống kiểu không tương thích bằng cách sử dụng ngôn ngữ lập trình hướng đối tượng. Nó tạo ra một "virtual object database" có thể được sử dụng từ trong ngôn ngữ lập trình.

### Cách Hibernate thực hiện ORM

**Entity Mapping**

Hibernate sử dụng annotations hoặc XML để map Java classes với database tables:

| Java | Database |
|------|----------|
| Class | Table |
| Object | Row |
| Field | Column |

**Ví dụ chi tiết về mapping:**

```java
@Entity  // Đánh dấu class này là một entity
@Table(name = "products")  // Map với table "products"
public class Product {
    
    @Id  // Primary key
    @GeneratedValue(strategy = GenerationType.IDENTITY)  // Auto-increment
    @Column(name = "product_id")
    private Long id;
    
    @Column(name = "product_name", length = 200, nullable = false)
    private String name;
    
    @Column(name = "price", precision = 10, scale = 2)
    private BigDecimal price;
    
    @Temporal(TemporalType.TIMESTAMP)  // Map với TIMESTAMP
    @Column(name = "created_date")
    private Date createdDate;
    
    @Enumerated(EnumType.STRING)  // Lưu enum dưới dạng string
    @Column(name = "status")
    private ProductStatus status;
    
    @Transient  // Field này không được persist vào database
    private double discount;
    
    // Getters and setters
}

enum ProductStatus {
    AVAILABLE, OUT_OF_STOCK, DISCONTINUED
}
```

### Các chiến lược Generation của Primary Key

```java
// IDENTITY - Auto-increment của database
@GeneratedValue(strategy = GenerationType.IDENTITY)
private Long id;

// SEQUENCE - Sử dụng database sequence
@GeneratedValue(strategy = GenerationType.SEQUENCE, generator = "user_seq")
@SequenceGenerator(name = "user_seq", sequenceName = "user_sequence", allocationSize = 1)
private Long id;

// TABLE - Sử dụng một table riêng để generate keys
@GeneratedValue(strategy = GenerationType.TABLE, generator = "user_gen")
@TableGenerator(name = "user_gen", table = "id_generator", pkColumnName = "gen_name", 
                valueColumnName = "gen_value", allocationSize = 1)
private Long id;

// AUTO - Hibernate tự chọn strategy phù hợp
@GeneratedValue(strategy = GenerationType.AUTO)
private Long id;
```

### Inheritance Mapping

Hibernate hỗ trợ 3 chiến lược mapping cho inheritance:

**Single Table (Table per class hierarchy)**

```java
@Entity
@Inheritance(strategy = InheritanceType.SINGLE_TABLE)
@DiscriminatorColumn(name = "payment_type")
public abstract class Payment {
    @Id
    @GeneratedValue
    private Long id;
    private BigDecimal amount;
}

@Entity
@DiscriminatorValue("CREDIT_CARD")
public class CreditCardPayment extends Payment {
    private String cardNumber;
}

@Entity
@DiscriminatorValue("BANK_TRANSFER")
public class BankTransferPayment extends Payment {
    private String bankAccount;
}
```

## 5. Hibernate Query Language (HQL)

### HQL là gì?

HQL (Hibernate Query Language) là ngôn ngữ query hướng đối tượng, tương tự SQL nhưng thao tác trên các Java objects thay vì database tables.

### So sánh HQL với SQL

| Đặc điểm | SQL | HQL |
|----------|-----|-----|
| Thao tác trên | Tables và Columns | Objects và Properties |
| Case sensitive | Không (phụ thuộc DB) | Có (với class và property names) |
| Database specific | Có | Không (database independent) |
| Joins | Explicit joins | Có thể sử dụng associations |

### Ví dụ HQL

**Simple queries**

```java
// Select all
String hql = "FROM User";
List<User> users = session.createQuery(hql, User.class).list();

// Select với điều kiện
String hql = "FROM User WHERE age > :age";
Query<User> query = session.createQuery(hql, User.class);
query.setParameter("age", 18);
List<User> users = query.list();

// Select specific columns
String hql = "SELECT u.name, u.email FROM User u WHERE u.age > 18";
List<Object[]> results = session.createQuery(hql).list();
```

**Joins trong HQL**

```java
// Inner join
String hql = "SELECT e FROM Employee e INNER JOIN e.department d WHERE d.name = :deptName";

// Left join
String hql = "SELECT e FROM Employee e LEFT JOIN e.department d";

// Fetch join (tránh N+1 problem)
String hql = "SELECT e FROM Employee e JOIN FETCH e.department";
```

**Aggregate functions**

```java
// Count
String hql = "SELECT COUNT(u) FROM User u";
Long count = (Long) session.createQuery(hql).uniqueResult();

// Average
String hql = "SELECT AVG(u.age) FROM User u";
Double avgAge = (Double) session.createQuery(hql).uniqueResult();

// Group by
String hql = "SELECT u.city, COUNT(u) FROM User u GROUP BY u.city";
List<Object[]> results = session.createQuery(hql).list();
```

**Update và Delete với HQL**

```java
// Update
String hql = "UPDATE User SET age = :newAge WHERE name = :name";
Query query = session.createQuery(hql);
query.setParameter("newAge", 30);
query.setParameter("name", "John");
int rowsAffected = query.executeUpdate();

// Delete
String hql = "DELETE FROM User WHERE age < :age";
Query query = session.createQuery(hql);
query.setParameter("age", 18);
int rowsDeleted = query.executeUpdate();
```

**Named Queries**

```java
// Định nghĩa trong Entity
@Entity
@NamedQueries({
    @NamedQuery(
        name = "User.findByEmail",
        query = "FROM User WHERE email = :email"
    ),
    @NamedQuery(
        name = "User.findAdults",
        query = "FROM User WHERE age >= 18"
    )
})
public class User {
    // ...
}

// Sử dụng
Query<User> query = session.createNamedQuery("User.findByEmail", User.class);
query.setParameter("email", "john@email.com");
User user = query.uniqueResult();
```

## 6. Lazy Loading vs Eager Loading

### Lazy Loading

Lazy loading là chiến lược load dữ liệu chỉ khi nó thực sự được truy cập, không load ngay lập tức.

**Ưu điểm:**
- Tiết kiệm memory
- Giảm số lượng queries ban đầu
- Tăng performance khi không cần toàn bộ data

**Nhược điểm:**
- Có thể gây N+1 query problem
- Cần session phải còn open khi access data
- Có thể gây LazyInitializationException

```java
@Entity
public class Department {
    @Id
    private Long id;
    
    // Mặc định, @OneToMany là LAZY
    @OneToMany(mappedBy = "department", fetch = FetchType.LAZY)
    private List<Employee> employees;
}

// Sử dụng
Department dept = session.get(Department.class, 1L);
// Query 1: SELECT * FROM departments WHERE id = 1

dept.getEmployees().size();  // Truy cập employees
// Query 2: SELECT * FROM employees WHERE department_id = 1
```

### Eager Loading

Eager loading load tất cả dữ liệu liên quan ngay lập tức trong một query.

**Ưu điểm:**
- Không có LazyInitializationException
- Tránh N+1 problem
- Dữ liệu sẵn sàng ngay cả khi session đóng

**Nhược điểm:**
- Tốn memory nếu load dữ liệu không cần thiết
- Query có thể chậm với dữ liệu lớn
- Cartesian product với multiple collections

```java
@Entity
public class Employee {
    @Id
    private Long id;
    
    // EAGER - load ngay department
    @ManyToOne(fetch = FetchType.EAGER)
    private Department department;
}

// Sử dụng
Employee emp = session.get(Employee.class, 1L);
// SELECT e.*, d.* FROM employees e LEFT JOIN departments d ON e.department_id = d.id WHERE e.id = 1
```

### Giải quyết N+1 Problem

**Vấn đề N+1:**

```java
// 1 query để lấy departments
List<Department> departments = session.createQuery("FROM Department", Department.class).list();

// N queries - một query cho mỗi department để lấy employees
for (Department dept : departments) {
    System.out.println(dept.getEmployees().size());  // Lazy load mỗi lần!
}
```

**Giải pháp 1: JOIN FETCH**

```java
String hql = "SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees";
List<Department> departments = session.createQuery(hql, Department.class).list();

// Chỉ 1 query duy nhất!
for (Department dept : departments) {
    System.out.println(dept.getEmployees().size());  // Không có query mới
}
```

**Giải pháp 2: Entity Graph**

```java
EntityGraph<Department> graph = session.createEntityGraph(Department.class);
graph.addAttributeNodes("employees");

Map<String, Object> hints = new HashMap<>();
hints.put("javax.persistence.loadgraph", graph);

Department dept = session.find(Department.class, 1L, hints);
```

**Giải pháp 3: Batch Fetching**

```java
@Entity
public class Department {
    @Id
    private Long id;
    
    @OneToMany(mappedBy = "department")
    @BatchSize(size = 10)  // Load tối đa 10 collections cùng lúc
    private List<Employee> employees;
}
```

## 7. Caching trong Hibernate

Hibernate cung cấp cơ chế caching để giảm số lần truy cập database và cải thiện performance.

### First-Level Cache (Session Cache)

First-level cache là cache mặc định, được bật tự động và gắn liền với Session object.

**Đặc điểm:**
- Phạm vi: Session scope
- Tự động bật, không thể tắt
- Lưu các entities đã load trong session
- Cache bị clear khi session close

```java
Session session = sessionFactory.openSession();

// Query 1: Truy cập database
User user1 = session.get(User.class, 1L);
System.out.println(user1.getName());

// Query 2: Lấy từ cache, KHÔNG truy cập database
User user2 = session.get(User.class, 1L);
System.out.println(user2.getName());

System.out.println(user1 == user2);  // true - cùng instance

session.close();
```

**Quản lý First-Level Cache:**

```java
Session session = sessionFactory.openSession();

User user = session.get(User.class, 1L);

// Xóa một entity khỏi cache
session.evict(user);

// Xóa tất cả entities khỏi cache
session.clear();

// Sync cache với database
session.flush();

session.close();
```

### Second-Level Cache

Second-level cache là cache ở cấp SessionFactory, được share giữa các sessions.

**Đặc điểm:**
- Phạm vi: SessionFactory scope (application-level)
- Cần configure và enable
- Cache entities, collections, queries
- Thread-safe

**Enable Second-Level Cache:**

```java
// hibernate.cfg.xml hoặc application.properties
hibernate.cache.use_second_level_cache=true
hibernate.cache.region.factory_class=org.hibernate.cache.jcache.JCacheRegionFactory
```

**Cấu hình caching cho Entity:**

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
public class Product {
    @Id
    private Long id;
    private String name;
    
    @OneToMany(mappedBy = "product")
    @org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_WRITE)
    private List<Review> reviews;
}
```

**Cache Concurrency Strategies:**

| Strategy | Use Case | Performance |
|----------|----------|-------------|
| READ_ONLY | Dữ liệu không thay đổi | Tốt nhất |
| READ_WRITE | Dữ liệu thay đổi thường xuyên | Trung bình |
| NONSTRICT_READ_WRITE | Dữ liệu thay đổi ít | Tốt |
| TRANSACTIONAL | Cần transaction isolation | Chậm nhất |

### Query Cache

Query cache lưu kết quả của các queries.

```java
// Enable query cache
hibernate.cache.use_query_cache=true

// Sử dụng trong code
Query<Product> query = session.createQuery("FROM Product WHERE price > :price", Product.class);
query.setParameter("price", 100);
query.setCacheable(true);  // Enable cache cho query này
query.setCacheRegion("productQueries");  // Optional: specify cache region

List<Product> products = query.list();
```

**Lưu ý về Query Cache:**
- Query cache chỉ lưu IDs của entities, không phải toàn bộ entity
- Phải kết hợp với second-level cache để hiệu quả
- Bị invalidate khi table thay đổi

### Cache Statistics

```java
Statistics stats = sessionFactory.getStatistics();
stats.setStatisticsEnabled(true);

// Thực hiện các operations...

System.out.println("Second level cache hits: " + stats.getSecondLevelCacheHitCount());
System.out.println("Second level cache miss: " + stats.getSecondLevelCacheMissCount());
System.out.println("Query cache hits: " + stats.getQueryCacheHitCount());
```

## 8. Transaction Management

### Transaction là gì?

Transaction là một đơn vị công việc logic, đảm bảo tính ACID:
- **Atomicity**: Tất cả hoặc không có thao tác nào được thực hiện
- **Consistency**: Dữ liệu luôn ở trạng thái hợp lệ
- **Isolation**: Các transactions không ảnh hưởng lẫn nhau
- **Durability**: Khi committed, dữ liệu được lưu vĩnh viễn

### Transaction trong Hibernate

```java
Session session = sessionFactory.openSession();
Transaction tx = null;

try {
    tx = session.beginTransaction();
    
    // Multiple operations trong cùng transaction
    User user = new User("John", "john@email.com");
    session.save(user);
    
    Account account = new Account(user, 1000.0);
    session.save(account);
    
    tx.commit();  // Commit nếu tất cả thành công
    
} catch (Exception e) {
    if (tx != null) {
        tx.rollback();  // Rollback nếu có lỗi
    }
    e.printStackTrace();
} finally {
    session.close();
}
```

### Isolation Levels

Hibernate hỗ trợ các mức độ isolation khác nhau:

```java
Session session = sessionFactory.openSession();
session.doWork(connection -> {
    connection.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
});

Transaction tx = session.beginTransaction();
// ... operations
tx.commit();
```

**Các mức isolation:**

| Level | Dirty Read | Non-repeatable Read | Phantom Read |
|-------|------------|---------------------|--------------|
| READ_UNCOMMITTED | Có | Có | Có |
| READ_COMMITTED | Không | Có | Có |
| REPEATABLE_READ | Không | Không | Có |
| SERIALIZABLE | Không | Không | Không |

### Optimistic Locking

Optimistic locking sử dụng versioning để phát hiện conflicts.

```java
@Entity
public class Product {
    @Id
    private Long id;
    
    private String name;
    
    @Version  // Hibernate tự động quản lý version
    private Integer version;
    
    // Getters and setters
}

// Sử dụng
Session session1 = sessionFactory.openSession();
Session session2 = sessionFactory.openSession();

// Session 1 lấy product
Transaction tx1 = session1.beginTransaction();
Product product1 = session1.get(Product.class, 1L);  // version = 1
product1.setName("New Name 1");

// Session 2 cũng lấy product
Transaction tx2 = session2.beginTransaction();
Product product2 = session2.get(Product.class, 1L);  // version = 1
product2.setName("New Name 2");

// Session 1 commit thành công
session1.update(product1);
tx1.commit();  // version = 2

// Session 2 commit - sẽ throw OptimisticLockException
session2.update(product2);
tx2.commit();  // FAIL! version conflict (expected 2, got 1)
```

### Pessimistic Locking

Pessimistic locking lock record ngay khi đọc.

```java
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

// Lock record khi select
Product product = session.get(Product.class, 1L, LockMode.PESSIMISTIC_WRITE);
// SQL: SELECT ... FROM products WHERE id = 1 FOR UPDATE

product.setPrice(new BigDecimal("99.99"));
session.update(product);

tx.commit();  // Release lock
session.close();
```

**Lock Modes:**
- `PESSIMISTIC_READ`: Shared lock, ngăn write
- `PESSIMISTIC_WRITE`: Exclusive lock, ngăn cả read và write
- `PESSIMISTIC_FORCE_INCREMENT`: Lock và tăng version

## 9. Ưu và Nhược điểm của Hibernate

### Ưu điểm

**1. Giảm Boilerplate Code**

So với JDBC thuần, Hibernate giảm đáng kể code cần viết, giúp developers tập trung vào business logic.

**2. Database Independence**

Code Java không phụ thuộc vào database cụ thể. Chuyển đổi giữa MySQL, PostgreSQL, Oracle chỉ cần thay đổi configuration.

**3. Automatic Table Creation**

Hibernate có thể tự động tạo tables từ entities (hữu ích trong development).

```java
hibernate.hbm2ddl.auto=update
```

**4. Powerful Query Options**

HQL, Criteria API, Native SQL - nhiều cách để query dữ liệu linh hoạt.

**5. Caching Mechanism**

First-level và second-level cache giúp tăng performance đáng kể.

**6. Transaction Management**

Quản lý transactions dễ dàng với rollback tự động khi có lỗi.

**7. Lazy Loading**

Load dữ liệu chỉ khi cần, tiết kiệm memory và bandwidth.

**8. Object-Oriented Approach**

Làm việc với objects thay vì SQL queries, phù hợp với tư duy OOP.

**9. Community và Documentation**

Cộng đồng lớn, tài liệu phong phú, nhiều tutorials và support.

### Nhược điểm

**1. Learning Curve**

Hibernate phức tạp, cần thời gian để hiểu rõ các concepts như caching, lazy loading, session management.

**2. Performance Overhead**

Vì là abstraction layer, Hibernate có overhead so với JDBC thuần hoặc SQL trực tiếp.

```java
// Hibernate generated SQL có thể không tối ưu như hand-written SQL
// Ví dụ: Hibernate có thể generate nhiều queries khi có thể dùng 1 query
```

**3. Complex Queries**

Với queries phức tạp, HQL có thể khó viết hơn SQL, và performance có thể không tốt bằng native SQL.

**4. Debugging Difficulty**

Khó debug khi có vấn đề vì SQL được generate tự động. Cần enable SQL logging để xem queries.

**5. N+1 Query Problem**

Nếu không cẩn thận với lazy loading, có thể gặp N+1 problem gây performance issue nghiêm trọng.

**6. Memory Consumption**

Session cache và second-level cache có thể tốn nhiều memory với large datasets.

**7. Overkill cho Simple Applications**

Với ứng dụng nhỏ, đơn giản, Hibernate có thể là quá phức tạp và không cần thiết.

**8. Steep Learning Curve cho Advanced Features**

Các tính năng như caching strategies, batch processing, stateless sessions đòi hỏi kiến thức sâu.

## 10. Best Practices khi sử dụng Hibernate

### 1. Luôn đóng Session và Transaction

```java
// BAD - có thể leak resources
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();
// ... operations
tx.commit();

// GOOD - luôn dùng try-with-resources hoặc finally
Session session = sessionFactory.openSession();
Transaction tx = null;
try {
    tx = session.beginTransaction();
    // ... operations
    tx.commit();
} catch (Exception e) {
    if (tx != null) tx.rollback();
    throw e;
} finally {
    session.close();
}

// BEST - Java 7+ try-with-resources
try (Session session = sessionFactory.openSession()) {
    Transaction tx = session.beginTransaction();
    // ... operations
    tx.commit();
}
```

### 2. Sử dụng JOIN FETCH để tránh N+1 Problem

```java
// BAD - N+1 queries
List<Department> departments = session.createQuery("FROM Department", Department.class).list();
for (Department dept : departments) {
    dept.getEmployees().size();  // Lazy load - mỗi department 1 query!
}

// GOOD - 1 query duy nhất
String hql = "SELECT DISTINCT d FROM Department d LEFT JOIN FETCH d.employees";
List<Department> departments = session.createQuery(hql, Department.class).list();
```

### 3. Sử dụng Pagination cho Large Result Sets

```java
// BAD - load tất cả records
List<User> users = session.createQuery("FROM User", User.class).list();

// GOOD - pagination
Query<User> query = session.createQuery("FROM User", User.class);
query.setFirstResult(0);     // Offset
query.setMaxResults(20);     // Limit
List<User> users = query.list();
```

### 4. Enable SQL Logging trong Development

```properties
# Hiển thị SQL queries
hibernate.show_sql=true
hibernate.format_sql=true

# Hiển thị bind parameters
logging.level.org.hibernate.type.descriptor.sql.BasicBinder=TRACE
```

### 5. Sử dụng Batch Processing cho Bulk Operations

```java
// BAD - commit từng record
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

for (int i = 0; i < 100000; i++) {
    User user = new User("User" + i, "user" + i + "@email.com");
    session.save(user);
}

tx.commit();  // OutOfMemoryError với dữ liệu lớn!

// GOOD - batch processing
Session session = sessionFactory.openSession();
Transaction tx = session.beginTransaction();

int batchSize = 50;
for (int i = 0; i < 100000; i++) {
    User user = new User("User" + i, "user" + i + "@email.com");
    session.save(user);
    
    if (i % batchSize == 0) {
        session.flush();   // Flush batch
        session.clear();   // Clear session cache
    }
}

tx.commit();
```

### 6. Chọn FetchType phù hợp

```java
// BAD - EAGER cho collections có thể tốn nhiều memory
@OneToMany(fetch = FetchType.EAGER)
private List<Order> orders;  // Load tất cả orders mỗi lần load Customer!

// GOOD - LAZY làm mặc định, dùng JOIN FETCH khi cần
@OneToMany(fetch = FetchType.LAZY)
private List<Order> orders;

// Khi cần, dùng JOIN FETCH
String hql = "SELECT c FROM Customer c JOIN FETCH c.orders WHERE c.id = :id";
```

### 7. Sử dụng DTO Projections cho Read-Only Data

```java
// BAD - load toàn bộ entity khi chỉ cần vài fields
List<User> users = session.createQuery("FROM User", User.class).list();
for (User user : users) {
    System.out.println(user.getName() + " - " + user.getEmail());
}

// GOOD - chỉ select fields cần thiết
String hql = "SELECT new com.example.UserDTO(u.name, u.email) FROM User u";
List<UserDTO> users = session.createQuery(hql, UserDTO.class).list();

// UserDTO class
public class UserDTO {
    private String name;
    private String email;
    
    public UserDTO(String name, String email) {
        this.name = name;
        this.email = email;
    }
}
```

### 8. Tránh Anti-patterns

**Anti-pattern 1: Session per Operation**

```java
// BAD
public User getUser(Long id) {
    Session session = sessionFactory.openSession();
    User user = session.get(User.class, id);
    session.close();
    return user;
}

public void updateUser(User user) {
    Session session = sessionFactory.openSession();
    Transaction tx = session.beginTransaction();
    session.update(user);
    tx.commit();
    session.close();
}

// GOOD - Session per Request (web apps)
// Sử dụng OpenSessionInView pattern hoặc manual session management
```

**Anti-pattern 2: Ignoring LazyInitializationException**

```java
// BAD
public User getUserWithOrders(Long id) {
    Session session = sessionFactory.openSession();
    User user = session.get(User.class, id);
    session.close();
    return user;  // orders chưa được load!
}

// Ở nơi khác
User user = getUserWithOrders(1L);
user.getOrders().size();  // LazyInitializationException!

// GOOD - load dữ liệu cần thiết trong session
public User getUserWithOrders(Long id) {
    Session session = sessionFactory.openSession();
    String hql = "SELECT u FROM User u JOIN FETCH u.orders WHERE u.id = :id";
    User user = session.createQuery(hql, User.class)
                       .setParameter("id", id)
                       .uniqueResult();
    session.close();
    return user;
}
```

### 9. Sử dụng @Immutable cho Read-Only Entities

```java
@Entity
@Immutable  // Hibernate sẽ không track changes
@Table(name = "countries")
public class Country {
    @Id
    private String code;
    private String name;
    
    // Getters only, no setters
}
```

### 10. Enable Second-Level Cache cho Frequently Accessed Data

```java
@Entity
@Cacheable
@org.hibernate.annotations.Cache(usage = CacheConcurrencyStrategy.READ_ONLY)
public class Category {
    @Id
    private Long id;
    private String name;
    // Category data ít thay đổi, phù hợp cho cache
}
```

### 11. Sử dụng Native Queries cho Complex Queries

```java
// Khi HQL quá phức tạp hoặc không đủ hiệu quả
String sql = "SELECT u.* FROM users u " +
             "INNER JOIN orders o ON u.id = o.user_id " +
             "WHERE o.total > :minTotal " +
             "GROUP BY u.id " +
             "HAVING COUNT(o.id) > :minOrders";

Query<User> query = session.createNativeQuery(sql, User.class);
query.setParameter("minTotal", 1000);
query.setParameter("minOrders", 5);
List<User> users = query.list();
```

### 12. Monitor và Tune Performance

```java
// Enable statistics
Statistics stats = sessionFactory.getStatistics();
stats.setStatisticsEnabled(true);

// Sau khi chạy operations
System.out.println("Queries executed: " + stats.getQueryExecutionCount());
System.out.println("Cache hits: " + stats.getSecondLevelCacheHitCount());
System.out.println("Cache misses: " + stats.getSecondLevelCacheMissCount());
System.out.println("Sessions opened: " + stats.getSessionOpenCount());
System.out.println("Sessions closed: " + stats.getSessionCloseCount());
```

