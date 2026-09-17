# User Stories — Backlog (Huila Travel Expedition)

> **What to fill in here:** The product's user story backlog.

> Each user story uses the standard format with acceptance criteria in Given/When/Then format.

--

## Backlog Status

| Cut | Sprint | Total User Stories | Refined | In Progress | Completed |
|--------|-----------|---------|-------------|-----------|
| Cut 1 | Sprint 1-2 | 10 | 10 | 0 | 0 |
| Cut 2 | Sprint 3-4 | 10 | 10 | 0 | 0 |

--

## Epics

| ID | Epic | Description |
|----|------|-------------|
| EP-001 | Authentication and Agency Management | Legal registration, RNT validation, secure login, and institutional profile management for travel agencies in Huila. |
EP-002 | Regional Tourism Catalog and Offerings | Creation, editing, categorization (ecotourism, adventure, culture), and temporary deactivation of tour packages. |
EP-003 | Search, Reservations, and Inventory | Search engine by municipality, availability/slot management by date, reservations, and operational blocks. |
EP-004 | Reputation, Notifications, and Administration | Verified reviews, content moderation, transactional emails, and a general statistics dashboard. |

---

## User Stories

### HU-AUTH-001 — Travel Agency Registration {#HU-AUTH-001}

**Epic:** EP-001

 -**As** the legal representative of a travel agency in Huila, 
 **I want** to register my company's business and legal information (NIT, RNT) 
 **so that** I can request corporate access to the HTE platform.

**Acceptance Criteria:**

Scenario 1: Successful Agency Application Registration
Since the agency enters the registration form with a valid NIT and RNT,
When they submit the completed form,
The system creates the account in PENDING status and issues a pending verification alert.

Scenario 2: Attempted registration with a duplicate NIT or RNT
Since the agency enters a previously registered NIT or RNT,
when they press the registration button,
the system displays the message "The NIT or RNT entered is already registered on the platform.

 ## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (agency.register.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/auth/register/agency)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 1 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [No one] |
| Affected service(s) | [auth-service,agency-service] |      

---

### HU-AUTH-002 — Secure Authentication and Login {#HU-AUTH-002}

**Epic:** EP-001

**As** a registered user (Agency / Administrator)
**I want** to log in with my credentials
**so that** I can access the protected functions according to my role.

**Acceptance Criteria:**

Scenario 1: Successful Login
Given that the user enters their email and correct password
When they press login
Then the system returns a valid JWT token (expiration 30 min) and redirects them to the control panel

Scenario 2: Invalid Credentials

Given that the user enters an incorrect password
When they attempt to log in
Then the system displays the message "Invalid Credentials" and does not generate the token

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (auth.login.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/auth/login)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 1 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-001] |
| Affected service(s) | [auth-service] |      

---

### HU-AUTH-003 — Password Recovery and Reset {#HU-AUTH-003}

**Epic:** EP-001

**As** a registered user
**I want** to request a recovery link to my email
**so that** I can reset my password if I forget it.

**Acceptance Criteria:**

Scenario 1: Successful Password Recovery Request
Given that the user enters their registered email address
When they click "Send Link"
The system generates a one-time token valid for 15 minutes and sends the email

Scenario 2: Expired or Invalid Token
Given that the user uses a link that has expired after 15 minutes
When they attempt to send the new password
The system rejects the request and informs the user that the link has expired

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (auth.reset-password.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/auth/reset-password)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 1 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-001, HU-AUTH-002] |
| Affected service(s) | [auth-service, notification-service] |

---

### HU-ADMIN-002 — Approval and Legal Verification of Agencies {#HU-ADMIN-002}

**Epic:** EP-001

**As** the HTE system administrator,
**I want** to verify the validity of the RNT (National Tourism Registry) of registered agencies
**so that** I can only authorize formal tourism providers.

**Acceptance Criteria:**

Scenario 1: Formal Agency Approval
Given that the administrator reviews an agency in PENDING status and validates its RNT
When they click "Approve"
The agency's status changes to ACTIVE, and they are notified by email.

Scenario 2: Rejection due to Inconsistent Documentation
Given that the RNT documentation does not match or presents irregularities
When the administrator clicks "Reject" and enters the reason
The agency's status changes to REJECTED, and they receive a notification with the justification.

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (agency.approval.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PATCH /api/v1/admin/agencies/{id}/status)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 1 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-001] |
| Affected service(s) | [admin-service, agency-service] |

---

### HU-AGENCY-001 — Agency Corporate Profile Management {#HU-AGENCY-001}

**Epic:** EP-001

**As** an authenticated travel agency,
**I want** to update my company's institutional information (logo, contact details, description)
**so that** tourists have reliable contact information.

**Acceptance Criteria:**

