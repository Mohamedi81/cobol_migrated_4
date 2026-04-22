# Target Architecture – Java Microservices for CICS GenApp

## 1. Overall Architecture

The migrated system is a set of Spring Boot–based microservices exposing GenApp’s business capabilities via REST APIs. The original CICS programs and SOA copybooks are mapped into HTTP endpoints and JSON payloads.

- **Pattern:** Domain-aligned microservices (bounded contexts: Customer, Policy, Account, Stats, Web-STS, API Gateway).
- **Tech stack:**
  - Java 17+
  - Spring Boot 3.x (Web, Data JPA, Validation, Actuator)
  - Maven multi-module project (one module per microservice + shared model)
  - Database: relational (e.g. PostgreSQL) with JPA entities; initial implementation can use in-memory H2.
- **Communication:**
  - External: REST/JSON
  - Internal: REST/JSON (optionally async later).
- **Deployment:** Container-first (Docker), suitable for Kubernetes.

## 2. Microservice Landscape

### 2.1 Services

1. **customer-service**
   - Owns Customer master data and customer-facing operations.
2. **policy-service**
   - Owns Policy data and operations (create/update/inquire policy).
3. **account-service**
   - Owns account/financial aspects related to customers and policies.
4. **stats-service**
   - Computes and exposes statistics and batch-like operations.
5. **web-sts-service**
   - Maps legacy `lgwebst5` / SOA copybooks to REST APIs.
6. **api-gateway**
   - Single entry point for external consumers.
7. **common-model**
   - Shared DTOs, error models and utilities.

### 2.2 Boundaries and COBOL Mapping (Indicative)

- **Customer Service** (`customer-service`)
  - COBOL: `lgicdb01.cbl`, `lgicus01.cbl`, `lgicvs01.cbl`, `lgucdb01.cbl`, `lgucus01.cbl`, `lgucvs01.cbl`, `lgacus01.cbl`, `lgcmarea.cpy`.
  - Responsibilities: create/update/retrieve customers; search/list customers; provide customer details to other services.

- **Policy Service** (`policy-service`)
  - COBOL: `lgapdb01.cbl`, `lgapol01.cbl`, `lgapvs01.cbl`, `lgdpdb01.cbl`, `lgdpol01.cbl`, `lgdpvs01.cbl`, `lgipdb01.cbl`, `lgipol01.cbl`, `lgipvs01.cbl`, `lgupdb01.cbl`, `lgupol01.cbl`, `lgupvs01.cbl`, `lgpolicy.cpy`, `pollook.cpy`, `polloo2.cpy`.
  - Responsibilities: quote/create/update/cancel policies; policy inquiries and list views; coordinate with customer-service.

- **Account Service** (`account-service`)
  - COBOL: `lgacdb01.cbl`, `lgacdb02.cbl`, `lgacvs01.cbl`, `lgacus01.cbl`.
  - Responsibilities: manage accounts/payments associated with policies/customers; balances and transactions.

- **Stats / Admin Service** (`stats-service`)
  - COBOL: `lgstsq.cbl`, `lgastat1.cbl`, aspects of `lgsetup.cbl`.
  - Responsibilities: compute system statistics; batch-style operations and rebuilds.

- **Web-STS Integration Service** (`web-sts-service`)
  - COBOL: `lgwebst5.cbl`, `soaic01.cpy`, `soaipb1.cpy`, `soaipe1.cpy`, `soaiph1.cpy`, `soaipm1.cpy`, `soavcii.cpy`, `soavcio.cpy`, `soavpii.cpy`, `soavpio.cpy`.
  - Responsibilities: expose a REST endpoint that accepts payloads shaped like the SOA copybooks and dispatches to customer/policy services.

- **API Gateway** (`api-gateway`)
  - Responsibilities: route `/customers/**`, `/policies/**`, `/accounts/**`, `/stats/**`, `/sts/**` to respective services; handle cross-cutting concerns such as logging and correlation IDs.

## 3. Logical Architecture (Text)

```
          +------------------------+
          |      API Gateway       |
          | (Spring Cloud Gateway) |
          +----------+-------------+
                     |
       +-------------+-------------+
       |             |             |
 /customers/**  /policies/**   /accounts/**  /stats/**  /sts/**
       |             |             |            |         |
+------+-----+ +-----+------+ +----+------+ +---+--------+ +--------------+
| Customer  | | Policy     | | Account   | | Statistics | | Web-STS      |
| Service   | | Service    | | Service   | | Service    | | Service      |
+------+-----+ +-----+------+ +----+------+ +---+--------+ +--------------+
       |             |             |            |
       +-------------+-------------+------------+
                     |
             +-------+--------+
             |  Relational DB |
             | (logical view) |
             +----------------+
```

