# Traceability Matrix — Huila Travel Expedition (HTE)

> Traceability connects every line of code to its business justification.
> It allows answering: "Why does this function exist?" and "Which HU covers this part of the system?"
> It also identifies: unimplemented requirements and code without a requirement (possible technical debt).

---

## How to use this matrix
# FR → HU → Test → Service matrix

| FR ID | FR Description | HU(s) | Tests that verify it | Service | Status |

-------|---------------|-------|---------------------|---------|--------|

| FR-001 | Legal registration and validation of travel agencies | `HU-AUTH-001`<br>`HU-ADMIN-002` | `agency.register.spec.ts`<br>`agency.approval.spec.ts` | auth-service<br>admin-service | 🟡 In progress |

| FR-002 | Authentication, secure login, and password recovery | `HU-AUTH-002`<br>`HU-AUTH-003` | `auth.login.spec.ts`<br>`auth.reset-password.spec.ts` | auth-service | 🟡 In progress |

FR-003 | Management and information of the corporate profile of agencies | `HU-AGENCY-001` | `agency.profile.spec.ts` | agency-service | 🔴 Pending |

FR-004 | Creation, editing, and temporary disabling of tourist plans | `HU-CATALOG-001`<br>`HU-CATALOG-003` | `catalog.crud.spec.ts`<br>`catalog.status.spec.ts` | catalog-service | 🟡 In progress |

FR-005 | Uploading and management of multimedia galleries and images | `HU-MEDIA-001` | `media.upload.spec.ts` | media-service | 🔴 Pending |

FR-006 | Multi-criteria search for plans (municipalities, price, category) | `HU-SEARCH-001` | `search.query.spec.ts` | catalog-service | 🟡 In progress |

| FR-007 | Categorization and labeling of regional tourist routes | `HU-CATALOG-002` | `catalog.categories.spec.ts` | catalog-service | 🔴 Pending |

| FR-008 | Tourist booking requests and inquiries | `HU-BOOK-001`<br>`HU-BOOK-003` | `booking.create.spec.ts`<br>`booking.lookup.spec.ts` | booking-service | 🟡 In progress |

| FR-009 | Inventory control, availability, and date blocking | `HU-INVENT-001`<br>`HU-INVENT-002` | `inventory.capacity.spec.ts`<br>`inventory.block.spec.ts` | inventory-service | 🟡 In progress |

| FR-010 | Management and confirmation of booking requests by agencies | `HU-BOOK-002` | `booking.status.spec.ts` | booking-service | 🔴 Pending |

| FR-011 | Recording ratings and reviews based on completed experiences | `HU-REVIEW-001` | `review.submit.spec.ts` | review-service | 🔴 Pending |

| FR-012 | Automatic sending of notifications and transactional emails | `HU-NOTIF-001` | `notification.email.spec.ts` | notification-service | 🔴 Pending |

| FR-013 | Statistics consolidation and general administration panel | `HU-ADMIN-001` | `admin.metrics.spec.ts` | admin-service | 🔴 Pending |

| FR-014 | Moderation of public comments and opinions by the administrator | `HU-REVIEW-002` | `review.moderation.spec.ts` | review-service | 🔴 Pending |

| FR-015 | Generation and export of operational reports in Excel/CSV | `HU-REPORT-001` | `report.export.spec.ts` | booking-service | 🔴 Pending |

--

## NFR → Validation matrix

| NFR ID | Description | How it is validated | Tool | Status |

|--------|-------------|-------------------|------|--------|

