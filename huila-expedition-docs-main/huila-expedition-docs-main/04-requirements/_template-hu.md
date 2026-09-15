# HU-[SERVICE]-[NNN]: [Story Title]

> **ID convention:** `HU-[SERVICE_ABBREVIATION]-[NNN]`
> Examples: HU-IAM-001, HU-SCHED-023, HU-REF-005

---

## Story

**As** [user role — e.g.: instructor, coordinator, learner, administrator]
**I want** [concrete action they want to perform]
**So that** [benefit they receive / problem they solve]

---

## Acceptance criteria

> Format: "Given [context/initial state], when [user action], then [expected and verifiable result]"

- [ ] **AC1:** Given that [context], when [action], then [result]
- [ ] **AC2:** Given that [context], when [action], then [result]
- [ ] **AC3:** [Error scenario] Given that [invalid context], when [action], then [expected error]

---

## Technical notes

> [Implementation constraints, performance considerations, required integrations]

**Responsible service(s):** [microservice name]
**Endpoint(s) implemented:** [method + path]
**Events generated:** [if applicable]
**Required permissions:** [minimum role to perform this action]

---

## Definition of Done (DoD)

> This HU can only be closed when it meets the team's full DoD.
> See: [`00-governance/definition-of-done.md`](../../00-governance/definition-of-done.md)

**Additional checks specific to this HU (if applicable):**
- [ ] [Additional check not covered by the general DoD — e.g.: DB migration executed in staging]
- [ ] [Remove this section if there are no additional checks]

---

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [1 / 2 / 3 / 5 / 8 / 13] |
| Priority | High / Medium / Low |
| Target sprint | Sprint N |
| Dependencies | [HU-XXX-NNN that must be completed first] |

# HU-AUTH-001: Travel Agency Registration

**ID convention**: HU-AUTH-001

## Story

**As**  a representative of a local tourism agency, 
**I want** to register my company via an online form by providing legal and contact details, 
**so that** it can be verified by the platform and I can start promoting my tourism offers. 

## Acceptance criteria

- [ ] AC1: Given that the agency enters valid data (Name, NIT, RNT, email, phone, and password), when the registration form is submitted, the system stores the account with a status of "Pending Approval" and sends an email containing a confirmation link.
- [ ] AC2: Given that registration is successfully completed, when the email is activated, the system generates an automatic notification on the Administrator dashboard for RNT verification.
- [ ] AC3: Given that the agency attempts to register using an NIT or RNT that already exists in the system, when the form is submitted, the system displays an error alert indicating that the data is already registered and does not process the registration.

## Technical notes

**Responsible service(s)**: auth-service, agency-service
**Endpoint(s) implemented**: POST /api/v1/agencies/register
**Events generated**: AgencyRegisteredEvent
**Required permissions**: Public / Anonymous

## Definition of Done (DoD)
- [ ] Migration executed for the agencies table with an initial status of PENDING.
- [ ] Uniqueness validation for NIT and RNT implemented in the backend.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 1 |
| Dependencies | [No one. It is the initial entry point for corporate users on the platform. It does not require the prior existence of any other service or module] |

---

# HU-AUTH-002: Authentication and Secure Login

**ID convention**: HU-AUTH-002

## Story

**As** a registered user (Travel Agency or Administrator), 
**I want** to log in using my access credentials (email and password) 
**so that** I can securely access my private dashboard based on my assigned role. 

## Acceptance criteria

- [ ] AC1: Given that the user enters the correct credentials, when they press the login button, the system generates a session token (JWT) and redirects them to the dashboard corresponding to their role.
- [ ] AC2: Given that the user remains inactive for 30 minutes, when they attempt to perform an action, the system expires the session and redirects them to the login screen.
- [ ] AC3: Given that incorrect credentials are entered 5 consecutive times, upon the fifth failed attempt, the system temporarily locks the account for 15 minutes and notifies the user.

## Technical notes

