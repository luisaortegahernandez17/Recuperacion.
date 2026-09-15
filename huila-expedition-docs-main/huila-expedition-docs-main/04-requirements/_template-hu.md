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

---

# HU-ADMIN-002: Agency Approval and Legal Verification

**ID convention**: HU-ADMIN-002

## Story

**As** an HTEI system administrator, 
**I want** to review the legal documentation (NIT and RNT) of registered agencies with a "pending" status 
**so that** I can enable only formal and authorized tourism service providers in the Huila department.

## Acceptance criteria
- [ ] AC1: When the administrator accesses the list of pending requests, verifies the RNT on the national platform, and clicks "Approve," the system changes the agency's status to ACTIVE and sends credentials/a welcome notification.
- [ ] AC2: When the administrator finds legal inconsistencies, selects "Reject," and enters the reason, the system changes the status to REJECTED and notifies the agency via email.
- [ ] AC3: When an agency attempts to publish offers or log in before being approved, the system displays a message indicating that their account is undergoing verification.
      
## Technical notes

**Responsible service(s)**: admin-service, agency-service
**Endpoint(s) implemented**: PATCH /api/v1/admin/agencies/{id}/status
**Events generated**: AgencyStatusApprovedEvent, AgencyStatusRejectedEvent
**Required permissions**: ROLE_ADMIN

## Definition of Done (DoD)
- [ ] Agency entity state transition (PENDING - ACTIVE/REJECTED) tested on the backend.
- [ ] Domain event emitted to notify the email microservice (notification-service).

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | High |
| Target sprint | Sprint 1 |
| Dependencies | [HU-AUTH-OO1 - The administrator cannot audit or approve/reject an agency if it has not previously completed the initial registration process and entered its NIT/RNT into the database.] |

---

# HU-NOTIF-001: Automated Transactional Email Dispatch
**ID convention**: HU-NOTIF-001

## Story

**As** an HTE system microservice
**I want** to process message bus events and dispatch email templates
**So that** users and agencies receive confirmations, booking alerts, and status updates in real time.

## Acceptance criteria
- [ ] AC1: Given a transactional event is emitted (e.g., BookingCreatedEvent), when the notification-service consumes the message from the queue, it generates and sends an HTML email using the corresponding template within 5 seconds.
- [ ] AC2: Given a temporary email provider failure, when a dispatch error occurs, the system retries the dispatch up to 3 times using a Dead Letter Queue (DLQ).
- [ ] AC3: Given an invalid email address, when the SMTP server returns a bounce, the system logs the incident without halting queue processing.

## Technical notes

**Responsible service(s)**: notification-service
**Endpoint(s) implemented**: Asynchronous (Message Broker Consumer)
**Events generated**: EmailSentEvent, EmailFailedEvent
**Required permissions**: Microservice-internal

## Definition of Done (DoD)

- [ ] Integration configured with SMTP server / messaging API (SendGrid/Mailgun/SES).
- [ ] Responsive HTML templates created for registration, confirmation, and cancellation.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [5] |
| Priority | High |
| Target sprint | Sprint 2 |
| Dependencies | [HU-AUTH-OO1, HU-BOOK-001 - The notification service requires the existence of actual event-emitting entities containing destination data (emails of agencies registered in HU-AUTH-001 and booking requests created in HU-BOOK-001)] |

---

# HU-REVIEW-002: Review Moderation by Administration

**ID convention**: HU-REVIEW-002

## Story

**As** a platform administrator
**I want** to audit and moderate comments and ratings submitted by tourists
**So that** I can approve their public publication or reject content containing offensive language or spam.

## Acceptance criteria

- [ ] AC1: When the administrator accesses the moderation module, reviews a review in PENDING status, and approves it, the comment changes to PUBLISHED status and becomes publicly visible.
- [ ] AC2: If a review violates the terms of use, when the administrator clicks "Reject," the comment changes to REJECTED status and is not included in the plan's average rating.
- [ ] AC3: Once the review is published or rejected and its status changes, the microservice automatically recalculates the weighted average rating for the corresponding plan.

## Technical notes

**Responsible service(s)**: review-service, admin-service
**Endpoint(s) implemented**: PATCH /api/v1/reviews/{id}/moderation
**Events generated**: ReviewModeratedEvent
**Required permissions**: ROLE_ADMIN

## Definition of Done (DoD)

- [ ] Moderation endpoint with ROLE_ADMIN authorization tested on the backend.
- [ ] Star rating average recalculation updated in the Plan entity.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Medium |
| Target sprint | Sprint 4 |
| Dependencies | [HU-REVIEW-OO1 - It is not possible to audit or moderate a rating if a tourist with a completed booking has not first written and submitted their review to the moderation queue] |

---

# HU-CATALOG-002: Categorization and Tagging of Tourist Routes

