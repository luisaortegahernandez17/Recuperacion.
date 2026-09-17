### ADR-001 — Backend Development Framework and Language

| Field | Value |
|-------|-------|
| **ID** | ADR-001 |
| **Date** | 2026-16-09 |
| **Status** | Accepted |
| **Authors** | Manuel Caviedes Cordero — Tech Lead |
| **Reviewers** | Natalia Trujillo Santofimio, Juan Oteca Pedreros, Luisa Ortega Hernandez — Development team |


## Context

The Huila Travel Expedition platform aims to consolidate the tourism offerings of the Huila department, integrating local travel agencies, the regional administration, and tourists. A robust, rapidly developable backend is required to ensure software maintainability, support API-based authentication, manage complex data relationships (plans, agencies, municipalities, reservations), and include an efficient ORM for relational queries.

**Known Constraints:**

- Limited development timeframes for the MVP phase.
- General response time requirement RNF1 (< 2 seconds).
- High availability of local technical talent with proficiency in the selected language.

## Decision

**We decided:** To adopt **Laravel (PHP 8.2+)** as the standard framework for developing the Huila Travel Expedition backend REST API.

**Justification:**
Laravel offers a mature out-of-the-box ecosystem (Eloquent ORM, migrations, queues, scheduled tasks, and API authentication) that accelerates time to market without sacrificing quality or security in the management of project modules. The tiebreaker was the ORM development speed and native handling of the business's relational schema.


## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|---|---|---|---|
| **Option A — Laravel 10/11 (PHP 8.2+)** | Powerful Eloquent ORM, native migration handling, integrated queues, and Sanctum/Passport ecosystem. | Lower throughput in requests per second compared to compiled languages. | — (chosen) |
| **Option B — Node.js (NestJS / TypeScript)** | Excellent handling of asynchronous I/O and high real-time performance. | Longer initial setup time and learning curve for modular architecture compared to Laravel. | Longer initial development time required to configure the ORM layer and migrations. |
| **Option C — Python (Django)** | Very powerful native admin panel and robust ORM. | Less flexibility for fine-coupling decoupled REST APIs without complex additional modules. | Less agility in structuring the catalog-oriented REST API and project reservations. |

## Consequences

**Positive:**
- Significant reduction in MVP delivery time.
- Easy to build automated tests using PHPUnit/Pest.
- Structured management of the relational database schema through code migrations.

**Negatives / Trade-offs:**
- Requires the use of caching layers (Redis) to optimize high-concurrency queries in the public destination catalog.

**Impact on the system:**
- Affected services: All backend modules (auth-module, catalog-module, booking-module).
- Documents that must be updated: overview.md, Development Environment Configuration Guide.


## Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Performance degradation under high synchronous load in the catalog | Medium | High | Implement Redis for caching frequently queried data from the plans and municipalities database. |


## References

- Official Laravel Documentation: https://laravel.com/docs
- Related to: ADR-002, ADR-003, ADR-004

---

### ADR-002 — Stateless Authentication and Role-Based Access Control (RBAC) Strategy

| Field | Value |
|-------|-------|
| **ID** | ADR-002 |
| **Date** | 2026-16-09 |
| **Status** | Accepted |
| **Authors** | Manuel Caviedes Cordero — Tech Lead |
| **Reviewers** | Natalia Trujillo Santofimio, Juan Oteca Pedreros, Luisa Ortega Hernandez — Development team |

## Context

The platform manages multiple user profiles with strictly differentiated permissions: Administrator (approval of agencies with RNT), Travel Agency (management of plans and availability), and Tourist (search and booking requests). It is required to secure credentials through encryption (RNF6) and restrict access according to specific roles (RF2, RF16).

**Known constraints:**
- Strict compliance with the RNF6 security requirement (secure encryption using a Hash algorithm).
- Need for a decoupled stateless scheme compatible with web clients and future mobile applications.


## Decision

**We decided:** Implement stateless token-based authentication using **Laravel Sanctum** combined with a **Role-Based Access Control (RBAC)** scheme managed through Laravel Middleware and Policies.

**Justification:** Laravel Sanctum provides a lightweight authentication layer based on Token APIs, natively integrating with the Bcrypt hashing required by RNF6 and allowing transparent isolation of resource access based on the authenticated user's role.


## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|---|---|---|---|
| **Option A — Laravel Sanctum (Token API) + RBAC** | Lightweight, stateless, native, ideal for SPA web interfaces and future mobile apps. | Requires manual token revocation management for invalidated sessions. | — (chosen) |
| **Option B — Traditional Sessions (Cookies / Stateful)** | Easy implementation on monolithic server-rendered architectures. | Incompatible with decoupled mobile clients and pure REST API services. | Discarded due to strict coupling to the web browser. |
| **Option C — Auth0 / Firebase Auth** | Complete delegation of security to a third-party identity provider. | Scalable operating costs and difficulty in linking local verification states (RNT). | Discarded due to external financial dependency and poor integration with RNT verification. |


## Consequences

**Positive:**
- Compliance with security standards (RNF6) through native password encryption (Bcrypt).
- Strict resource isolation: an agency can only manipulate the plans it owns (agency_id).

**Negative / Trade-offs:**
- Requires maintaining a revocation list of active tokens when an Administrator deactivates an agency (RF2).

**Impact on the system:**
- Affected services: auth-module, REST API security middleware.
- Documents that must be updated: Endpoint documentation and REST API Permissions Matrix.


## Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Bearer token leakage or interception in network requests | Low | High | Require mandatory HTTPS protocol for all API requests and configure token expiration. |

## References

- Laravel Sanctum documentation: https://laravel.com/docs/sanctum
- Related to: ADR-001

---

### ADR-003 — Optimized Processing and Storage of Images for Tourism Services

| Field | Value |
|-------|-------|
| **ID** | ADR-003 |
| **Date** | 2026-16-09 |
| **Status** | Accepted |
| **Authors** | Manuel Caviedes Cordero — Tech Lead |
| **Reviewers** | Natalia Trujillo Santofimio, Juan Oteca Pedreros, Luisa Ortega Hernandez — Development team |


## Context

Travel agencies publish photographs to promote their destinations and tourism packages. According to the non-functional requirement **RNF2**, no uploaded image should exceed 500 KB to avoid performance issues, reduce bandwidth consumption, and optimize loading times on mobile devices with slow connections.

**Known constraints:**
- Strict maximum file size limit per processed image: 500 KB (RNF2).
- Heterogeneous image formats uploaded by users (JPG, PNG, WEBP).


## Decision

**We decided:** Implement an initial validation pipeline in the API that asynchronously compresses and converts received images to the **.webp** format, ensuring a file size of less than 500 KB before persisting them.

**Justification:** Server-side conversion to the .webp format guarantees the preservation of the visual quality necessary for tourism marketing while strictly adhering to the non-functional file size requirement (RNF2).


## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|---|---|---|---|
| **Option A — Server Compression (.webp < 500KB)** | Ensures RNF2 compliance regardless of the user's upload size. | CPU/Memory consumption on the server during the conversion process. | — (chosen) |
| **Option B — Client-side compression only (JS)** | Reduces network traffic during upload to the server. | Depends on the user's browser and does not guarantee the integrity of the uploaded file. | Discarded due to lack of direct control over the business rule (RNF2). |
| **Option C — Direct storage without compression** | Instant upload process on the backend. | Degrades website loading speed and saturates storage. | Discarded due to direct violation of the RNF2 requirement. |

## Consequences

**Positive:**
- Guaranteed compliance with the RNF2 performance requirement.
- Ultra-fast loading times in the public catalog of tourist destinations.

**Negative / Trade-offs:**
- Requires the installation and maintenance of the GD or Imagick graphics rendering libraries in the PHP runtime environment.

**Impact on the system:**
- Affected services: catalog-module, File Storage Service.

- Documents that must be updated: Server Installation Requirements and File Upload Specification.


## Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Server processor saturation due to simultaneous bulk image uploads | Low | Medium | Processing graphic conversion using background tasks (*Queue Jobs*) in Laravel. |


## References

- Web Image Optimization Guide: https://web.dev/fast/compress-images
- Related to: ADR-001

---

### ADR-004 — Booking Engine and Availability Management using Pessimistic Transactions

| Field | Value |
|-------|-------|
| **ID** | ADR-004 |
| **Date** | 2026-16-09 |
| **Status** | Accepted |
| **Authors** | Manuel Caviedes Cordero — Tech Lead |
| **Reviewers** | Natalia Trujillo Santofimio, Juan Oteca Pedreros, Luisa Ortega Hernandez — Development team |

## Context

The booking engine processes requests for slots on specific dates for tour packages offered by agencies. It is critical to avoid overbooking in scenarios where multiple tourists try to reserve the last available slots for the same date at the same time.

**Known constraints:**
- Strictly adhere to the slots declared in the calendar by the agencies (RF9, RF10).
- Ensure ACID consistency of the data in the MySQL relational database.


## Decision

**We decided:** To use **ACID transactions with Pessimistic Lock (lockForUpdate())** at the relational database level for processing the reservation request flow.

**Justification:**
Pessimistic locking ensures that only one transaction at a time can read and decrement the availability of a specific calendar, eliminating any possibility of a race condition or overbooking.


## Evaluated alternatives

| Alternative | Pros | Cons | Reason for discarding |
|---|---|---|---|
| **Option A — Pessimistic Lock (SELECT FOR UPDATE)** | Ensures absolute atomicity and zero overbooking of slots. | Temporarily locks the record in the database for the duration of the transaction. | — (chosen) |
| **Option B — Simple verification without locking** | High read and write speed. | Prone to race conditions under high concurrency, generating overbooking. | Discarded due to the risk of data inconsistency. |
| **Option C — Distributed locks in Redis** | Extremely fast in memory. | Increases operational complexity to keep Redis synchronized with MySQL. | Discarded due to unnecessary complexity for the initial scale of the project. |

## Consequences

**Positive:**
- Complete elimination of the risk of overbooking.
- Guaranteed consistency in confirming and controlling the status of reservations.

**Negative / Trade-offs:**
- Slight increase in wait time if exact simultaneous collisions occur for the same date and plan.

**Impact on the system:**
- Affected services: booking-module, Reservation Repository, and Calendar.
- Documents that must be updated: Reservation Process Sequence Diagram, Reservations Module Specification.


## Risks

| Risk | Probability | Impact | Mitigation |
|---|---|---|---|
| Deadlocks in concurrent transactions | Low | High | Keep transactions as short as possible and define a strict timeout for database queries. |



## References

- Laravel Transactions and Locking Documentation: https://laravel.com/docs/database#database-transactions
- Related to: ADR-001

