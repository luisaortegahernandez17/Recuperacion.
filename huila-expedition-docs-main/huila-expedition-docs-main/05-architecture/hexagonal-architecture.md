# Hexagonal Architecture (Ports and Adapters)

> The hexagonal architecture, proposed by Alistair Cockburn, organizes the system so that the **business domain is completely independent** of external technologies. The database, web framework, and email or payment services are interchangeable details. What is truly crucial is the business logic of **Huila Travel Expedition**, which resides at the heart of the application.


> > **Note on the Technical Stack:** Although the principles of this architecture are universal, the implementation and code structure presented in this document are tailored specifically to the project's technical specifications:
> - **Language and Framework:** PHP 8.2+ with Laravel 10+
> - **Database & Cache:** MySQL 8.0 and Redis
> - **Template Engine / API:** Blade / Tailwind CSS and REST API

### Hexagonal Architecture in Huila Travel Expedition:

                  ┌─────────────────────────────────────────┐
                  │          PRIMARY ADAPTERS               │
                  │        (Driving / Input)                │
                  └────────────────────┬────────────────────┘
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            ▼                          ▼                          ▼
     [HTTP Controller]           [Artisan CLI]          [PHPUnit / Pest Test]
            │                          │                          │
            └──────────────────────────┼──────────────────────────┘
                                       │
                                       ▼
                             ┌───────────────────┐
                             │   DRIVING PORT    │  (Use Case Interface)
                             └─────────┬─────────┘  e.g., CreateBookingUseCase
                                       │
                                       ▼
                   ┌───────────────────────────────────────┐
                   │               DOMAIN                  │
                   │      (Huila Travel Expedition)        │
                   │                                       │
                   │   Entities: Booking, Agency, Plan     │
                   │   Rules: RNT Validation, Capacity     │
                   │    * No Laravel / Eloquent logic *    │
                   └───────────────────┬───────────────────┘
                                       │
                                       ▼
                             ┌───────────────────┐
                             │    DRIVEN PORT    │  (Required Interface)
                             └─────────┬─────────┘  e.g., BookingRepositoryPort
                                       │
            ┌──────────────────────────┼──────────────────────────┐
            ▼                          ▼                          ▼ 
    [Eloquent Repository]         [Wompi / PayU]             [SMTP Mailer]
                                                         
            └──────────────────────────┼──────────────────────────┘
                                       │
                  ┌────────────────────┴────────────────────┐
                  │          SECONDARY ADAPTERS             │
                  │        (Driven / Output)                │
                  └─────────────────────────────────────────┘

## Folder Structure

Within the Laravel app/ folder, we will organize the bounded contexts or main aggregates defined in the SRS (Booking, Agency, Plan, Review): 

