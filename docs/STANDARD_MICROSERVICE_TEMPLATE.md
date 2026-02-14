# Standard Microservice Architecture - Production Ready

## 🎯 Overview

This document defines the **standard structure and patterns** for ALL microservices in the ride-hailing system. Follow this template for consistency, maintainability, and production readiness.

---

## 🏗️ Standard Package Structure (DDD Hexagonal Architecture)

### **Template - Apply to ALL Services**

```
<service-name>/src/main/java/com/ridehailing/<service>/
│
├── domain/                          # ⭐ CORE DOMAIN LAYER (Pure Business Logic)
│   ├── model/                       # Entities & Aggregates
│   │   ├── <Aggregate>.java        # Aggregate Root (e.g., User, Trip, Driver)
│   │   └── <Entity>.java           # Domain Entities
│   ├── valueobject/                # Value Objects (Immutable)
│   │   ├── <ValueObject>.java      # e.g., Money, Address, Coordinates
│   │   └── ...
│   ├── enums/                      # Domain Enumerations
│   │   └── <Status>.java           # e.g., UserStatus, TripStatus
│   ├── repository/                 # Repository Interfaces (Ports)
│   │   └── <Entity>Repository.java # Interface only, no implementation
│   ├── service/                    # Domain Services (Complex business logic)
│   │   └── <Domain>Service.java   # e.g., PricingCalculator
│   └── event/                      # Domain Events
│       └── <Event>.java            # e.g., TripCompletedEvent
│
├── application/                     # ⚙️ APPLICATION LAYER (Use Cases)
│   ├── service/                    # Application Services (Orchestration)
│   │   └── <UseCase>Service.java  # e.g., CreateTripService
│   ├── dto/                        # Data Transfer Objects
│   │   ├── request/
│   │   │   └── <Action>Request.java
│   │   └── response/
│   │       └── <Action>Response.java
│   ├── mapper/                     # Entity ↔ DTO Mappers
│   │   └── <Entity>Mapper.java    # MapStruct recommended
│   ├── command/                    # CQRS Commands (Write operations)
│   │   └── <Action>Command.java
│   ├── query/                      # CQRS Queries (Read operations)
│   │   └── <Entity>Query.java
│   └── validator/                  # Business validation
│       └── <Action>Validator.java
│
├── infrastructure/                  # 🔧 INFRASTRUCTURE LAYER (Technical Details)
│   ├── persistence/                # Database Implementations
│   │   ├── Jpa<Entity>Repository.java  # Implements domain repository
│   │   ├── entity/                 # JPA Entities (if separating from domain)
│   │   └── mapper/                 # Domain ↔ JPA Mapper
│   ├── messaging/                  # Event Bus / Kafka
│   │   ├── producer/
│   │   │   └── <Event>Producer.java
│   │   └── consumer/
│   │       └── <Event>Consumer.java
│   ├── config/                     # Spring Configuration
│   │   ├── DatabaseConfig.java
│   │   ├── KafkaConfig.java
│   │   ├── SecurityConfig.java
│   │   └── AsyncConfig.java
│   ├── security/                   # Authentication & Authorization
│   │   ├── JwtUtils.java           # (IAM only)
│   │   ├── JwtAuthenticationFilter.java
│   │   └── SecurityContextProvider.java
│   ├── external/                   # External Service Clients
│   │   └── <Service>Client.java   # e.g., PaymentGatewayClient
│   └── outbox/                     # Outbox Pattern (Transactional Messaging)
│       ├── OutboxEvent.java
│       ├── OutboxEventRepository.java
│       └── OutboxPublisher.java
│
└── api/                            # 📡 PRESENTATION LAYER (REST API)
    ├── controller/                 # REST Controllers
    │   └── <Entity>Controller.java
    ├── graphql/                    # GraphQL (optional)
    │   └── <Entity>Resolver.java
    ├── websocket/                  # WebSocket (real-time)
    │   └── <Event>WebSocketHandler.java
    ├── advice/                     # Exception Handling
    │   └── GlobalExceptionHandler.java
    └── filter/                     # HTTP Filters
        └── RequestLoggingFilter.java
```

---

## 📦 Common-Lib Structure (Shared Across Services)

