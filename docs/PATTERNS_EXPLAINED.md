# Architecture Patterns - Detailed Explanation

## 📚 Pattern Catalog with Use Cases & Complexity

This document explains **WHY** each pattern exists, **WHEN** to use it, and the **COST** of implementation.

---

## 1. Advanced RBAC (Role-Based Access Control)

### **Basic RBAC** (Included in Phase 5)
```
User → Role → Permissions
Simple: hasRole('DRIVER')
```

### **Advanced RBAC** (Phase 8-9)

#### **Hierarchical Roles**
**Problem**: Admin should inherit all Manager permissions
**Solution**: Parent-child role relationships
**Complexity**: Medium
**Learning Curve**: 1-2 weeks

#### **Resource-Level Permissions**
**Problem**: User can edit THEIR trip only, not others'
**Solution**: Permission evaluators check ownership
**Complexity**: Medium-High
**Learning Curve**: 2 weeks

#### **ABAC (Attribute-Based)**
**Problem**: "Verified drivers in same city only"
**Solution**: Permissions based on attributes
**Complexity**: High
**Learning Curve**: 2-3 weeks

**When to use**: Multi-tenant apps, complex authorization

---

## 2. MFA (Multi-Factor Authentication)

### **What**: Login requires password + OTP/SMS

### **Implementation Options**:
1. **TOTP** (Google Authenticator) - Best
2. **SMS OTP** - User-friendly
3. **Email OTP** - Fallback
4. **Backup Codes** - Recovery

**Complexity**: Medium  
**Learning Curve**: 1 week  
**Libraries**: `google-authenticator`, Twilio

**When to use**:
- ✅ Admin panels
- ✅ Driver accounts (vehicle access)
- ⚠️ Customer accounts (optional)

---

## 3. Rate Limiting

### **What**: Prevent API abuse (e.g., 100 req/min)

### **Levels**:
1. **API Gateway** (Kong) - Easiest ⭐
2. **Application** (Bucket4j)
3. **Redis-based** (Distributed)

**Complexity**: Low  
**Learning Curve**: 2-3 days  
**When to use**: ALL public APIs

---

## 4. Distributed Tracing

### **What**: Track request across services

**Example Trace**:
```
Trace abc-123 (2.5s total):
├─ API Gateway: 10ms
├─ IAM Service: 50ms
├─ Trip Service: 200ms
│  ├─ Payment: 2000ms ← BOTTLENECK!
│  └─ Driver: 30ms
└─ Notification: 100ms
```

**Complexity**: Very Low (auto-configured)  
**Learning Curve**: 3-4 days  
**Tools**: Zipkin, Jaeger

**When to use**: ALWAYS in microservices

---

## 5. CQRS (Command Query Responsibility Segregation)

### **What**: Separate read and write models

**Example**:
```
Write Model (Complex):
User → Trip → validate → save → publish event

Read Model (Optimized):
Denormalized table for fast queries
trip_history_view: customer_id, trip_id, fare, rating
```

**Complexity**: High  
**Learning Curve**: 2-3 weeks  
**When to use**: High read/write ratio (90% reads)

**Trade-off**:
- ✅ Fast queries
- ❌ Eventual consistency
- ❌ More code

**Recommendation**: Start WITHOUT, add if needed!

---

## 6. Event Sourcing

### **What**: Store events, not state

**Traditional**:
```sql
UPDATE trips SET status = 'COMPLETED';
-- Lost history!
```

**Event Sourcing**:
```
TripCreated → TripAssigned → TripStarted → TripCompleted
-- Full history retained!
```

**Complexity**: VERY HIGH  
**Learning Curve**: 3-4 weeks  
**When to use**: Audit-critical domains (banking)

**Trade-off**:
- ✅ Complete audit trail
- ✅ Time-travel queries
- ❌ Complex
- ❌ Hard to debug

**Recommendation**: SKIP for most apps!  
**Alternative**: Simple audit_log table

---

## 7. Saga Pattern

### **What**: Distributed transactions across services

**Example: Trip Completion**:
```
1. Complete trip (Trip Service)
2. Charge customer (Payment) ← FAILS!
3. Compensation: Revert trip status
```