```
app/
├── Domain/                          # The Hexagon — Zero dependencies
│   ├── Booking/                     # Booking Aggregate
│   │   ├── Model/
│   │   │   ├── Booking.php          # Entity / Aggregate Root
│   │   │   ├── BookingId.php        # ID Value Object
│   │   │   ├── BookingStatus.php    # Enum / Value Object
│   │   │   └── PassengersCount.php  # Validation Value Object
│   │   ├── Events/
│   │   │   ├── BookingRequested.php # Domain Event
│   │   │   └── BookingApproved.php
│   │   ├── Services/
│   │   │   └── AvailabilityChecker.php
│   │   └── Ports/
│   │       ├── In/
│   │       │   └── CreateBookingInputPort.php
│   │       └── Out/
│   │           ├── BookingRepositoryPort.php
│   │           └── NotificationPublisherPort.php
│   ├── Agency/                      # Agency Aggregate
│   │   ├── Model/
│   │   │   ├── Agency.php
│   │   │   └── RNTNumber.php
│   │   └── Ports/
│   │       └── Out/
│   │           └── AgencyRepositoryPort.php
│   ├── Plan/                        # Tour Plan Aggregate
│   └── Shared/                      # Shared Value Objects & Exceptions
│       ├── ValueObjects/
│       │   └── Money.php
│       └── Exceptions/
│           └── DomainException.php
│
├── Application/                     # Use Cases
│   ├── Booking/
│   │   ├── CreateBookingUseCase.php
│   │   └── Dtos/
│   │       ├── CreateBookingRequest.php
│   │       └── CreateBookingResponse.php
│   └── Agency/
│       └── RegisterAgencyUseCase.php
│
└── Infrastructure/                  # External Layer (Laravel/Eloquent)
    ├── Adapters/
    │   ├── In/                      # Primary Adapters
    │   │   ├── Http/
    │   │   │   ├── Controllers/
    │   │   │   │   └── BookingController.php
    │   │   │   └── Requests/
    │   │   │       └── StoreBookingHttpRequest.php
    │   │   └── Console/
    │   │       └── CancelExpiredBookingsCommand.php
    │   └── Out/                     # Secondary Adapters
    │       ├── Persistence/
    │       │   ├── Eloquent/
    │       │   │   ├── Models/
    │       │   │   │   └── BookingModel.php
    │       │   │   └── Mappers/
    │       │   │       └── BookingMapper.php
    │       │   └── EloquentBookingRepository.php
    │       ├── Mail/
    │       │   └── SmtpNotificationAdapter.php
    │       └── Payment/
    │           └── WompiPaymentAdapter.php
    └── Providers/
        └── HexagonalBindingsServiceProvider.php
```

###  Port Specification

Ports are abstract interfaces in PHP. They are defined by the domain and implemented by the infrastructure.

## Driving Port (Input Port)
Defines the domain's API from the perspective of incoming requests (e.g., from the web or API).

```php
<?php

namespace App\Domain\Booking\Ports\In;

use App\Application\Booking\Dtos\CreateBookingRequest;
use App\Application\Booking\Dtos\CreateBookingResponse;

interface CreateBookingInputPort
{ 
public function execute(CreateBookingRequest $request): CreateBookingResponse;
}
```

## Driven Port (Output Port)
Defines what the domain requires from the outside world (persistence, email sending, payment gateways).

```php
<?php

namespace App\Domain\Booking\Ports\Out;

use App\Domain\Booking\Model\Booking;
use App\Domain\Booking\Model\BookingId;

interface BookingRepositoryPort
{ 
public function save(Booking $booking): void; 
public function findById(BookingId $id): ?Booking; 
public function getAvailableCapacityForDate(string $planId, \DateTimeImmutable $date): int;
}

<? php

namespace App\Domain\Booking\Ports\Out;

use App\Domain\Booking\Model\Booking;
use App\Domain\Booking\Model\BookingId;

interface BookingRepositoryPort
{ 
public function save(Booking $booking): void; 
public function findById(BookingId $id): ?Booking; 
public function getAvailableCapacityForDate(string $planId, \DateTimeImmutable $date): int;
}
```
### Adapters

## Primary Adapter — HTTP Controller (Laravel)
Receives the HTTP request, maps the data to an application DTO, and calls the input port. It does not contain business logic.

```php
<?php

namespace App\Infrastructure\Adapters\In\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Infrastructure\Adapters\In\Http\Requests\StoreBookingHttpRequest;
use App\Application\Booking\Dtos\CreateBookingRequest;
use App\Domain\Booking\Ports\In\CreateBookingInputPort;
use Illuminate\Http\JsonResponse;

class BookingController extends Controller
{ 
// We inject the Interface (Entry Port), not the concrete implementation 
public function __construct( 
private readonly CreateBookingInputPort $createBookingUseCase 
) {} 

public function store(StoreBookingHttpRequest $request): JsonResponse 
{ 
$dto = new CreateBookingRequest( 
planId: $request->validated('plan_id'), 
touristId: $request->user()->id, 
travelDate: new \DateTimeImmutable($request->validated('travel_date')), 
passengers: $request->validated('passengers'), 
termsAccepted: $request->validated('terms_accepted') 
); 

$response = $this->createBookingUseCase->execute($dto); 

return response()->json([ 
'message' => 'Reservation created successfully', 
'booking_id' => $response->bookingId, 
'status' => $response->status 
], 201); 
}
 }
```
## Secondary Adapter — Eloquent Repository
Implements the outbound port. Translates domain entities to MySQL database records using Eloquent and Mappers