**Responsible service(s)**: auth-service
**Endpoint(s) implemented**: POST /api/v1/auth/login
**Events generated**: UserLoggedInEvent, AccountLockedEvent
**Required permissions**: Public / Anonymous 

## Definition of Done (DoD)
- [ ] JWT integration with a 30-minute expiration.
- [ ] Automatic lockout mechanism configured in Redis/Memcached.
  
## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | High |
| Target sprint | Sprint 1 |
| Dependencies | [HU-AUTH-001 - It is not possible to authenticate or validate the credentials of a user or agency unless a registration mechanism and a credential storage structure already exist] |

---

# HU-CATALOG-001: Creation and Editing of Tourism Plans 

**ID convention**: HU-CATALOG-001

## Story

**As** an authenticated travel agency, 
**I want** to register, update, or remove my tourism plans and packages
**so that** tourists can view clear, accurate, and up-to-date information about my offers. | Acceptance criteria

- [ ] AC1: When the agency is in the catalog module, fills in the mandatory fields (name, description, Huila municipality, tourism type, price, and duration), and saves, the plan is published in the public catalog.
- [ ] AC2: When the agency updates the details of an existing plan and confirms the edit, the changes are immediately reflected in the user search view.
- [ ] AC3: When the agency attempts to delete or disable a plan with active reservations and confirms the action, the system displays a warning preventing deletion and suggesting it be marked as "Unavailable."

## Technical notes

**Responsible service(s)**: catalog-service
**Endpoint(s) implemented**: POST /api/v1/plans, PUT /api/v1/plans/{id}, DELETE /api/v1/plans/{id}
**Events generated**: PlanCreatedEvent, PlanUpdatedEvent
**Required permissions**: ROLE_AGENCY 

## Definition of Done (DoD)

- [ ] Complete CRUD endpoints for tourism plans.
- [ ] Validation of active reservation status before disabling a plan.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 1 |
| Dependencies | [HU-AUTH-002 - To create or modify tourism offers, the system must first verify the agency's identity and role (ROLE_AGENCY) using the JWT generated during login] |

---

# HU-MEDIA-001: Destination Photo Gallery Management

**ID convention**: HU-MEDIA-001

## Story

**As** a travel agency, 
**I want** to upload and manage illustrative images for each tour package 
**so that** the offer is attractive and accurately depicts the included sites and services.

## Acceptance criteria

- [ ] AC1: When the agency edits a package and selects up to 10 images (JPG, PNG, or WEBP format; under 5 MB per file), the system processes the storage and links the gallery to the package.
- [ ] AC2: If images exceed standard web dimensions, the system automatically compresses them upon upload while maintaining visual quality.
- [ ] AC3: If the agency attempts to upload a file in an unsupported format (e.g., PDF or ZIP), the system displays a message indicating accepted formats and rejects the file.
      
## Technical notes
**Responsible service(s)**: media-service
**Endpoint(s) implemented**: POST /api/v1/plans/{id}/images, DELETE /api/v1/plans/{id}/images/{imageId}
**Events generated**: ImageUploadedEvent
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)

- [ ] Connection configured with object storage service (S3 / Cloudinary).
- [ ] Server-side image compression enabled.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Medium |
| Target sprint | Sprint 2 |
| Dependencies | [HU-CATALOG-001 - Images do not exist in isolation; they must be associated with the identifier (plan_id) of a previously registered tourism plan.] |

---

# HU-INVENT-001: Calendar and Capacity Management

**ID convention**: HU-INVENT-001

## Story

**As** a travel agency, 
**I want** to configure a calendar specifying maximum capacities and operational dates for each plan, 
**so that** overbooking is prevented and tourists can see real-time availability. 

## Acceptance criteria

- [ ] AC1: When the agency selects a date on a plan's calendar, assigns the maximum capacity, and enables it, the system updates that date's status to "Available."
- [ ] AC2: When the number of confirmed bookings reaches the capacity limit for a date, the system automatically changes the day's status to "Sold Out" upon processing the final booking.
- [ ] AC3: When the agency disables a previously configured date and confirms the block, that date becomes unselectable for new requests in the public view.

