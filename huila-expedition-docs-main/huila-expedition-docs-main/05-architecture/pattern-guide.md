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
