# Design Patterns and Microservices Guide — Huila Travel Expedition (HTE)

This document is the design pattern catalog for the Huila Travel Expedition (HTE) project.
For each pattern: when to use it, when NOT to use it, and an implementation example.
The patterns are not recipes—they are tools. Use them when the project problem requires it.

**Stack note**: The descriptions and diagrams are technology-agnostic.
The illustrative code uses PHP 8.2+ / Laravel 10+ (the project's main stack) and pseudo-TypeScript as the reference language.
Repository and specific stack guides:
> [`_stacks/node-typescript.md`]
> [`_stacks/php-laravel.md`]
> [`_stacks/python-fastapi.md`]

---

## Index
**Design Patterns (GoF and SOLID)**
1. Creational Patterns
2. Structural Patterns
3. Behavioral Patterns

**Architectural Patterns and Microservices**
4. System Decomposition
5. Communication Between Services
6. Resilience
7. Data and Consistency
8. Observability

## Design Patterns (GoF)

### 1. Factory Method
**Problem:** Creating reservation or offer records without coupling the domain logic to the specific rate type or initial state.

**When to use it:** When instantiating complex domain objects such as the Reservation or TourPlan entity, ensuring business validations before construction.

**Example domain (PHP/Laravel):**

```php
// Factory Method within the Reservation Model/Entity
class Reservation extends Model {
public static function createRequest(int $touristId, int $planId, string $date, int $people): self {
if ($people <= 0) {
throw new \InvalidArgumentException('The number of people must be greater than zero.');
}

return new self([
'tourist_id' => $touristId,
'plan_id' => $planId,
'reservation_date' => $date,
'number_of_people' => $people,
'status' => 'pending' // Guaranteed initial status
]);
}
 }
```
### 2. Builder

**Problem:** Creating a tour plan with multiple optional settings (itineraries, categories, adult/child rates, images) becomes complex.

**When to use it:** Building complex data in automated tests or generating DTOs for the experience catalog.

**Example domain**

```php
// Builder for creating Tourist Plans for testing or service
$plan = (new PlanTuristicoBuilder())
--withAgency($agencyId)
--withTitle('Tatacoa Ecological Walk')
--withLocation('Villavieja')
--withPrice('adult', 80000)
--withCategory('ecological')
---build();
```
### 3. Singleton

**Problem:** Ensuring a single instance for connection managers or global system configuration.

**When to use it:** Directly managed by Laravel's dependency container (Service Container) for external API clients (e.g., Wompi/PayU payment gateway or WhatsApp client). 

**Example domain**

```php
// Registro Singleton en AppServiceProvider de Laravel
$this->app->singleton(PasarelaPagoClient::class, function ($app) {
    return new PasarelaPagoClient(config('services.pasarela.key'));
});
```
### 4. Adapter

**Problem:** Adapting the booking engine interface to different payment gateway providers (Wompi, PayU, MercadoPago) without altering the domain code.

**When to use it:** Integration with external APIs and notification services.

**Example domain**

```php
interface PaymentPortGateway { 
public function processPayment(float $amount, array $customerdata): PaymentResponse;
}

class WompiAdapter implements PaymentPortGateway { 
public function __construct(private WompiClient $wompi) {} 

public function processPayment(float $amount, array $customerdata): PaymentResponse { 
$response = $this->wompi->createTransaction([ 
'amount_in_cents' => $amount * 100, 
'customer_email' => $customerdata['email'] 
]); 
return new PaymentResponse($response->id, $response->status === 'APPROVED'); 
}
 }
```
### 5. Decorator

**Problem:** Adding caching or audit logging layers to catalog queries without modifying the repositories.

**When to use:** Optimizing read operations for featured plans or verifying real-time availability using Redis.

**Example domain**
```php
class CachedPlanRepository implements PlanRepositoryInterface {
    public function __construct(
        private PlanRepositoryInterface $repository,
        private CacheManager $cache
    ) {}

    public function findFeatured(): array {
        return $this->cache->remember('featured_plans', 3600, function() {
            return $this->repository->findFeatured();
        });
    }
}
```
### 6. Observer (Event Bus)

**Problem:** Notify the agency, send transactional emails, and update availability when a reservation's status changes.

**When to use it:** Decoupling business events (ReservationCreated, ReservationApproved) from the core logic.

**Example domain**

```php
// Event registration and listener in Laravel
class ReservaObserver {
    public function updated(Reserva $reserva) {
        if ($reserva->wasChanged('estado') && $reserva->estado === 'aprobada') {
            event(new ReservaAprobadaEvent($reserva));
        }
    }
}
```
### 7. Strategy

**Problem:** Applying different rate calculation methods (differentiated rates for children, adults, groups, or seasons).

**When to use it:** Dynamic price calculation during the booking process.

**Example domain**

```php
interface CalculoTarifaStrategy {
    public function calcular(float $precioBase, int $cantidad): float;
}

class TarifaGrupoStrategy implements CalculoTarifaStrategy {
    public function calcular(float $precioBase, int $cantidad): float {
        $descuento = $cantidad >= 10 ? 0.15 : 0.0;
        return ($precioBase * $cantidad) * (1 - $descuento);
    }
}
```
### 8. Template Method

**Problem:** Exporting reservation and sales reports in multiple formats (PDF, Excel) while sharing the same data filtering and preparation structure.

**When to use it:** Generating system documents and reports (RF18).

**Example domain**

```php
abstract class ReportExporter {
    final public function export(array $filters) {
        $data = $this->getData($filters);
        $processedData = $this->processFormats($data);
        return $this->generateFile($processedData);
    }

    abstract protected function generateFile(array $data);
    
    protected function getData(array $filters) {
        return Reserva::whereBetween('created_at', [$filters['start'], $filters['end']])->get();
    }
}
```
## Microservices Patterns / Architecture

### Decomposition {#decomposition}

#### API Gateway

**Problem:** Tourists and agencies need to access various modules (Authentication, Bookings, Catalog, Reports) via the web or mobile application.

 ```
                     ┌─────────────────────────┐
Tourists (Web) ───▶ │                         │ ──▶ [Authentication Module]
Agencies (Panel) ──▶│  API Gateway / Router   │ ──▶ [Catalog/Plans Module]
Mobile (Future) ──▶ │                         │ ──▶ [Booking/Calendar Module]
                     └─────────────────────────┘
                               Handles:
                    - REST API routing
                    - JWT / Session authentication
                    - Rate limiting (request limits)
                    - SSL/HTTPS termination
```
**When to use it:** Essential for centralizing entry points, validating access tokens, and enforcing security (HTTPS / CORS).

**Recommended tools:** Laravel Router / NGINX / Traefik / Kong.

#### Backeend for Frontend  (BFF)       

**Problem:** The tourist-facing web application displays lightweight visual cards, whereas the agency dashboard requires dense inventory data and calendars.

```
Turista Web ──▶ [BFF Turistas]  ──▶ Servicios del sistema HTE
Panel Agencia ─▶ [BFF Agencias]  ──▶ Servicios del sistema HTE
```