```
common-lib/src/main/java/com/ridehailing/common/
│
├── domain/                          # Shared Domain Primitives
│   ├── BaseEntity.java             # id, createdAt
│   ├── BaseAuditEntity.java        # + updatedAt, createdBy, updatedBy
│   ├── DomainEvent.java            # Base event class
│   └── AggregateRoot.java          # Marker interface
│
├── valueobject/                    # Shared Value Objects
│   ├── Money.java                  # Currency + amount
│   ├── Coordinates.java            # lat, lng
│   └── PhoneNumber.java
│
├── exception/                      # Standard Exceptions
│   ├── BusinessException.java
│   ├── NotFoundException.java
│   ├── ValidationException.java
│   ├── UnauthorizedException.java
│   └── ServiceUnavailableException.java
│
├── dto/                            # Cross-Service DTOs
│   ├── ApiResponse.java            # Standard wrapper
│   ├── ErrorResponse.java
│   └── PageResponse.java           # Pagination
│
├── util/                           # Generic Utilities
│   ├── DateTimeUtil.java
│   ├── StringUtil.java
│   ├── ValidationUtil.java
│   └── JsonUtil.java
│
├── constant/                       # Global Constants
│   ├── ErrorCodes.java
│   └── HttpHeaders.java
│
├── annotation/                     # Custom Annotations
│   ├── @DomainService.java
│   └── @AuditLog.java
│
└── event/                          # Cross-Service Events
    ├── UserCreatedEvent.java
    └── TripCompletedEvent.java
```

---

## 🧪 Testing Structure (Per Service)

```
<service-name>/src/test/java/com/ridehailing/<service>/
│
├── domain/                         # Unit Tests (Pure Domain Logic)
│   └── model/
│       └── <Entity>Test.java
│
├── application/                    # Service Tests (Mock dependencies)
│   └── service/
│       └── <UseCase>ServiceTest.java
│
├── infrastructure/                 # Integration Tests
│   ├── persistence/
│   │   └── Jpa<Entity>RepositoryTest.java  # @DataJpaTest
│   └── messaging/
│       └── KafkaIntegrationTest.java        # @EmbeddedKafka
│
├── api/                            # Controller Tests
│   └── controller/
│       └── <Entity>ControllerTest.java      # @WebMvcTest
│
└── e2e/                            # End-to-End Tests
    └── <Feature>E2ETest.java                # @SpringBootTest
```

---

## 📊 Production-Ready Patterns

### **1. CQRS (Command Query Responsibility Segregation)**

**Separate read and write operations**

```java
// Command (Write)
@Service
public class CreateTripCommandHandler {
    public TripId handle(CreateTripCommand command) {
        // Validate, create entity, save, publish event
    }
}

// Query (Read)
@Service
public class TripQueryService {
    public TripResponse findById(TripId id) {
        // Optimized for reading
    }
}
```

**When to use**:
- High read/write ratio
- Complex domain logic
- Need for separate read models

---

### **2. Event Sourcing** (Advanced - Optional)

**Store state changes as events**

```java
@Entity
public class Trip extends AggregateRoot {
    private List<DomainEvent> uncommittedEvents = new ArrayList<>();
    
    public void complete() {
        apply(new TripCompletedEvent(this.id));
    }
    
    private void apply(DomainEvent event) {
        // Update state based on event
        uncommittedEvents.add(event);
    }
}
```

**EventStore Schema**:
```sql
CREATE TABLE event_store (
    id UUID PRIMARY KEY,
    aggregate_id UUID NOT NULL,
    aggregate_type VARCHAR(100),
    event_type VARCHAR(100),
    event_data JSONB,
    version INT,
    created_at TIMESTAMP
);
```

---

### **3. Saga Pattern (Distributed Transactions)**

**Outbox Pattern** (All services MUST implement):

```sql
-- In EVERY service database
CREATE TABLE outbox_events (
    id UUID PRIMARY KEY,
    aggregate_type VARCHAR(255),
    aggregate_id VARCHAR(255),
    event_type VARCHAR(255),
    event_payload JSONB,
    created_at TIMESTAMP DEFAULT NOW(),
    published_at TIMESTAMP NULL,
    status VARCHAR(50) DEFAULT 'PENDING'
);
```

**Publisher** (Background job in each service):
```java
@Scheduled(fixedDelay = 5000)
public void publishPendingEvents() {
    List<OutboxEvent> events = outboxRepository.findPending();
    events.forEach(event -> {
        kafkaTemplate.send(event.getEventType(), event.getPayload());
        event.markAsPublished();
    });
}
```

---

### **4. Circuit Breaker** (Resilience)

```java
@Service
public class PaymentServiceClient {
    
    @CircuitBreaker(name = "payment", fallbackMethod = "fallback")
    @Retry(name = "payment")
    public PaymentResponse charge(ChargeRequest request) {
        return restTemplate.postForObject(url, request, PaymentResponse.class);
    }
    
    private PaymentResponse fallback(ChargeRequest request, Exception e) {
        // Return cached response or queue for later
    }
}
```

**Config** (`application.yml`):
```yaml
resilience4j:
  circuitbreaker:
    instances:
      payment:
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        sliding-window-size: 10
```

---

### **5. API Gateway Pattern**