## Technical notes

**Responsible service(s)**: inventory-service  
**Endpoint(s) implemented**: POST /api/v1/plans/{id}/availability, GET /api/v1/plans/{id}/availability
**Events generated**: AvailabilityUpdatedEvent
**Required permissions**: ROLE_AGENCY  

## Definition of Done (DoD)

- [ ] Real-time capacity decrement logic implemented.
- [ ] Date blocking control for maintenance or contingencies implemented.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 2 |
| Dependencies | [HU-CATALOG-001-An availability and capacity calendar requires an existing tourism plan to which operational dates and participant limits can be assigned] |


---

# HU-SEARCH-001: Multi-criteria Search for Tourism Plans

**ID convention**: HU-SEARCH-001

## Story
**As** a tourist or platform visitor, 
**I want** to filter tourism offers by municipality, price range, tourism type, and duration 
**so** that I can quickly find experiences that match my budget and interests. 

## Acceptance criteria

- [ ] AC1: When the tourist selects one or more filters (e.g., Municipality: San Agustín, Type: Archaeological) and views the results, the system displays only the plans that meet all the selected parameters.
- [ ] AC2: If no plans exactly match the search criteria, the system displays an informational message recommending that the user adjust the filters and shows featured plans from the region.
- [ ] AC3: When the tourist clicks "Clear filters," the form resets all criteria and displays the complete list of active plans.

## Technical notes

**Responsible service(s)**: search-service  
**Endpoint(s) implemented**: GET /api/v1/search/plans  
**Events generated**: No one
**Required permissions**: Public / Anonymous  

## Definition of Done (DoD)
- [ ] Search indices optimized by municipality and category.
- [ ] Query response optimized to return results in less than 3 seconds.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | High |
| Target sprint | Sprint 2 |
| Dependencies | [HU-CATALOG-001 - The search engine requires published plan data to be present in the database in order to apply filters by municipality, price, or type of tourism] |

---

# HU-BOOK-001: Tour Package Booking Request

**ID convention**: HU-BOOK-001

## Story

**As** a tourist interested in a tour package, 
**I want** to fill out a booking request specifying the date, number of participants, and my contact details, 
**so that** the organizing agency receives my request and reserves the selected spots. 

## Acceptance criteria

- [ ] AC1: Given that the tourist selects an available date and a number of spots within the limit, when they complete their personal details (name, phone, email) and submit the request, the system registers the booking with a "Pending" status and issues a unique request code.
- [ ] AC2: Given that the request is successfully generated, when the transaction is completed, the system sends a confirmation email containing the booking details to both the tourist and the operating agency.
- [ ] AC3: Given that the requested spots exceed the availability shown on the calendar, when the user attempts to submit the form, the system notifies them that there are insufficient spots for that date.

## Technical notes

**Responsible service(s)**: booking-service
**Endpoint(s) implemented**: POST /api/v1/bookings
**Events generated**: BookingCreatedEvent
**Required permissions**: Public / Anonymous 

## Definition of Done (DoD)

- [ ] Integration with inventory-service for temporary blocking of spots.
- [ ] Automatic generation of a unique alphanumeric code per booking.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 3 |
| Dependencies | [HU-INVENT-001, HU-SEARCH-001 - A tourist cannot request a reservation if they cannot search for or view the plan (HU-SEARCH-001) and if the system cannot validate whether there is availability for the selected date (HU-INVENT-001)] |

---

# HU-BOOK-002: Booking Management and Confirmation by the Agency

**ID convention**: HU-BOOK-002

## Story

**As** a travel agency operator, 
**I want** to review received booking requests and change their status to "Approved" or "Rejected" 
**so that** I can coordinate payment and logistics directly with the tourist.

## Acceptance criteria