| NFR-001 | P95 latency < 300ms under 300 RPS load | Load & performance test in staging pipeline | k6 / Postman | 🟡 In progress |
| NFR-002 | High Availability SLO: 99.9% uptime | Health check probes (`/health/ready`) & Uptime monitoring | Grafana / Prometheus | 🟡Monitoring |
| NFR-003 | Auto-scaling horizontally on CPU > 70% | Stress tests & HPA scaling limits verification | Kubernetes HPA | 🔴Pending |
| NFR-004 | Authentication JWT & BCrypt password hashing | SAST analysis & Automated Security Contract tests | SonarQube / Postman | ✅ Validated |
| NFR-005 | Observability: JSON Logs & Correlation ID | End-to-end distributed tracing across HTTP requests | OpenTelemetry/Winston | 🟡Monitoring |
| NFR-006 | Maintainability: Test Coverage ≥ 80% | CI pipeline gate execution on git push | Jest / Istanbul | 🟡 In progress |
| NFR-007 | Portability: Containerized Docker images | Docker image build execution in CI environment | Docker/K8s | ✅ Validated |
| NFR-008 | Disaster Recovery: Database failover < 5 min | Manual failover simulation in staging environment | PostgreSQL Replication | 🔴Pending |

---

## Inverse traceability: HU → FR

| HU | Title | FR(s) it implements | Sprint |
|----|-------|---------------------|--------|
| HU-AUTH-001 | Travel Agencies Registry | FR-001 | Sprint 1 |
| HU-AUTH-002 | Authentication and Secure Login | FR-002 | Sprint 1 |
| HU-AUTH-003 | Password Recovery and Reset | FR-002 | Sprint 1 |

HU-ADMIN-002 | Agency Approval and Legal Verification | FR-001 | Sprint 1 |

HU-AGENCY-001 | Agency Corporate Profile Management | FR-003 | Sprint 2 |

HU-CATALOG-001 | Creating and Editing Tour Packages | FR-004 | Sprint 2 |

HU-CATALOG-002 | Categorizing and Labeling Tour Routes | FR-007 | Sprint 2 |

HU-CATALOG-003 | Deactivating and Temporarily Disabling Packages | FR-004 | Sprint 2 |

HU-MEDIA-001 | Destination Photo Gallery Management | FR-005 | Sprint 2 |

HU-NOTIF-001 | Automatic Transactional Email Delivery | FR-012 | Sprint 2 |

HU-SEARCH-001 | Multi-Criteria Search for Tourist Packages

| HU-INVENT-001 | Calendar and Available Slot Management | FR-009 | Sprint 3 |

| HU-INVENT-002 | Blocking Dates for Contingencies or Operations | FR-009 | Sprint 3 |

| HU-BOOK-001 | Tourist Plan Reservation Request | FR-008 | Sprint 3 |

| HU-BOOK-002 | Agency Reservation Management and Confirmation | FR-010 | Sprint 3 |

| HU-BOOK-003 | Tourist Reservation History and Inquiry | FR-008 | Sprint 3 |

| HU-ADMIN-001 | General Control Statistics Panel | FR-013 | Sprint 4 |

| HU-REVIEW-001 | Rating and Review of Tourist Services | FR-011 | Sprint 4 |

| HU-REVIEW-002 | Moderation of Reviews by Administration | FR-014 | Sprint 4 |
| HU-REPORT-001 | Export of Operational Reports for Agencies | FR-015 | Sprint 4 |

---

## Status legend

| Status | Meaning |
|--------|---------|
| ✅ Donate | Implemented, tested, and in production |
| 🟡 In progress | Under development in the current sprint |
| 🔴Pending | In the backlog, not started |
| ⏸Blocked | Has an external blocker |
| ❌ Canceled | Removed from scope |

---

## Identified gaps (requirements without coverage)

| Gap type | Description | Required action | Owner | Date |
|----------|-------------|-------------|-------|------|
| None | All 15 FRs from SRS are covered by HUs | Maintain coverage during implementation | Product Owner | 2026-09-15 |

---

##How to maintain this matrix

1. When an HU is created: add the row in the FR → HU → Test → Service section.
2. When a test is written: note the file in the "Tests that verify it" column.
3. When an HU is completed: change the status to ✅.
4. At each Sprint Planning: review gaps and assign actions.

---

## Correlations

- User Stories → `04-requirements/user-stories.md`[cite: 3]
- Non-Functional Requirements → `04-requirements/non-functional.md`
- Testing strategy → `11-quality/testing-strategy.md`