Scenario 1: Successful Profile Update
Given that the agency modifies its support phone number and institutional logo
When it saves the changes
Then the system validates the information and updates the agency's public profile

Scenario 2: Attempt to Modify Verified Legal Data
Given that the agency attempts to edit its already approved Tax ID (NIT) or National Taxpayer Registry (RNT)
When it attempts to save
Then the system blocks those fields, specifying that they require administrative validation

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (agency.profile.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PUT /api/v1/agencies/me)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 2 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Frontend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-ADMIN-002] |
| Affected service(s) | [media-service,agency-service] |

---

### HU-CATALOG-001 — Creation and Editing of Tour Packages {#HU-CATALOG-001}

**Epic:** EP-002

**As** a verified travel agency,
**I want** to register tour packages with itineraries and rates
**so that** they are available for tourists to view and purchase.

**Acceptance Criteria:**

Scenario 1: Successful publication of a tour package
Given that the agency enters the name, description, itinerary, price per person, and municipality in Huila
When the information is saved
Then the package is registered in ACTIVE status and visible in the catalog

Scenario 2: Registration without required fields
Given that the agency omits the price or municipality from the package
When attempting to save the package
Then the system displays the required validation messages

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (catalog.crud.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/plans)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-002-HU-ADMIN-002] |
| Affected service(s) | [catalog-service] |

---

### HU-CATALOG-002 — Route Categorization and Labeling {#HU-CATALOG-002}

**Epic:** EP-002

**As** a travel agency,
**I want** to assign categories (ecotourism, archaeological, adventure) to my plans
**so that** they are correctly classified within regional searches.

**Acceptance Criteria:**

Scenario 1: Successful Category Assignment
Given that the agency edits a plan and selects the tags "Adventure" and "Saint Augustine"
When saving the configuration
Then the plan is indexed under those thematic categories

Scenario 2: Attempted Saving Without Primary Category
Given that the agency attempts to save a plan without selecting at least one category
When submitting the form
Then the system requires the selection of at least one primary category

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (catalog.categories.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PUT /api/v1/plans/{id}/categories)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 2 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Frontend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-CATALOG-001] |
| Affected service(s) | [catalog-service] |

---

### HU-CATALOG-003 — Deactivation and Temporary Disabling of Plans {#HU-CATALOG-003}

**Epic:** EP-002

**As** a travel agency
**I want** to temporarily pause the visibility of a travel plan
**so that** I can suspend sales due to weather or season without losing the history.

**Acceptance Criteria:**

Scenario 1: Pause Plan Visibility
Given that the agency selects "Pause Publication" on an active plan
When they confirm the action
Then the plan goes to INACTIVE status and is automatically hidden from public searches

Scenario 2: Reactivate Offer
Given that the agency selects "Activate" on a plan in INACTIVE status
When they confirm the reactivation
Then the plan goes to ACTIVE status and becomes visible again in the catalog

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (catalog.status.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PATCH /api/v1/plans/{id}/status)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 2 ] |
| Priority | [Could Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-CATALOG-001] |
| Affected service(s) | [catalog-service] |

---

### HU-MEDIA-001 — Destination Photo Gallery Management {#HU-MEDIA-001}

**Epic:** EP-002

**As** a travel agency,
**I want** to attach high-quality photos to my tour packages
**so that** tourists can visualize the facilities and attractions included in the package.

**Acceptance Criteria:**

Scenario 1: Successful Image Upload
Given that the agency attaches up to 5 JPG/PNG files, each no larger than 5MB
When the upload is confirmed
Then the images are optimized and linked to the plan's gallery

Scenario 2: Unauthorized File Upload
Given that the agency attempts to upload a PDF file or one larger than 5MB
When the upload is processed
Then the system rejects the file and displays the invalid format/size message

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (media.upload.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/plans/{id}/images)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-CATALOG-001] |
| Affected service(s) | [media-service,catalog-service] |

---

### HU-NOTIF-001 — Automatic Sending of Transactional Emails {#HU-NOTIF-001}

**Epic:** EP-004

**As** a microservice of the HTE system
**I want** to process events and send automatic emails
**so that** users and agencies receive booking alerts and status changes.

**Acceptance Criteria:**

Scenario 1: Booking Event Processing
Given that a BookingCreatedEvent is emitted by the broker
When the notification-service consumes the message
Then generate and send an HTML confirmation email in less than 5 seconds

Scenario 2: Retry due to SMTP Server Failure
Given that the email provider generates a timeout
When the sending failure is detected
Then retry sending up to 3 times using the DLQ queue

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (notification.email.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (Event Consumer)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 2 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-001, HU-BOOK-001] |
| Affected service(s) | [notification-service] |

---