**ID convention**: HU-CATALOG-002

## Story

**As** a travel agency, 
**I want** to assign specific categories (ecotourism, archaeological, adventure, gastronomic) and tags to my plans 
**so that** my offers are correctly classified within the region's search engines.

## Acceptance criteria

- [ ] AC1: When the agency creates or edits a plan and selects one or more categories from the predefined Huila list, the system links the tags to the plan.
- [ ]  AC2: When a tourist searches for a specific category (e.g., "Archaeological") on the platform, the system lists only the plans tagged under that category.
- [ ] AC3: When the agency attempts to save a plan without selecting at least one main category and submits the form, the system requires the selection of a mandatory category.
      
## Technical notes

**Responsible service(s)**: catalog-service
**Endpoint(s) implemented**: POST /api/v1/categories, PUT /api/v1/plans/{id}/categories
**Events generated**: PlanCategoriesUpdatedEvent
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)
- [ ] Many-to-Many {N}:{M} relationship mapping between Plan and Category migrated in the database.
- [ ] Active categories query endpoint optimized.

## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [2] |
| Priority | Medium |
| Target sprint | Sprint 2 |
| Dependencies | [HU-CATALOG-OO1 - Categories and tags are complementary attributes directly associated with the identifier of a primary tourism plan that has already been structured |

---

# HU-AUTH-003: Password Recovery and Reset

**ID convention**: HU-AUTH-003

## Story

**As** a registered user (Agency / Administrator)
**I want** to request a recovery link using my registered email address
**So that** I can securely reset my password if I forget it.

## Acceptance criteria

- [ ] AC1: Given that the user enters their email into the recovery form and clicks submit, the system generates a unique, one-time-use token valid for 15 minutes and sends the email.
- [ ] AC2: Given that the user clicks the link and enters a valid new password, upon confirming the change, the token is invalidated and the password is updated in encrypted form.
- [ ] AC3: Given that the user attempts to use an expired or previously used token, upon submitting the new password, the system displays an invalid token alert and rejects the update.

## Technical notes
**Responsible service(s)**: auth-service, notification-service
**Endpoint(s) implemented**: POST /api/v1/auth/forgot-password, POST /api/v1/auth/reset-password
**Events generated**: PasswordResetRequestedEvent, PasswordChangedEvent
**Required permissions**: Public / Anonymous

## Definition of Done (DoD)

- [ ] Bcrypt/Argon2 encryption mechanism for password updates configured.
- [ ] Expiration control for temporary recovery tokens verified.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Medium |
| Target sprint | Sprint 1 |
| Dependencies | [HU-AUTH-OO1, HU-AUTH-002 - Passwords can only be reset for accounts that already exist in the system (HU-AUTH-001) and use the credential-based authentication scheme (HU-AUTH-002) |

---

# HU-AGENCY-001: Agency Corporate Profile Management

**ID convention**: HU-AGENCY-001

## Story

**As** an authenticated travel agency
**I want** to update my company's institutional details (logo, description, social media, support phone number)
**So that** tourists have reliable and up-to-date contact information about my company.

## Acceptance criteria

- [ ] AC1: Given that the agency is in its configuration panel, when it modifies business information and uploads its logo, the system validates the dimensions and saves the data to the profile.
- [ ] AC2: Given that the data has been saved, when a tourist views the agency's public profile, they see the updated institutional information.
- [ ] AC3: Given that the agency attempts to modify its already verified NIT or RNT, when it tries to save, the system blocks those fields, specifying that they require administrative validation.

## Technical notes

**Responsible service(s)**: agency-service, media-service
**Endpoint(s) implemented**: GET /api/v1/agencies/me, PUT /api/v1/agencies/me
**Events generated**: AgencyProfileUpdatedEvent
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)
- [ ] Editing restriction for identifiable legal fields (NIT, RNT) implemented in the backend.
- [ ] Brand logo upload endpoint integrated with object storage.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [2] |
| Priority | Medium |
| Target sprint | Sprint 2 |
| Dependencies | [HU-ADMIN-002 - An agency cannot self-manage or publish its official business profile unless it has successfully completed the administrative verification and approval stage for its RNT. |

---

# HU-CATALOG-003: Temporary Deactivation and Disabling of Plans

**ID convention**: HU-CATALOG-003

## Story

**As** a travel agency
**I want** to temporarily pause the visibility of a tour plan without deleting it from the system
**So that** I can suspend sales due to weather or seasonal factors without losing the plan's history.

## Acceptance criteria

- [ ] AC1: When the agency selects the "Pause publication" option for one of its active plans and confirms, the plan's status changes to INACTIVE and it is hidden from public search results.
- [ ] AC2: When the agency decides to reactivate the offer and presses "Activate," the plan returns to ACTIVE status and becomes immediately visible in the catalog.
- [ ] AC3: When a plan is in INACTIVE status and a tourist attempts to access it via a saved direct link, the system displays a message indicating that the plan is temporarily unavailable.

## Technical notes

**Responsible service(s)**: catalog-service
**Endpoint(s) implemented**: PATCH /api/v1/plans/{id}/status
**Events generated**: PlanStatusChangedEvent
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)

- [ ] Filtering logic in search-service to exclude plans with INACTIVE status.
- [ ] Database status update validated via integration tests.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [2] |
| Priority | Low |
| Target sprint | Sprint 2 |
| Dependencies | [HU-CATALOG-001 - The functionality to pause or activate visibility presupposes the prior existence of the Plan entity registered in the catalog |      

---

# HU-BOOK-003: Tourist Booking History and Lookup

** ID convention**: HU-BOOK-003

## Story

**As** a tourist or platform visitor
**I want** to check the status of my booking requests by entering my unique code and email
**So that** I can verify if my experience has been confirmed by the agency or view trip details.

## Acceptance criteria

- [ ] AC1: Given that the tourist enters their alphanumeric booking code and email address, when they press "Lookup," the system displays the full details of the plan, date, and current status (PENDING, APPROVED, CANCELLED).
- [ ] AC2: Given that the booking is in APPROVED status, when the user views the lookup result, the system displays payment instructions and the agency's contact numbers.
- [ ] AC3: Given that the combination of code and email does not match any record, when the lookup is performed, the system displays an error alert indicating that the booking was not found.

## Technical notes

**Responsible service(s)**: booking-service
**Endpoint(s) implemented**: POST /api/v1/bookings/lookup
**Events generated**: No one
**Required permissions**: Public / Anonymous

## Definition of Done (DoD)
- [ ] Public booking lookup optimized using a composite index (booking_code + customer_email).
- [ ] Status display interface tested in the development environment.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [2] |
| Priority | Medium |
| Target sprint | Sprint 3 |
| Dependencies | [HU-BOOK-001 - This query strictly requires that the booking engine has previously generated requests with a unique tracking code in the database |

---

# HU-INVENT-002: Date Blocking for Contingency or Operational Reasons

**ID convention**: HU-INVENT-002

## Story

**As** a travel agency
**I want** to manually disable or block specific days on a plan's calendar
**So that** no requests are received for dates involving maintenance, track maintenance, or non-operational holidays.

## Acceptance criteria
- [ ] AC1: Given that the agency selects a date range on the calendar, when the "Block for operational reasons" option is selected, the system changes the status of those days to BLOCKED.
- [ ] AC2: Given that a date has a BLOCKED status, when a tourist attempts to select it on the booking form, the date appears disabled/greyed out in the public interface.
- [ ] AC3: Given that there are requests with a PENDING status for the date to be blocked, when the agency confirms the block, the system warns them that they must first reject or rebook those requests.

## Technical notes

**Responsible service(s)**: inventory-service
**Endpoint(s) implemented**: POST /api/v1/plans/{id}/block-dates
**Events generated**: DatesBlockedEvent
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)
- [ ] Manual blocking logic based on date ranges implemented in inventory-service.
- [ ] Validation check to prevent blocking dates that have active APPROVED bookings.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Medium |
| Target sprint | Sprint 3 |
| Dependencies | [HU-INVENT-001 - The manual block operates on the availability and quota matrix created and initialized by the inventory service |

---

# HU-REPORT-001: Exporting Operational Reports for Agencies

**ID convention**: HU-REPORT-001

## Story

**As** a travel agency
**I want** to download reports in Excel/CSV format containing the list of bookings and passengers by date
**So that** I can coordinate logistics regarding transportation, tour guides, and medical insurance for the attendees.

## Acceptance criteria
- [ ] AC1: Given that the agency selects a date range or a specific plan, when they click "Export Report (CSV/Excel)," the system generates a downloadable file containing the list of passengers.
- [ ] AC2: Once the file is generated, when the agency opens it, it includes data on the primary contact, ID document, phone number, number of people, rate, and booking status.
- [ ] AC3: If no bookings exist for the selected criteria, when the export is requested, the system displays a notification stating that there is no data available to generate the document.

## Technical notes

**Responsible service(s)**: booking-service, report-service
**Endpoint(s) implemented**: GET /api/v1/reports/bookings/export
**Events generated**: No one
**Required permissions**: ROLE_AGENCY

## Definition of Done (DoD)
- [ ] Dynamic generation of CSV/XLSX files implemented on the backend.
- [ ] Performance tests verifying report downloads with more than 500 records.

 ## Estimation and priority

| Field | Value |
|-------|-------|
| Story Points | [3] |
| Priority | Low |
| Target sprint | Sprint 4 |
| Dependencies | [HU-BOOK-002 - Tourism operation reports are compiled based on the consolidated data of reservations managed and confirmed by the agencies |