```php
<?php

namespace App\Infrastructure\Adapters\Out\Persistence;

use App\Domain\Booking\Ports\Out\BookingRepositoryPort;
use App\Domain\Booking\Model\Booking;
use App\Domain\Booking\Model\BookingId;
use App\Infrastructure\Adapters\Out\Persistence\Eloquent\Models\BookingModel;
use App\Infrastructure\Adapters\Out\Persistence\Eloquent\Mappers\BookingMapper;

class EloquentBookingRepository implements BookingRepositoryPort
{ 
public function save(Booking $booking): void 
{ 
// Mapper converts the Domain Entity to an Eloquent array/model 
$data = BookingMapper::toPersistence($booking); 

BookingModel::query()->updateOrCreate( 
['id' => $booking->getId()->getValue()], 
$data 
); 
} 

public function findById(BookingId $id): ?Booking 
{ 
$model = BookingModel::query()->find($id->getValue()); 
if (!$model) { 
return null; 
} 

return BookingMapper::toDomain($model); 
} 

public function getAvailableCapacityForDate(string $planId, \DateTimeImmutable $date): int 
{ 
// Specific query to verify spaces in the plan calendar 
return BookingModel::query() 
->where('plan_id', $planId) 
->whereDate('travel_date', $date->format('Y-m-d')) 
->whereIn('status', ['PENDING', 'APPROVED']) 
->sum('passengers_count'); 
}
 }
```
## The Use Case (Application Service)

This is the executor of the business processes. It receives the user request, applies the system rules using the entities, and requests the ports to save or send the information.

```php
<?php

namespace App\Application\Booking;

use App\Domain\Booking\Ports\In\CreateBookingInputPort;
use App\Domain\Booking\Ports\Out\BookingRepositoryPort;
use App\Domain\Booking\Ports\Out\NotificationPublisherPort;
use App\Application\Booking\Dtos\CreateBookingRequest;
use App\Application\Booking\Dtos\CreateBookingResponse;
use App\Domain\Booking\Model\Booking;
use App\Domain\Booking\Exceptions\NoAvailableCapacityException;

class CreateBookingUseCase implements CreateBookingInputPort
{ 
public function __construct( 
private readonly BookingRepositoryPort $bookingRepository, 
private readonly NotificationPublisherPort $notificationPublisher 
) {} 

public function execute(CreateBookingRequest $request): CreateBookingResponse 
{ 
// 1. Validate availability rules 
$currentBooked = $this->bookingRepository->getAvailableCapacityForDate( 
$request->planId, 
$request->travelDate 
); 

// 2. Creation of the Aggregate in Domain (Applies HTE / RF10 / RF20 business rules) 
$booking = Booking::create( 
planId: $request->planId, 
touristId: $request->touristId, 
travelDate: $request->travelDate, 
passengers: $request->passengers, 
termsAccepted: $request->termsAccepted, 
currentCapacityUsed: $currentBooked 
); 

// 3. Persist through the Port 
$this->bookingRepository->save($booking); 

// 4. Publish Domain Notification/Event 
foreach ($booking->pullDomainEvents() as $event) { 
$this->notificationPublisher->notifyBookingCreated($event); 
} 

return new CreateBookingResponse( 
bookingId: $booking->getId()->getValue(), 
status: $booking->getStatus()->getValue() 
); 
}
 }
```
## The Dependency Rule
It functions as the system connector. It teaches Laravel which actual tool (Adapter) to give to the application each time it requests a working contract (Port).