### HU-SEARCH-001 — Multi-Criteria Search for Tourist Packages {#HU-SEARCH-001}

**Epic:** EP-003

**As** a tourist interested in visiting Huila,
**I want** to filter tourist offerings by municipality, price range, and type of activity
**so that** I can find experiences tailored to my preferences.

**Acceptance Criteria:**

Scenario 1: Search with matching results
Given that the tourist selects the municipality "San Agustín" and the type "Archaeological"
When performing the search
Then the system displays only the active plans that meet both criteria

Scenario 2: Search without matches
Given that the tourist applies filters where no plans are registered
When executing the query
Then the system displays the message "No plans were found that match the selected filters"

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (search.query.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (GET /api/v1/plans/search)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Frontend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-CATALOG-001] |
| Affected service(s) | [catalog-service] |

---

### HU-INVENT-001 — Calendar and Availability Management {#HU-INVENT-001}

**Epic:** EP-003

**As** a travel agency,
**I want** to set a limit on the number of people allowed per operating date
**so that** the system automatically controls availability and prevents overbooking.

**Acceptance Criteria:**

Scenario 1: Successful allocation of slots by date
Given that the agency selects a date range and allocates a maximum of 15 people per day
When the configuration is saved
Then the system initializes the inventory with 15 slots available per date

Scenario 2: Slots sold out on a specified date
Given that confirmed reservations have filled all 15 slots for a specific day
When a traveler attempts to check that date
Then the system marks the day as SOLD OUT and blocks the selection

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (inventory.capacity.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PUT /api/v1/plans/{id}/inventory)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-CATALOG-001] |
| Affected service(s) | [inventory-service] |      

---

### HU-INVENT-002 — Blocking Dates Due to Contingency or Operational Issues {#HU-INVENT-002}

**Epic:** EP-003

**As** a travel agency,
**I want** to manually disable specific days on a plan's calendar
**so that** no requests are received on maintenance dates or non-operational holidays.

**Acceptance Criteria:**

Scenario 1: Manual Date Blocking
Given that the agency selects a date range and checks "Block for operational reasons"
When they confirm the action
Then the system changes the status of those days to BLOCKED in the public calendar

Scenario 2: Attempted Blocking with Active Reservations
Given that there are pending requests for the dates to be blocked
When the agency executes the block
Then the system warns them that they must reject or reschedule the previous requests

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (inventory.block.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/plans/{id}/block-dates)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-INVENT-001] |
| Affected service(s) | [inventory-service] |      

---

### HU-BOOK-001 — Tourist Plan Reservation Request {#HU-BOOK-001}

**Epic:** EP-003

**As** a tourist
**I want** to request a reservation by selecting the date, number of travelers, and my contact information
**so that** I can secure my spot in the desired tourist experience.

**Acceptance Criteria:**

Scenario 1: Successful Booking with Available Spaces
Given that the tourist selects a date with sufficient space in the inventory
When they submit their booking details
Then the system generates a unique booking code, temporarily deducts the available space, and leaves the booking in PENDING status

Scenario 2: Booking Attempt with Insufficient Spaces
Given that the tourist attempts to book 5 spaces for a date that only has 2 available
When they click "Request Booking"
Then the system prevents the transaction and displays the message "Only 2 spaces are available for this date"

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (booking.create.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/bookings)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 8 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-INVENT-001, HU-SEARCH-001] |
| Affected service(s) | [booking-service, inventory-service] |      

---

### HU-BOOK-002 — Agency Reservation Management and Confirmation {#HU-BOOK-002}

**Epic:** EP-003

**As** a travel agency,
**I want** to review received reservation requests and confirm their payment/status
**so that** I can ensure the logistics of the trip.


Acceptance Criteria:

Scenario 1: Booking Confirmation by the Agency
Given that the agency verifies the payment confirmation for a booking in PENDING status
When they press "Confirm Booking"
The booking then changes to APPROVED status and an email with the voucher is sent to the traveler

Scenario 2: Booking Rejection or Cancellation
Given that the agency does not receive payment within the deadline
When they press "Reject Booking"
Then the system changes the booking to CANCELLED and returns the slots to inventory

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (booking.status.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PATCH /api/v1/bookings/{id}/status)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Must Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-BOOK-001] |
| Affected service(s) | [booking-service, inventory-service, notification-service] |      


---

### HU-BOOK-003 — Tourist Booking History and Inquiry {#HU-BOOK-003}

**Epic:** EP-003

**As** a tourist
**I want** to check the status of my booking by entering my unique code and email address
**so that** I can verify the confirmation and details of my trip.


**Acceptance Criteria:**

Scenario 1: Successful Booking Inquiry
Given that the traveler enters their alphanumeric booking code and correct email address
When they press "Check"
Then the system displays the complete details of the plan, date, and current status