## 4. Data Model (High-Level)

Entities are persisted using JPA; DTOs live in `common-model`.

### 4.1 Core Entities

- `Customer`
  - `id` (UUID/long), `name`, `dateOfBirth`, `address`, `email`, `phone`, `createdAt`, `updatedAt`.
- `Policy`
  - `id`, `customerId`, `policyNumber`, `productCode`, `status`, `effectiveDate`, `expiryDate`, `premiumAmount`, `coverageDetails`.
- `Account`
  - `id`, `customerId`, `balance`, `status`.
- `AccountTransaction`
  - `id`, `accountId`, `policyId` (optional), `amount`, `type`, `timestamp`.
- `SystemStats`
  - Derived values (counts of policies, customers, etc.).

### 4.2 DTOs (Copybook Mapping)

- `PolicyDto` – from `lgpolicy.cpy`.
- `PolicySummaryDto` – from `pollook.cpy`, `polloo2.cpy`.
- `CustomerDto` – aligned with `lgcmarea.cpy` customer and VSAM layouts.
- `WebStsRequestDto` / `WebStsResponseDto` – from `soa*.cpy`, `soav*.cpy`.

## 5. REST APIs (Service Interfaces)

### 5.1 Customer Service – `/customers`

- `POST /customers` – create a new customer.
- `GET /customers/{id}` – retrieve a customer.
- `PUT /customers/{id}` – update a customer.
- `GET /customers` – list/search customers.

### 5.2 Policy Service – `/policies`

- `POST /policies` – create a new policy.
- `GET /policies/{id}` – retrieve policy details.
- `PUT /policies/{id}` – update policy.
- `GET /policies` – search/list policies.
- `GET /policies/customer/{customerId}` – policies for a customer.

### 5.3 Account Service – `/accounts`

- `POST /accounts` – create account.
- `GET /accounts/{id}` – retrieve account.
- `PUT /accounts/{id}` – update account.
- `GET /accounts/customer/{customerId}` – accounts for a customer.
- `POST /accounts/{id}/transactions` – add transaction.

### 5.4 Stats Service – `/stats`

- `GET /stats/system` – current `SystemStats`.
- `POST /stats/rebuild` – trigger stats rebuild.

### 5.5 Web-STS Service – `/sts`

- `POST /sts/operation` – accept SOA-shaped request, call underlying services, return structured response.

## 6. Internal Structure per Service

Each microservice follows:

- **API layer** – Spring MVC controllers.
- **Application layer** – services/orchestration.
- **Domain layer** – entities and business rules.
- **Persistence layer** – Spring Data JPA repositories.

Example for `policy-service`:

```
policy-service
 ├─ src/main/java/com/example/policy
 │   ├─ api
 │   │   └─ PolicyController.java
 │   ├─ application
 │   │   └─ PolicyService.java
 │   ├─ domain
 │   │   ├─ Policy.java
 │   │   └─ PolicyStatus.java
 │   └─ infrastructure
 │       ├─ repository
 │       │   └─ PolicyRepository.java
 │       └─ client
 │           └─ CustomerClient.java
 └─ src/main/resources
     └─ application.yml
```

## 7. Cross-Cutting Concerns

- Spring profiles (`dev`, `test`, `prod`).
- Logging with SLF4J + Logback, structured logs, correlation IDs propagated from the gateway.
- Global error handling returning consistent error payloads.
- Bean validation (`@Valid`) for request DTOs.
- Security placeholder for later (JWT/OAuth2 at the gateway).

## 8. Repository Structure (Maven Multi-Module)

Intended structure for `Cobol_migrated_4`:

```
Cobol_migrated_4/
 ├─ pom.xml                    (parent; packaging=pom)
 ├─ docs/
 │   ├─ original-system-overview.md
 │   ├─ target-architecture.md
 │   └─ mapping-cobol-to-microservices.md
 ├─ common-model/
 │   └─ pom.xml
 ├─ customer-service/
 │   └─ pom.xml
 ├─ policy-service/
 │   └─ pom.xml
 ├─ account-service/
 │   └─ pom.xml
 ├─ stats-service/
 │   └─ pom.xml
 ├─ web-sts-service/
 │   └─ pom.xml
 ├─ api-gateway/
 │   └─ pom.xml
 └─ .github/
     └─ workflows/
         └─ ci.yml
```

This architecture is the blueprint for Jira tasks and the subsequent microservice implementation.
