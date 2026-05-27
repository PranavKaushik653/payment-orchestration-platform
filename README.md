# Payment Orchestration Platform

A production-inspired payment orchestration platform built using Java and Spring Boot.

The system accepts merchant payment requests, routes transactions to different payment providers based on payment method, supports retries and failover handling, and guarantees idempotent payment execution using Redis.

---

# Features

- Payment routing engine
- Provider failover handling
- Retry and circuit breaker support
- Redis-backed idempotency
- PostgreSQL persistence
- RESTful APIs
- OpenAPI / Swagger documentation
- Prometheus metrics support
- Integration + unit testing
- Production-inspired modular architecture

---

# Tech Stack

| Layer | Technology |
|---|---|
| Language | Java 21 |
| Framework | Spring Boot 3 |
| Database | PostgreSQL 16 |
| Cache / Idempotency Store | Redis 7 |
| Build Tool | Maven 3.9+ |
| Resilience | Resilience4j |
| API Docs | SpringDoc OpenAPI |
| Metrics | Micrometer + Prometheus |

---

# System Workflow

```text
Merchant Request
      ↓
Payment Controller
      ↓
Idempotency Validation (Redis)
      ↓
Routing Engine
      ↓
Provider Connector
      ↓
Retry + Circuit Breaker Logic
      ↓
Persist Result (PostgreSQL)
      ↓
Return Response
```

---

# Prerequisites

Install the following before running the application.

## Java 21

```bash
java -version
```

Expected:

```bash
openjdk 21
```

Download:
https://adoptium.net/

---

## Maven 3.9+

```bash
mvn -version
```

Download:
https://maven.apache.org/download.cgi

---

## Docker

```bash
docker --version
```

Download:
https://docs.docker.com/get-docker/

---

# Running the Application

## Step 1 — Start PostgreSQL and Redis

```bash
docker run -d \
  --name payment-postgres \
  -e POSTGRES_DB=payment_orchestrator \
  -e POSTGRES_USER=app_user \
  -e POSTGRES_PASSWORD=app_secret \
  -p 5432:5432 \
  postgres:16-alpine

docker run -d \
  --name payment-redis \
  -p 6379:6379 \
  redis:7-alpine
```

Verify containers:

```bash
docker ps
```

---

## Step 2 — Clone the Repository

```bash
git clone https://github.com/YOUR_USERNAME/payment-orchestration-platform.git

cd payment-orchestration-platform
```

---

## Step 3 — Build the Project

```bash
mvn clean package -DskipTests
```

Expected:

```bash
BUILD SUCCESS
```

---

## Step 4 — Run the Application

```bash
mvn spring-boot:run
```

Application starts at:

```text
http://localhost:8080
```

---

# Health Check

```bash
curl http://localhost:8080/actuator/health
```

Expected response:

```json
{
  "status": "UP"
}
```

---

# API Usage

# Create CARD Payment

```bash
curl -X POST http://localhost:8080/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: payment-key-001" \
  -d '{
    "merchantId": "merchant-001",
    "amount": 150.00,
    "currency": "USD",
    "paymentMethod": "CARD"
  }'
```

Example response:

```json
{
  "paymentId": "a1b2c3d4",
  "status": "SUCCESS",
  "assignedProvider": "PROVIDER_A",
  "providerTransactionId": "PA-XXXXXXXX",
  "merchantId": "merchant-001",
  "amount": 150.00,
  "currency": "USD",
  "paymentMethod": "CARD"
}
```

---

# Create UPI Payment

```bash
curl -X POST http://localhost:8080/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: payment-key-002" \
  -d '{
    "merchantId": "merchant-002",
    "amount": 500.00,
    "currency": "INR",
    "paymentMethod": "UPI"
  }'
```

---

# Fetch Payment By ID

```bash
curl http://localhost:8080/api/v1/payments/{paymentId}
```

---

# Idempotency Demo

