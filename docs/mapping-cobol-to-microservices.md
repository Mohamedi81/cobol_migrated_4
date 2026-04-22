# Mapping COBOL to Java Microservices

This document links the original CICS GenApp COBOL artefacts to the target Java microservices.

## Programs → Services/Endpoints (high level)

- Customer-related (e.g. `lgicdb01.cbl`, `lgicus01.cbl`, `lgicvs01.cbl`, `lgucdb01.cbl`, `lgucus01.cbl`, `lgucvs01.cbl`, `lgacus01.cbl`)
  - → **customer-service**
  - → Endpoints under `/customers` (CRUD, search/list).

- Policy-related (e.g. `lgapdb01.cbl`, `lgapol01.cbl`, `lgapvs01.cbl`, `lgdpdb01.cbl`, `lgdpol01.cbl`, `lgdpvs01.cbl`, `lgipdb01.cbl`, `lgipol01.cbl`, `lgipvs01.cbl`, `lgupdb01.cbl`, `lgupol01.cbl`, `lgupvs01.cbl`)
  - → **policy-service**
  - → Endpoints under `/policies` (CRUD, search, customer-linked lists).

- Account-related (e.g. `lgacdb01.cbl`, `lgacdb02.cbl`, `lgacvs01.cbl`, `lgacus01.cbl`)
  - → **account-service**
  - → Endpoints under `/accounts` for accounts and `/accounts/{id}/transactions`.

- Statistics/setup (e.g. `lgstsq.cbl`, `lgastat1.cbl`, `lgsetup.cbl`)
  - → **stats-service** for statistics endpoints (`/stats/system`, `/stats/rebuild`).
  - → Setup responsibilities may be modelled as separate admin or migration tools.

- Web/STS integration (`lgwebst5.cbl`, `soa*.cpy`, `soav*.cpy`)
  - → **web-sts-service**
  - → Endpoint `/sts/operation` for handling legacy web-style requests.

## Copybooks → Java Types

- `lgcmarea.cpy`
  - → Shared request/response context classes in **common-model**.

- `lgpolicy.cpy`
  - → `Policy` entity (JPA) in **policy-service** and `PolicyDto` in **common-model**.

- `pollook.cpy`, `polloo2.cpy`
  - → `PolicySummaryDto` in **common-model** and relevant query models.

- `soaic01.cpy`, `soaipb1.cpy`, `soaipe1.cpy`, `soaiph1.cpy`, `soaipm1.cpy`, `soavcii.cpy`, `soavcio.cpy`, `soavpii.cpy`, `soavpio.cpy`
  - → `WebStsRequestDto`, `WebStsResponseDto` and related supporting DTOs in **common-model** and used by **web-sts-service**.

## BMS Maps → Web UI / API

- `ssmap.bms`
  - Original 3270 screen layout; in the Java microservices world this is replaced by REST APIs that a modern UI (e.g. web or SPA) can call.

## Data Stores → Persistence

- VSAM `ksdscust` → `Customer` table (e.g., `customers`) managed by **customer-service**.
- VSAM `ksdspoly` → `Policy` table (e.g., `policies`) managed by **policy-service**.

These mappings drive the creation of Jira tasks and Java implementations.