- [ ] AC1: When the agency accesses the booking list in its panel and changes a request's status from "Pending" to "Approved," the booking is confirmed, and the calendar slot is permanently occupied.
- [ ] AC2: When the agency decides to "Reject" or "Cancel" a booking, enters a brief reason, and confirms, the system releases the calendar slots and notifies the tourist via email.
- [ ] AC3: Whenever a change is made to the booking status and the update is saved, the system triggers an event for the notification service.

## Technical notes

**Responsible service(s)**: booking-service, notification-service
**Endpoint(s) implemented**: PATCH /api/v1/bookings/{id}/status
**Events generated**: BookingStatusChangedEvent
**Required permissions**: ROLE_AGENCY 

## Definition of Done (DoD)

- [ ] Logic for automatic slot reallocation and release.
- [ ] Email templates configured for status change notifications.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 3 |
| Dependencies | [HU-BOOK-001 - An agency cannot approve, reject, or change the status of a reservation if the tourist has not previously generated the request] |

---

# HU-REVIEW-001: Rating and Review of Tourism Services

**ID convention**: HU-REVIEW-001

## Story

**As** a tourist who has completed a tourism plan, 
**I want** to assign a 1-to-5-star rating and write a comment about my experience 
**so that** other users have real references regarding the agency's quality.

## Acceptance criteria

- [ ] AC1: Given that a tourist has a booking code with a "Completed" status, when they access the rating form, assign stars, and write their review, the system records the opinion with a "Pending moderation" status.
- [ ] AC2: Given that the Administrator approves the published review, when the status is updated, the comment becomes visible on the plan's details page, and the agency's average star rating is recalculated.
- [ ] AC3: Given that a user attempts to rate a plan without having had a prior or valid booking, when they try to submit the form, the system rejects the action, indicating that only verified users can leave reviews.
      
## Technical notes

**Responsible service(s)**: review-service
**Endpoint(s) implemented**: POST /api/v1/plans/{id}/reviews, GET /api/v1/plans/{id}/reviews
**Events generated**: ReviewCreatedEvent
**Required permissions**: Public / Tourist with validated booking

## Definition of Done (DoD)
- [ ] Validation of the completed booking code before accepting the review.
- [ ] Automatic calculation of the weighted average rating.
      
## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Medium |
| Target sprint | Sprint 4 |
| Dependencies | [HU-BOOK-002 - To ensure genuine reviews and prevent spam, the system requires verification that the user has a confirmed and completed booking (HU-BOOK-002) before enabling the rating form] |

---

# HU-ADMIN-001: General Statistics Dashboard

**ID convention**: HU-ADMIN-001

## Story

**As** an HTE system administrator, 
**I want** to view a dashboard with general platform metrics (active agencies, monthly bookings, published plans) 
**so that** I can monitor the growth of the tourism ecosystem and general activity.

## Acceptance criteria

- [ ] AC1: Given that the Administrator authenticates successfully, when accessing the dashboard route, the system consolidates and displays key metrics in real-time using cards and indicators.
- [ ] AC2: Given that the Administrator filters metrics by a date range, when the filter is applied, agency booking and projected revenue data are recalculated based on that period.
- [ ] AC3: Given that a user with the "Agency" role or an unauthenticated user attempts to access the administration endpoint, when the request is sent, the system returns an HTTP 403 Forbidden code, blocking access.
      
## Technical notes

**Responsible service(s)**: admin-service
**Endpoint(s) implemented**: GET /api/v1/admin/dashboard/metrics
**Events generated**: No one
**Required permissions**: ROLE_ADMIN 

## Definition of Done (DoD)

- [ ] Metric aggregation queries optimized to avoid slowing down the database.
- [ ] Access control based on user profile (ROLE_ADMIN) verified via testing.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | Medium |
| Target sprint | Sprint 4 |
| Dependencies | [HU-AUTH-OO2, HU-BOOK-002 - It requires administrator role authentication (HU-AUTH-002) to restrict access, and necessitates the existence of active and historical booking transactions (HU-BOOK-002) to calculate and consolidate dashboard metrics] |