Run this command twice using the same idempotency key.

```bash
curl -X POST http://localhost:8080/api/v1/payments \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: idempotency-demo-key" \
  -d '{
    "merchantId": "merchant-demo",
    "amount": 200.00,
    "currency": "USD",
    "paymentMethod": "CARD"
  }'
```

Both responses return the same `paymentId`, proving duplicate payments are prevented.

---

# Swagger API Documentation

Open in browser:

```text
http://localhost:8080/swagger-ui/index.html
```

Swagger provides:
- interactive API testing
- request/response schemas
- endpoint documentation

---

# Running Tests

```bash
mvn test
```

Expected:

```text
BUILD SUCCESS
```

---

# Test Coverage

| Suite | Purpose |
|---|---|
| Integration Tests | End-to-end payment flow |
| Unit Tests | Routing and orchestration logic |
| Negative Tests | Validation and failure scenarios |
| Repository Tests | Database persistence validation |

---

# Project Structure

```text
src/main/java/com/payments/orchestrator/
├── controller/
│   └── PaymentController.java
├── service/
│   └── PaymentOrchestrationService.java
├── routing/
│   └── RoutingEngine.java
├── provider/
│   └── ProviderConnectors.java
├── idempotency/
│   └── IdempotencyStore.java
├── model/
│   ├── Payment.java
│   └── PaymentDTOs.java
├── repository/
│   └── PaymentRepository.java
└── exception/
    ├── PaymentExceptions.java
    └── GlobalExceptionHandler.java
```

---

# Architecture Overview

```text
Client
  ↓
PaymentController
  ↓
PaymentOrchestrationService
  ↓
Redis Idempotency Validation
  ↓
RoutingEngine
  ↓
Provider Connector
  ↓
Retry + Circuit Breaker
  ↓
PostgreSQL Persistence
  ↓
API Response
```

---

# Reliability Features

| Feature | Purpose |
|---|---|
| Idempotency | Prevent duplicate payment execution |
| Retry Logic | Recover from temporary provider failures |
| Circuit Breakers | Prevent cascading failures |
| Provider Failover | Switch provider if one becomes unavailable |
| Stateless Design | Easier horizontal scaling |
| Validation Layer | Reject invalid requests early |

---

# Planned Improvements

## Scalability

- Kafka-based asynchronous processing
- Distributed locking
- Read replicas
- Horizontal autoscaling
- Event-driven workflows

## Observability

- OpenTelemetry tracing
- Grafana dashboards
- Centralized logging
- Correlation IDs

## Cloud-Native Deployment

- Docker Compose
- Kubernetes deployment
- Rolling updates
- Externalized configuration

---

# Planned Async Architecture

```text
Merchant Request
      ↓
Save Payment as PENDING
      ↓
Publish Kafka Event
      ↓
Immediate API Response

Background Worker
      ↓
Provider Processing
      ↓
Update Payment Status
      ↓
Webhook Notification
```

---

# Troubleshooting

## PostgreSQL or Redis Connection Refused

Check running containers:

```bash
docker ps
```

Restart containers if necessary.

---

## Build Failure

Ensure Java 21 is installed:

```bash
java -version
```

---

## Port 8080 Already In Use

Run on another port:

```bash
mvn spring-boot:run \
  -Dspring-boot.run.arguments=--server.port=9090
```

---

# Architectural Philosophy

This project intentionally follows a modular monolith architecture before introducing distributed-system complexity.

The goal is to:
- simplify development
- improve maintainability
- reduce operational overhead
- preserve transactional consistency

The internal modules are designed with clean boundaries so they can later evolve into independently deployable microservices if required.

---

# Future Goals

- Multi-provider smart routing
- Dynamic provider health scoring
- Real payment gateway integrations
- Async settlement workflows
- Fraud detection hooks
- Merchant webhook infrastructure
- Distributed tracing support

---

# License

This project is intended for educational and portfolio purposes.