```php
<?php
namespace App\Infrastructure\Providers;

use Illuminate\Support\ServiceProvider;

// Interfaces (Ports)
use App\Domain\Booking\Ports\In\CreateBookingInputPort;
use App\Domain\Booking\Ports\Out\BookingRepositoryPort;
use App\Domain\Booking\Ports\Out\NotificationPublisherPort;

// Implementations (Use Cases & Adapters)
use App\Application\Booking\CreateBookingUseCase;
use App\Infrastructure\Adapters\Out\Persistence\EloquentBookingRepository;
use App\Infrastructure\Adapters\Out\Mail\SmtpNotificationAdapter;

class HexagonalBindingsServiceProvider extends ServiceProvider
{ 
public function register(): void 
{ 
// Link Entry Port to Use Case 
$this->app->bind( 
CreateBookingInputPort::class, 
CreateBookingUseCase::class 
); 

// Link Output Ports with Infrastructure Adapters 
$this->app->bind( 
BookingRepositoryPort::class, 
EloquentBookingRepository::class 
); 

$this->app->bind( 
NotificationPublisherPort::class, 
SmtpNotificationAdapter::class 
);

    }
}
```
## Advantages for Automated Testing (TDD in HTE)

Thanks to architecture independence, the development team can efficiently apply TDD:

- Isolation: Huila Travel Expedition's business rules are tested without launching Laravel or MySQL.
- Speed: Unit tests run in milliseconds, facilitating rapid bug detection.
- Reliability: Ensures that technical changes do not alter the domain's behavior.

```php
  <?php

namespace Tests\Unit\Domain\Booking;

use PHPUnit\Framework\TestCase;
use App\Domain\Booking\Model\Booking;
use App\Domain\Booking\Exceptions\TermsNotAcceptedException;

class BookingTest extends TestCase
{ 
public function test_cannot_create_booking_without_accepting_terms(): void 
{ 
$this->expectException(TermsNotAcceptedException::class); 

Booking::create( 
planId: 'plan-123', 
touristId: 'user-456', 
travelDate: new \DateTimeImmutable('2026-10-15'), 
passengers: 2, 
termsAccepted: false, // RF20/RNF9 rule violation 
currentCapacityUsed: 0 
); 
}
 }
```
## Hexagonal Architecture Checklist for HTE
Before submitting a Pull Request to the project's GitHub repository, verify the following:

- [ ] The Domain/ folder does not contain any use Illuminate\... or references to Eloquent.
- [ ] All database operations are performed through interfaces located in Domain/Ports/Out/.
- [ ] Controllers in Infrastructure/Adapters/In/Http/ only call incoming ports (Ports/In/).
- [ ] Eloquent mappings to entities are performed using mappers within Infrastructure/.
- [ ] Pure unit tests exist for the Domain entities (Booking, Agency, Plan).

     ## Common Mistakes (Antipatterns)

| Antipattern | Why it's bad | Solution |
|:----------- |:------------ |:-------- |
`use Illuminate\Database\Eloquent\Model` in the Domain | Couples business logic to Laravel's ORM | Define a custom Outbound Port (`Interface`) in the Domain. |
Business logic within the HTTP Controller | If the route or request changes, the business rule breaks | Move the business rule to the corresponding Entity/Aggregate. |
Repository returning an Eloquent Model or Array instead of an Entity | The Domain loses the ability to validate its rules and invariants | Use a `Mapper` to rebuild the Domain Entity. |
Use case with more than 5 injected dependencies | The Use Case is taking on too many responsibilities | Break it down into smaller, more specific Use Cases. |
| Use of `mixed` or generic `array` objects in interfaces | PHP's strict typing and design contract are lost | Use explicit typing, DTOs, and value objects. |


---

## References and Correlations

- **Bounded Contexts:** `app/Domain/Booking`, `app/Domain/Agency`, `app/Domain/Plan`
- **Entities and Invariants:** Pure business rules in `app/Domain/{Context}/Model/`
- **Domain Events:** `app/Domain/{Context}/Events/` (e.g., `BookingRequested`, `BookingApproved`)
- **Dependency Injection:** Bindings registered in `app/Infrastructure/Providers/HexagonalBindingsServiceProvider.php`
- **Automated Testing (TDD):** Execution guide with PHPUnit/Pest in `tests/Unit/Domain/`
