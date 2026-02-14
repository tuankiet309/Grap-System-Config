# IAM Service - Phase 3: Domain Models (UPDATED)

## 🏗️ DDD Hexagonal Architecture Structure

### **Package Structure** (Following DDD)

```
iam-service/src/main/java/com/ridehailing/iam/
├── domain/                          # CORE DOMAIN LAYER
│   ├── model/                       # Entities & Aggregates
│   │   ├── User.java               # Aggregate Root
│   │   ├── Role.java               # Entity
│   │   ├── Permission.java         # Entity
│   │   ├── RefreshToken.java       # Entity
│   │   └── AuthEvent.java          # Entity
│   ├── valueobject/                # Value Objects (immutable)
│   │   ├── Email.java              # (Future: validation logic)
│   │   └── PhoneNumber.java
│   ├── enums/                      # Enumerations
│   │   ├── UserStatus.java
│   │   └── EventType.java
│   └── repository/                 # Repository Interfaces (ports)
│       ├── UserRepository.java
│       ├── RoleRepository.java
│       └── RefreshTokenRepository.java
│
├── application/                     # APPLICATION LAYER
│   ├── service/                    # Application Services
│   │   ├── AuthService.java       # (Phase 6)
│   │   ├── UserService.java
│   │   └── TokenService.java
│   ├── dto/                        # Data Transfer Objects
│   │   ├── request/
│   │   │   ├── LoginRequest.java
│   │   │   └── RegisterRequest.java
│   │   └── response/
│   │       ├── LoginResponse.java
│   │       └── UserResponse.java
│   └── mapper/                     # Entity ↔ DTO mappers
│       └── UserMapper.java
│
├── infrastructure/                  # INFRASTRUCTURE LAYER
│   ├── persistence/                # JPA implementations
│   │   ├── JpaUserRepository.java  # Implements domain repository
│   │   └── entity/                 # (Optional: if separating JPA from domain)
│   ├── config/                     # Configuration
│   │   ├── SecurityConfig.java
│   │   └── JwtConfig.java
│   └── external/                   # External integrations
│       └── OAuth2Provider.java
│
└── api/                            # PRESENTATION LAYER (REST)
    ├── controller/
    │   ├── AuthController.java
    │   └── UserController.java
    └── advice/
        └── GlobalExceptionHandler.java
```

---

## 🎯 Phase 3 Focus: Domain Layer Only

**Tạo gì trong Phase 3**:
- ✅ `domain/model/*` - Entities
- ✅ `domain/enums/*` - Enums
- ❌ `domain/repository/*` - (Phase 4)
- ❌ `application/*` - (Phase 6)
- ❌ `infrastructure/*` - (Phase 5)
- ❌ `api/*` - (Phase 7)

---

## 📁 Detailed: Domain Layer (Phase 3)

### **1. Entities**
```
domain/model/
├── User.java                 # Aggregate Root
├── Role.java                 # Entity
├── Permission.java           # Entity
├── RefreshToken.java         # Entity
└── AuthEvent.java            # Entity
```

### **2. Enums**
```
domain/enums/
├── UserStatus.java           # ACTIVE, INACTIVE, LOCKED
└── EventType.java            # LOGIN, LOGOUT, REGISTER...
```

---

## 🔧 Common-Lib Structure

```
common-lib/src/main/java/com/ridehailing/common/
├── domain/
│   ├── BaseEntity.java              # Base for all entities
│   ├── BaseAuditEntity.java         # + audit fields
│   └── DomainEvent.java             # (Future: event sourcing)
├── exception/
│   ├── BusinessException.java
│   ├── NotFoundException.java
│   └── ValidationException.java
├── util/
│   └── DateTimeUtil.java
└── constant/
    └── Constants.java
```

---

## 📝 Entity Details (Unchanged from previous plan)

