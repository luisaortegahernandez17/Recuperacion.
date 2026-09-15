# Non-Functional Requirements (NFR) — Huila Travel Expedition (HTE)

> NFRs define the **qualities of the system** — not what it does but how well it does it.
> The golden rule: every NFR must have a metric.

---

## NFR-001: Performance

| Attribute | Metric | Test conditions |
|-----------|--------|---------------|
| P95 latency — critical endpoints | <300ms | Under 300 RPS load |
| P99 latency — critical endpoints | <500ms | Under 300 RPS load |
| P95 latency — non-critical endpoints | < 1000ms | Under 100 RPS normal load |
| Minimum throughput | 300RPS | Without degradation across services |
| Service startup time | < 15 seconds | Cold start in containerized environment |

**Defined critical endpoints:**
- `POST /api/v1/bookings` — Critical for concurrency and real-time booking with `inventory-service`[cite: 3, 5].

- `GET /api/v1/plans/search` — Critical as it is the main multi-criteria search query in `catalog-service` and for tourist spending[cite: 3, 5].

- `POST /api/v1/auth/login` — Critical for the JWT token signing and validation process in `auth-service`[cite: 5].

**Load testing tools:**
- Kubernetes 6, Postman / Newman CLI

**Where is it validated?** CI/CD in the staging pipeline before production.

----

## NFR-002: Availability

| Environment | SLO | Maintenance window | Max downtime/month |

|---------|-----|-------------------|-------------------|
| Production | 99.9% | Sundays 2am-4am | 44 minutes |
| Staging | 95% | No restriction | 36 hours |

**Monthly error budget in production:** 44 minutes
**Error Budget policy:** If > 50% of the error budget is consumed in the first half of the month, feature deploys are frozen until the next month and stability is prioritized.

**Health checks:**
- `GET /health` — Liveness: responds 200 if the microservice process is alive.
- `GET /health/ready` — Readiness: responds 200 only if the microservice can process traffic (PostgreSQL/MongoDB connected, Message Broker OK).

---

## NFR-003: Scalability

| Scenario | Expected behavior |
|---------|--------|
| Gradual load growth | Horizontal auto-scaling (HPA) activated when CPU > 70% or RAM > 80% |
| Sudden spike (High tourist season in Huila) | System scales in < 2 minutes |
| Load reduction | Scale-down without interrupting active transactions |
| Horizontal scaling limit | Up to 5 instances per microservice (`booking-service`, `catalog-service`) |

**Strategy:** Stateless horizontal scaling — HTE microservices do not store sessions in local memory. State is managed using JWT tokens and database persistence.

---

## NFR-004: Security

### Authentication and Authorization
- All private endpoints require a valid JWT in the `Authorization: Bearer <token>` header[cite: 5].
- JWT tokens expire in **30 minutes**[cite: 5].
- Refresh tokens valid for **7 days**.
- RBAC (Role-Based Access Control): defined roles (`ROLE_ADMIN`, `ROLE_AGENCY`, `ROLE_USER`)[cite: 5].

### Data transmission
- HTTPS mandatory in production (TLS 1.2+).
- HTTP only in local development.

### Sensitive data
- Passwords: hashing with BCrypt (cost factor = 12)[cite: 5].
- PII (personal data) & RNT/NIT data: encrypted at rest.
- Secrets/keys: managed strictly via environment variables (`.env`) or Kubernetes Secrets, **never hardcoded in code**.

### OWASP Top 10
Code must be reviewed against OWASP Top 10 on each release.
Tools: SonarQube / Snyk for SAST and dependency vulnerability scanning.

### Regulatory compliance
- **Law 1581 of 2012 (Habeas Data - Colombia):** Protection of personal data of tourists and agency representatives.

---

## NFR-005: Observability

| Pillar | Requirement | Tools |
|--------|------------|------|
| Logs | Structured JSON format + Correlation ID | Winston/Logback |
| Metrics | RED (Rate, Errors, Duration) per endpoint | Prometheus + Grafana |
| Traces | End-to-end distributed traces between microservices | OpenTelemetry / Jaeger |
| Alerts | Alert in < 5 min when SLI violates SLO | Alertmanager / Discord Webhook |

**Correlation ID:** Each external request generates a UUID `correlationId` propagated in HTTP headers (`X-Correlation-ID`) across all microservices and transaction logs.

---

## NFR-006: Maintainability

| Metric | Target |
|--------|--------|
| Test coverage | ≥ 80% of lines (≥ 90% in core domain logic)[cite: 3] |
| Cyclomatic complexity | ≤ 10 per function |
| Technical debt | Resolution time < 1 sprint from registration |
| Onboarding time | A new dev can deploy HTE locally in < 1 hour following `10-devops/local-setup.md` |
| Average build time | < 5 minutes in CI pipeline |

---

## NFR-007: Portability

- All microservices of HTE are deployed as Docker containers.
- Images work in any environment with Kubernetes 1.28+ or Docker Compose.
- No microservice depends on the host operating system.
- Environment variables are the only source of environment-specific configuration.

---

## NFR-008: Disaster Recovery (DR / Recovery)

| Scenario | RTO (Recovery Time Objective) | RPO (Recovery Point Objective) |
|---------|----------------------------|----------------------------|
| Single service failure | < 2 minutes (K8s restart / Docker Auto-restart) | 0 (stateless) |
| Primary database failure | < 5 minutes (failover to replica) | < 1 second (synchronous replication) |
| Availability zone loss | < 15 minutes | <5 minutes |
| Full region disaster | < 4 hours (DR backup restore) | < 1 hour |

---

## NFR priority matrix

| NFR | Priority (P1/P2/P3) | Validated in CI? | Owner |
|-----|---------------------|-----------------|-------|
| Performance | P1 | Yes (k6 in staging) | Tech Lead |
| Availability | P1 | Yes (health checks) | DevOps |
| Security | P1 | Yes (SAST + SonarQube) | Security / Backend Lead |
| Scalability | P2 | Manual (per sprint) | DevOps |
| Observability | P1 | Yes (smoke test in CI) | Tech Lead |
| Maintainability | P2 | Yes (coverage in CI) | Development Team |

---

## Correlations

- Detailed SLOs and SLAs → `13-operations/README.md`
- Pipeline that validates NFRs → `10-devops/README.md`
- Incidents related to NFR violations → `13-operations/incident-management.md`
- Security checklist → `00-governance/security-policy.md`