**Types**:
- **Choreography**: Event-based (easier) ⭐
- **Orchestration**: Centralized control

**Complexity**: Medium-High  
**Learning Curve**: 2 weeks  
**When to use**: REQUIRED for microservices!

---

## 8. Outbox Pattern

### **What**: Guarantee event delivery

**Problem**:
```
1. Save to DB ✅
2. Publish to Kafka ❌ (Kafka down)
→ Data inconsistency!
```

**Solution**:
```
1. Save DB + Outbox (same transaction) ✅
2. Background job publishes from Outbox ✅
→ Guaranteed delivery!
```

**Complexity**: Low-Medium  
**Learning Curve**: 2-3 days  
**When to use**: ALL event-driven services!

---

## 9. Circuit Breaker

### **What**: Prevent cascade failures

**States**:
```
CLOSED → Requests pass through
  ↓ (50% failures)
OPEN → Fail fast (no requests sent)
  ↓ (10 seconds)
HALF_OPEN → Test with 1 request
  ↓ (success)
CLOSED
```

**Complexity**: Very Low  
**Learning Curve**: 1 day  
**Libraries**: Resilience4j (just annotations!)

**When to use**: ALL external calls!

---

## 10. API Gateway

### **What**: Single entry point for all services

**Features**:
- Routing
- Authentication (JWT validation)
- Rate limiting
- Load balancing

**Complexity**: Low  
**Learning Curve**: 2-3 days  
**Tools**: Kong, Spring Cloud Gateway

**When to use**: ALWAYS in microservices!

---

## 📊 Complexity Comparison

| Pattern | Complexity | Learning | Required? | Phase |
|---------|-----------|----------|-----------|-------|
| **DDD Hexagonal** | Medium | Medium | ✅ YES | 3 |
| **CQRS** | High | High | ❌ NO | 7 (optional) |
| **Event Sourcing** | Very High | High | ❌ NO | Skip |
| **Saga** | Medium-High | Medium | ✅ YES | 6 |
| **Outbox** | Low-Medium | Low | ✅ YES | 6 |
| **Circuit Breaker** | Very Low | Very Low | ✅ YES | 8 |
| **API Gateway** | Low | Low | ✅ YES | 7 |
| **Advanced RBAC** | High | Medium-High | ⚠️ MAYBE | 9 |
| **MFA** | Medium | Medium | ⚠️ MAYBE | 10 |
| **Rate Limiting** | Low | Very Low | ✅ YES | 7 |
| **Distributed Tracing** | Very Low | Low | ✅ YES | 10 |

---

## 🎯 Recommendation Summary

### **MUST HAVE** (Production-ready):
1. ✅ DDD Hexagonal
2. ✅ Saga + Outbox
3. ✅ Circuit Breaker
4. ✅ API Gateway
5. ✅ Rate Limiting
6. ✅ Distributed Tracing

### **SHOULD HAVE** (Security):
7. ✅ MFA (for sensitive accounts)
8. ⚠️ Advanced RBAC (if complex permissions)

### **NICE TO HAVE** (Optional):
9. ⚠️ CQRS (if high read/write ratio)

### **SKIP** (Overkill):
10. ❌ Event Sourcing (use audit logs instead)

---

## 💡 Decision Framework

**Use CQRS when**:
- Read/write ratio > 80/20
- Complex queries
- OK with eventual consistency

**Use Event Sourcing when**:
- Banking/finance domain
- Required by compliance
- Budget for complexity

**Use Advanced RBAC when**:
- Multi-tenant app
- Complex hierarchies
- Dynamic permissions

**Use MFA when**:
- High-security requirements
- Admin access
- PCI/HIPAA compliance

---

## 📈 Learning Investment

**Total learning time (full stack)**:
- DDD + Hexagonal: 2-3 weeks
- Event-Driven (Saga + Outbox): 2 weeks
- Resilience (Circuit Breaker): 3 days
- Security (JWT + RBAC): 1 week
- MFA: 1 week
- CQRS: 2 weeks
- Observability: 1 week
- Advanced RBAC: 2 weeks

**Total**: ~12-14 weeks to master all patterns

**Recommended pace**: 1-2 patterns per week

---

**Remember**: Start simple, add complexity only when needed! 🚀