### **User.java** (Aggregate Root)
```java
@Entity
@Table(name = "users")
@Getter @Setter
@NoArgsConstructor
public class User extends BaseAuditEntity {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(nullable = false, unique = true, length = 50)
    private String username;
    
    @Column(nullable = false, unique = true, length = 100)
    private String email;
    
    @Column(name = "password_hash", nullable = false)
    private String passwordHash;
    
    // Profile fields
    @Column(name = "first_name", length = 100)
    private String firstName;
    
    @Column(name = "last_name", length = 100)
    private String lastName;
    
    @Column(unique = true, length = 20)
    private String phone;
    
    @Column(name = "avatar_url", length = 500)
    private String avatarUrl;
    
    // Status
    @Enumerated(EnumType.STRING)
    @Column(nullable = false, length = 20)
    private UserStatus status = UserStatus.ACTIVE;
    
    @Column(name = "email_verified")
    private Boolean emailVerified = false;
    
    // Security tracking
    @Column(name = "failed_login_attempts")
    private Integer failedLoginAttempts = 0;
    
    @Column(name = "account_locked_until")
    private LocalDateTime accountLockedUntil;
    
    @Column(name = "last_login_at")
    private LocalDateTime lastLoginAt;
    
    @Column(name = "last_login_ip", length = 45)
    private String lastLoginIp;
    
    // Password management
    @Column(name = "password_changed_at")
    private LocalDateTime passwordChangedAt;
    
    @Column(name = "password_reset_token")
    private String passwordResetToken;
    
    @Column(name = "password_reset_expires")
    private LocalDateTime passwordResetExpires;
    
    // Relationships
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "user_roles",
        joinColumns = @JoinColumn(name = "user_id"),
        inverseJoinColumns = @JoinColumn(name = "role_id")
    )
    private Set<Role> roles = new HashSet<>();
    
    // Business methods
    public boolean isAccountLocked() {
        return accountLockedUntil != null && 
               accountLockedUntil.isAfter(LocalDateTime.now());
    }
    
    public void incrementFailedLogins() {
        this.failedLoginAttempts++;
        if (this.failedLoginAttempts >= 5) {
            this.accountLockedUntil = LocalDateTime.now().plusHours(1);
            this.status = UserStatus.LOCKED;
        }
    }
    
    public void resetFailedLogins() {
        this.failedLoginAttempts = 0;
        this.accountLockedUntil = null;
    }
}
```

### **Role.java**
```java
@Entity
@Table(name = "roles")
@Getter @Setter
@NoArgsConstructor
public class Role {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(nullable = false, unique = true, length = 50)
    private String name;
    
    @Column(length = 255)
    private String description;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @ManyToMany(fetch = FetchType.LAZY)
    @JoinTable(
        name = "role_permissions",
        joinColumns = @JoinColumn(name = "role_id"),
        inverseJoinColumns = @JoinColumn(name = "permission_id")
    )
    private Set<Permission> permissions = new HashSet<>();
}
```

### **Permission.java**
```java
@Entity
@Table(name = "permissions", 
       uniqueConstraints = @UniqueConstraint(columnNames = {"resource", "action"}))
@Getter @Setter
@NoArgsConstructor
public class Permission {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(nullable = false, length = 100)
    private String resource;  // TRIP, PROFILE, USER
    
    @Column(nullable = false, length = 50)
    private String action;    // CREATE, READ, UPDATE, DELETE
    
    @Column(length = 255)
    private String description;
}
```

### **RefreshToken.java**
```java
@Entity
@Table(name = "refresh_tokens")
@Getter @Setter
@NoArgsConstructor
public class RefreshToken {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(name = "user_id", nullable = false)
    private UUID userId;
    
    @Column(name = "token_hash", nullable = false, unique = true)
    private String tokenHash;
    
    @Column(name = "family_id", nullable = false)
    private UUID familyId;
    
    @Column(name = "expires_at", nullable = false)
    private LocalDateTime expiresAt;
    
    @Column(nullable = false)
    private Boolean revoked = false;
    
    @Column(name = "revoked_at")
    private LocalDateTime revokedAt;
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", insertable = false, updatable = false)
    private User user;
}
```

### **AuthEvent.java**
```java
@Entity
@Table(name = "auth_events")
@Getter @Setter
@NoArgsConstructor
public class AuthEvent {
    
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private UUID id;
    
    @Column(name = "user_id")
    private UUID userId;
    
    @Enumerated(EnumType.STRING)
    @Column(name = "event_type", nullable = false, length = 50)
    private EventType eventType;
    
    @Column(name = "ip_address", length = 45)
    private String ipAddress;
    
    @Column(name = "user_agent")
    private String userAgent;
    
    @Column(nullable = false)
    private Boolean success = true;
    
    @Column(name = "error_message", columnDefinition = "TEXT")
    private String errorMessage;
    
    @Column(columnDefinition = "jsonb")
    private String metadata;  // JSON string
    
    @CreationTimestamp
    @Column(name = "created_at", nullable = false, updatable = false)
    private LocalDateTime createdAt;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name = "user_id", insertable = false, updatable = false)
    private User user;
}
```

---

## 🔧 Implementation Order

1. **Common-lib** base classes (15 min)
2. **Enums** (10 min)
3. **User** entity (30 min)
4. **Role & Permission** (20 min)
5. **Add relationships** (20 min)
6. **RefreshToken & AuthEvent** (20 min)

**Total**: ~2 hours

---

## ✅ Success Criteria

- [ ] DDD package structure created
- [ ] Base classes in common-lib
- [ ] All 5 entities implemented
- [ ] Enums defined
- [ ] Relationships configured
- [ ] Build passes: `./gradlew :iam-service:build`

---

**Key Difference**: Proper DDD layered architecture with clear separation! 🎯
