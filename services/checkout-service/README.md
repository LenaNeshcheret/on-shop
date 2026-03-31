# checkout-service

`checkout-service` owns the short-lived workflow that turns a mutable cart into a valid immutable order. It is the orchestration boundary for repricing, stock reservation, payment authorization, idempotency, retries, and compensation.

## Main Info

- Runtime: Java / Spring Boot
- Modules: `api` for the public Java contract marker, `impl` for the Spring Boot runtime
- Storage: PostgreSQL
- Primary callers: `api-gateway`
- Primary downstreams: `cart-service`, `pricing-service`, `inventory-service`, `payment-service`, `order-service`, Kafka checkout events
- Owns: checkout attempts or sessions, idempotency state, compensation state, failure reasons
- Typical lifecycle: `INITIATED`, `PRICED`, `INVENTORY_RESERVED`, `PAYMENT_AUTHORIZED`, `ORDER_CREATED`, `FAILED`, `EXPIRED`
- Does not own: cart persistence, pricing rules, inventory truth, payment ledger, shipment execution, durable order history

## Primary Sequence

```plantuml
@startuml
hide footbox
participant "api-gateway" as Edge
participant "checkout-service" as Checkout
participant "cart-service" as Cart
participant "pricing-service" as Pricing
participant "inventory-service" as Inventory
participant "payment-service" as Payment
participant "order-service" as Order
queue "Kafka" as Kafka

Edge -> Checkout: POST /checkouts/{id}/place-order
Checkout -> Cart: load cart snapshot
Cart --> Checkout: cart lines
Checkout -> Pricing: quote(cart, address, coupon)
Pricing --> Checkout: totals + quote expiry
Checkout -> Inventory: reserve(items)
Inventory --> Checkout: reservationId
Checkout -> Payment: authorize(amount)
Payment --> Checkout: authorizationId
Checkout -> Order: create immutable order
Order --> Checkout: orderId
Checkout -> Kafka: CheckoutCompleted
Checkout --> Edge: 201 Created + orderId
@enduml
```

## Boundary Notes

- Keep temporary checkout state here instead of polluting `order-service` with pre-order statuses.
- Use this service to detect duplicate requests, manage retries, and coordinate compensation.
- On pricing mismatch, fail before inventory reservation and payment authorization.
- On payment failure, release inventory reservation.
- On order creation failure after payment authorization, void payment authorization, release reservation, and expose a safe retry state.