**Features**:
- ✅ Routing
- ✅ Authentication (JWT validation)
- ✅ Rate limiting
- ✅ Request/Response transformation
- ✅ Load balancing

**Kong Configuration** (Example):
```yaml
services:
  - name: iam-service
    url: http://iam-service:8081
    routes:
      - paths: ["/api/auth/*"]
    plugins:
      - name: jwt
      - name: rate-limiting
        config:
          minute: 60
```

---

## 🔐 Security Standards

### **1. JWT Authentication**

**IAM Service** (Generate):
```java
// infrastructure/security/JwtUtils.java
public String generateAccessToken(User user) {
    return Jwts.builder()
        .setSubject(user.getId().toString())
        .claim("roles", user.getRoles())
        .setIssuedAt(new Date())
        .setExpiration(new Date(System.currentTimeMillis() + 900000))
        .signWith(secretKey)
        .compact();
}
```

**Other Services** (Verify only):
```java
// infrastructure/security/JwtAuthenticationFilter.java
public void doFilterInternal(HttpServletRequest request) {
    String token = extractToken(request);
    if (jwtUtils.validateToken(token)) {
        UserContext context = jwtUtils.parseToken(token);
        SecurityContextHolder.setAuthentication(context);
    }
}
```

### **2. RBAC (Role-Based Access Control)**

```java
@PreAuthorize("hasRole('DRIVER')")
@GetMapping("/api/drivers/me/earnings")
public EarningsResponse getMyEarnings() {
    // Only drivers can access
}

@PreAuthorize("hasPermission('TRIP', 'CREATE')")
@PostMapping("/api/trips")
public TripResponse createTrip(@RequestBody CreateTripRequest request) {
    // Fine-grained permission check
}
```

---

## 📈 Observability Standards

### **1. Logging**

```java
@Slf4j
@Service
public class TripService {
    
    public TripResponse createTrip(CreateTripRequest request) {
        log.info("Creating trip for customer: {}", request.getCustomerId());
        try {
            // Business logic
            log.debug("Trip created successfully: {}", trip.getId());
        } catch (Exception e) {
            log.error("Failed to create trip", e);
            throw e;
        }
    }
}
```

**Structured Logging** (logback-spring.xml):
```xml
<encoder class="net.logstash.logback.encoder.LogstashEncoder">
    <customFields>{"service":"trip-service"}</customFields>
</encoder>
```

### **2. Metrics**

```java
@Timed(value = "trip.create.time")
@Counted(value = "trip.create.count")
public TripResponse createTrip(CreateTripRequest request) {
    // Automatically tracked by Micrometer
}
```

### **3. Distributed Tracing**

```yaml
# application.yml
management:
  tracing:
    sampling:
      probability: 0.1  # 10% sampling
  zipkin:
    tracing:
      endpoint: http://zipkin:9411/api/v2/spans
```

---

## 🚀 Deployment Standards

### **1. Dockerfile** (Every service)

```dockerfile
FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
COPY build/libs/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### **2. Kubernetes Deployment**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: trip-service
spec:
  replicas: 3
  template:
    spec:
      containers:
      - name: trip-service
        image: ridehailing/trip-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: prod
        resources:
          requests:
            memory: "512Mi"
            cpu: "500m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /actuator/health/liveness
            port: 8080
        readinessProbe:
          httpGet:
            path: /actuator/health/readiness
            port: 8080
```

---

## ✅ Production Readiness Checklist

### **Each Service MUST Have**:

- [ ] DDD hexagonal architecture structure
- [ ] Unit tests (>80% coverage)
- [ ] Integration tests
- [ ] Outbox pattern for events
- [ ] Circuit breaker for external calls
- [ ] Health endpoints (`/actuator/health`)
- [ ] Metrics endpoint (`/actuator/prometheus`)
- [ ] Structured logging (JSON)
- [ ] Distributed tracing (Zipkin headers)
- [ ] API documentation (OpenAPI/Swagger)
- [ ] Docker image
- [ ] Kubernetes manifests
- [ ] CI/CD pipeline

---

## 🎯 Summary

| Layer | Responsibility | Examples |
|-------|---------------|----------|
| **Domain** | Business logic, entities | User, Trip, Money |
| **Application** | Use cases, orchestration | CreateTripService |
| **Infrastructure** | Technical details | JpaRepository, KafkaProducer |
| **API** | HTTP endpoints | TripController |

**Key Principles**:
- ✅ Domain layer has ZERO dependencies
- ✅ Dependencies point INWARD (Hexagonal)
- ✅ Infrastructure implements domain interfaces
- ✅ All cross-service communication via events
- ✅ Transactions = single service only

---

**Apply this template to ALL 12 services for consistency!** 🚀
