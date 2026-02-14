# Grap System - Master Architecture Document

> **Last Updated**: 2026-02-14  
> **Production Readiness**: 15.9%  
> **Current Phase**: Phase 3 - Domain Models

---

## 📚 Table of Contents

1. [System Overview](#system-overview)
2. [Technology Stack](#technology-stack)
3. [Architecture Patterns](#architecture-patterns)
4. [Service Catalog](#service-catalog)
5. [Development Workflow](#development-workflow)
6. [Quick Reference](#quick-reference)

---

## 🎯 System Overview

**Grap** is a production-ready ride-hailing microservices system built with modern enterprise patterns.

### **Business Capabilities**
- 🔐 Identity & Access Management (IAM)
- 👤 Customer Profile Management
- 🚗 Driver Management & Onboarding
- 🗺️ Real-time Location Tracking
- 🚕 Trip Matching & Management
- 💰 Dynamic Pricing
- 💳 Payment Processing
- 📧 Multi-channel Notifications
- 📊 Analytics & Reporting

### **Non-Functional Requirements**
- **Scalability**: Thousands of concurrent trips
- **Availability**: 99.9% uptime
- **Performance**: <100ms API response, <5s driver matching
- **Security**: OAuth2, JWT, RBAC, MFA
- **Observability**: Distributed tracing, centralized logging, metrics

---

## 🛠️ Technology Stack

### **Core Technologies**
| Layer | Technology | Version | Purpose |
|-------|-----------|---------|---------|
| Language | Java | 21 | Main programming language |
| Framework | Spring Boot | 3.3.5 | Microservices framework |
| Build Tool | Gradle | 8.x | Dependency & build management |

### **Data Layer**
| Component | Technology | Purpose |
|-----------|-----------|---------|
| Primary DB | PostgreSQL 15 | Transactional data |
| Migrations | Flyway | Schema versioning |
| Cache | Redis 7 | Session, rate limiting, geospatial |
| Search | Elasticsearch | Trip history, analytics (planned) |

### **Infrastructure**
| Component | Technology | Purpose |
|-----------|-----------|---------|
| Messaging | Apache Kafka | Event-driven architecture |
| Config | Spring Cloud Config | Centralized configuration |
| Discovery | Eureka | Service registration & discovery |
| Gateway | Kong / Spring Cloud Gateway | API Gateway |
| Orchestration | Kubernetes | Container orchestration (planned) |

### **Observability**
| Tool | Purpose |
|------|---------|
| Zipkin/Jaeger | Distributed tracing |
| Prometheus | Metrics collection |
| Grafana | Visualization & dashboards |
| ELK Stack | Centralized logging |

---

## 🏛️ Architecture Patterns

### **1. DDD Hexagonal Architecture** ✅ CORE
**Purpose**: Separate business logic from infrastructure

**Structure**:
```
domain/         → Pure business logic (zero dependencies)
application/    → Use cases & orchestration
infrastructure/ → Technical implementation (DB, Kafka, etc.)
api/            → REST endpoints
```

**Benefits**:
- Testable domain logic
- Replaceable infrastructure
- Clear boundaries

### **2. Event-Driven Architecture** ✅ REQUIRED
**Purpose**: Loose coupling between services

**Patterns**:
- **Saga**: Distributed transactions (Choreography-based)
- **Outbox**: Guaranteed event delivery
- **CQRS**: Separate read/write models (optional)

**Flow Example**:
```
Trip Completed → Event Published → Kafka
  ├→ Payment Service (charge customer)
  ├→ Driver Service (update earnings)
  └→ Notification Service (send receipt)
```

### **3. Resilience Patterns** ✅ REQUIRED
**Patterns**:
- **Circuit Breaker**: Prevent cascade failures
- **Retry**: Transient failure handling
- **Timeout**: Prevent hanging requests
- **Bulkhead**: Resource isolation

**Implementation**: Resilience4j

### **4. Security Patterns**
**Authentication**:
- JWT access tokens (15 min)
- Refresh tokens (7 days)
- OAuth2 (Google login)

**Authorization**:
- RBAC (Role-Based Access Control)
- Permission-based (@PreAuthorize)
- Resource-level permissions (optional)

**Advanced** (Phase 9-10):
- MFA (Multi-Factor Authentication)
- Advanced RBAC (hierarchical roles)
- ABAC (Attribute-Based Access Control)

---

## 📦 Service Catalog

### **Infrastructure Services**
| Service | Port | Status | Purpose |
|---------|------|--------|---------|
| Config Server | 8888 | ✅ Running | Centralized configuration |
| Discovery Server | 8761 | ✅ Running | Service registry (Eureka) |
| API Gateway | 8080 | ⚠️ Planned | Routing, auth, rate limiting |

### **Business Services**
| Service | Port | Status | Phase | Database |
|---------|------|--------|-------|----------|
| IAM Service | 8081 | 🚧 Phase 3 | Auth, users, RBAC | iam_db |
| Customer Service | 8082 | ⏳ Planned | Profiles, preferences | customer_db |
| Driver Service | 8083 | ⏳ Planned | Drivers, vehicles | driver_db |
| Trip Service | 8084 | ⏳ Planned | Trip lifecycle | trip_db |
| Location Service | 8085 | ⏳ Planned | Real-time tracking | MongoDB + Redis |
| Pricing Service | 8086 | ⏳ Planned | Fare calculation | pricing_db |
| Payment Service | 8087 | ⏳ Planned | Transactions | payment_db |
| Notification Service | 8088 | ⏳ Planned | Email, SMS, push | notification_db |
| Analytics Service | 8089 | ⏳ Planned | Reporting | Elasticsearch |

---

## 🔄 Development Workflow

### **1. Project Structure**
```
grap-system/
├── common-lib/              # Shared utilities & base classes
├── config-server/           # Configuration server
├── discovery-server/        # Eureka server
├── api-gateway/            # API Gateway
├── iam-service/            # 🚧 Current focus
├── customer-service/
├── driver-service/
├── trip-service/
├── location-service/
├── pricing-service/
├── payment-service/
├── notification-service/
├── analytics-service/
├── infra/
│   ├── docker/            # Docker Compose files
│   ├── scripts/           # Start/stop scripts
│   └── docs/              # Infrastructure docs
└── build.gradle           # Root build file

grap-system-config/         # External config repo
├── application.yml         # Global config
├── application-dev.yml     # Dev environment
├── iam-service.yml         # IAM common config
├── iam-service-dev.yml     # IAM dev config
└── docs/                   # 📚 THIS FOLDER
    ├── README.md
    ├── MASTER_ARCHITECTURE.md
    ├── STANDARD_MICROSERVICE_TEMPLATE.md
    ├── CURRENT_STATUS.md
    ├── PHASE3_DOMAIN_MODELS.md
    └── PATTERNS_EXPLAINED.md
```

### **2. Daily Development Cycle**

**A. Start Infrastructure**:
```powershell
cd grap-system
./infra/scripts/start-infrastructure.ps1
# Starts: PostgreSQL, Redis, Kafka, Config Server
```

**B. Run Service** (Example: IAM):
```powershell
./infra/scripts/run-iam-dev.ps1
# Sets env vars + starts IAM service
```

**C. Verify**:
```powershell
# Health check
curl http://localhost:8081/actuator/health

# Check database
# pgAdmin: http://localhost:5050
```

**D. Code**:
- Follow DDD structure (see `STANDARD_MICROSERVICE_TEMPLATE.md`)
- Write tests
- Commit frequently

**E. Stop**:
```powershell
# Ctrl+C in terminals
./infra/scripts/stop-infrastructure.ps1
```

### **3. Git Workflow**
```bash
# Code in grap-system
cd grap-system
git add .
git commit -m "feat: implement User entity"
git push

# Config in grap-system-config
cd ../grap-system-config
git add .
git commit -m "config: add JWT secret for dev"
git push
```

---

## 🎓 Learning Path

### **Completed** ✅
- [x] Phase 1: Infrastructure Setup
- [x] Phase 2: IAM Database Schema

### **Current** 🚧
- [/] Phase 3: Domain Models (Week 1-2)
  - Common-lib base classes
  - IAM entities (User, Role, Permission)
  - Repositories

### **Upcoming** ⏳
- [ ] Phase 4: Repositories (Week 2)
- [ ] Phase 5: Security (JWT, OAuth2) (Week 3)
- [ ] Phase 6: Application Services (Week 4)
- [ ] Phase 7: REST APIs (Week 4-5)
- [ ] Phase 8: Event-Driven (Saga, Outbox) (Week 5-6)
- [ ] Phase 9-14: Advanced Features (Week 7-16)

**Total Timeline**: 16 weeks full implementation

---

## 📖 Quick Reference

### **Essential Commands**

```powershell
# Build entire project
./gradlew build

# Build specific service
./gradlew :iam-service:build

# Run tests
./gradlew :iam-service:test

# Clean build
./gradlew clean build

# Check dependencies
./gradlew :iam-service:dependencies
```

### **Database Access**

**pgAdmin**:
- URL: http://localhost:5050
- Email: admin@ridehailing.com
- Password: admin123

**Connection**:
- Host: postgres (or localhost)
- Port: 5432
- Database: iam_db
- Username: admin
- Password: admin123

### **Service URLs**

| Service | Dev URL | Actuator |
|---------|---------|----------|
| Config Server | http://localhost:8888 | /actuator |
| Discovery | http://localhost:8761 | /actuator |
| IAM | http://localhost:8081 | /actuator/health |

### **Important Files**

**Documentation** (This folder):
- Start here: `README.md`
- Architecture: `MASTER_ARCHITECTURE.md` (this file)
- Standards: `STANDARD_MICROSERVICE_TEMPLATE.md`
- Status: `CURRENT_STATUS.md`

**Config**:
- Global: `grap-system-config/application.yml`
- Dev: `grap-system-config/application-dev.yml`
- IAM: `grap-system-config/iam-service-dev.yml`

**Build**:
- Root: `grap-system/build.gradle`
- IAM: `grap-system/iam-service/build.gradle`

**Database**:
- IAM Schema: `grap-system/iam-service/src/main/resources/db/migration/`

---

## 🚀 Next Steps

1. **Read** the other docs in this folder
2. **Follow** `PHASE3_DOMAIN_MODELS.md` for current implementation
3. **Code** following `STANDARD_MICROSERVICE_TEMPLATE.md`
4. **Check** `CURRENT_STATUS.md` for progress tracking

---

## 🤝 Support

**Documentation Location**: 
```
grap-system-config/docs/
```

**Version Control**: All docs committed to git for cross-machine access.

**Keep Updated**: Update docs when architecture evolves!

---

**Happy Coding!** 🚀