Scenario 2: Inquiry with Incorrect Data
Given that the combination of code and email address does not match any record
When they execute the search
Then the system displays the alert "Booking Not Found"

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (booking.lookup.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/bookings/lookup)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 2 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 3 |
| Assigned to | [Frontend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-BOOK-001] |
| Affected service(s) | [booking-service] |      

---

### HU-ADMIN-001 — General Control Statistical Panel {#HU-ADMIN-001}

**Epic:** EP-004

**As** HTE administrator
**I want** to visualize a consolidated dashboard with metrics on reservations, active agencies, and plans
**so that** I can make governance decisions and promote tourism in the region.

**Acceptance Criteria:**

Scenario 1: Administrative Dashboard Display
Given that the administrator logs in and accesses the main panel
When the interface loads
Then the system calculates and displays the total confirmed bookings, estimated revenue, and agencies in ACTIVE status

Scenario 2: Filtering by Date Range
Given that the administrator selects the last quarter
When the filter is applied
Then the dashboard metrics are recalculated to reflect only the activity of that period

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (admin.metrics.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (GET /api/v1/admin/dashboard)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 5 ] |
| Priority | [Should Have] |
| Target sprint | Sprint 4 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-AUTH-002, HU-BOOK-002] |
| Affected service(s) | [admin-service] |      

---

### HU-REVIEW-001 — Rating and Review of Tourist Services {#HU-REVIEW-001}

**Epic:** EP-004

**As** a tourist who completed a trip,
**I want** to assign a rating from 1 to 5 stars and post a review of the experience
**so that** other users know the quality of service offered by the agency.

**Acceptance Criteria:**

Scenario 1: Verified Review Post
Given that the traveler has a booking code in APPROVED and completed status
When they submit their 5-star rating and a comment
Then the system registers the review in PENDING status for moderation

Scenario 2: Attempted Rating without a Prior Booking
Given that a user attempts to review a plan without a completed booking
When they attempt to submit the form
Then the system denies the registration, indicating that a confirmed booking is required

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (review.submit.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (POST /api/v1/reviews)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Could Have] |
| Target sprint | Sprint 4 |
| Assigned to | [Frontend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-BOOK-002] |
| Affected service(s) | [review-service] |      

---

### HU-REVIEW-002 — Review Moderation by Management {#HU-REVIEW-002}

**Epic:** EP-004

**As** the HTE system administrator,
**I want** to audit the comments submitted by tourists
**so that** I can reject content containing offensive or inappropriate language.

**Acceptance Criteria:**

Scenario 1: Review Approval
Given that the administrator reviews a respectful review in PENDING status
When they press "Approve"
The review then moves to PUBLISHED status and updates the plan average

Scenario 2: Rejection for Inappropriate Content
Given that a review violates community guidelines
When the administrator presses "Reject"
The review then moves to REJECTED status and is not displayed in the catalog

## Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (review.moderation.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (PATCH /api/v1/reviews/{id}/moderation)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Could Have] |
| Target sprint | Sprint 4 |
| Assigned to | [Backend Lead] |
| Status | [ Ready ] |
| Dependencies | [HU-REVIEW-OO1] |
| Affected service(s) | [review-service, admin-service] |      

---

### HU-REPORT-001 — Exporting Operational Reports for Agencies {#HU-REPORT-001}

**Epic:** EP-004

**As** a travel agency,
**I want** to download reports in Excel/CSV format with a list of reservations by date
**so that** I can coordinate transportation logistics and passenger guides.

**Acceptance Criteria:**

Scenario 1: Successful download of operational report
Given that the agency selects a plan and a date range with confirmed bookings
When they click "Export Report (CSV/Excel)"
Then the system generates and downloads a structured file with the list of attendees

Scenario 2: Generation without available data
Given that the agency requests the report for dates with no registered bookings
When they request the export
Then the system reports that there is no data to generate the document

##Definition of Done:

- [ ] Code reviewed and approved
- [ ] Unit tests written (report.export.spec.ts)
- [ ] Acceptance criteria verified
- [ ] API contract updated (GET /api/v1/reports/bookings/export)
- [ ] Deployed to staging

| Field | Value |
|-------|-------|
| Story Points | [ 3 ] |
| Priority | [Could Have] |
| Target sprint | Sprint 4 |
| Assigned to | [Fullstack Developer] |
| Status | [ Ready ] |
| Dependencies | [HU-BOOK-002] |
| Affected service(s) | [booking-service, report-service] |      

## Correlations
Non-functional requirements → 04-requirements/non-functional.md
Traceability matrix → 04-requirements/traceability-matrix.md
API contracts derived from these HUs → 07-api/contracts/openapi/

---




