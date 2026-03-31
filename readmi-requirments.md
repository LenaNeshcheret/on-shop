# Service Requirements And Implementation Order

This document defines:

1. Functional requirements per service.
2. Non-functional requirements per service.
3. Recommended implementation order.
4. Whether API-first is a good strategy.

## Global Rules (Apply To All Services)

- Every external API is REST and versioned.
- Internal synchronous calls use REST first, gRPC only for hot paths.
- Async integration uses Kafka events with idempotent consumers.
- Each service owns its own data store and schema.
- Each service exposes health, readiness, metrics, and trace correlation.
- Secrets are externalized and never stored in source control.
- State-changing endpoints require idempotency keys.

## Service Requirements

## 1. edge-api

### Functional Requirements

- Expose a single public entry point for clients.
- Route requests to internal services.
- Validate auth and propagate identity/context to downstream services.
- Provide API versioning and standardized error responses.

### Non-Functional Requirements

- Keep gateway overhead low (target p95 under 150ms without downstream time).
- Stateless design for horizontal scaling.
- Enforce rate limits and request size limits.
- Do not own business data.

## 2. customer-service

### Functional Requirements

- Manage customer profile, addresses, and preferences.
- Provide customer summary needed by checkout and order flows.
- Support update and retrieval of default shipping and billing data.

### Non-Functional Requirements

- Use PostgreSQL as source of truth.
- Protect PII with encryption in transit and at rest.
- Support audit trail for profile and address changes.
- Target p95 read latency under 120ms for profile lookups.

## 3. catalog-service

### Functional Requirements

- Manage products, categories, attributes, and product status.
- Provide product details to browsing flows.
- Publish product change events for search indexing.

### Non-Functional Requirements

- Use MongoDB as product source of truth.
- Support schema evolution for category-specific attributes.
- Guarantee event publishing for product changes.
- Target p95 read latency under 200ms.

## 4. pricing-service (Go)

### Functional Requirements

- Return deterministic price quotes for cart lines.
- Evaluate promotions, coupons, taxes, and discounts.
- Return quote breakdown and expiration timestamp.
- Publish price-change events for downstream read models.

### Non-Functional Requirements

- Keep quote latency low (target p95 under 120ms).
- Ensure deterministic output for same input and rule version.
- Version pricing rules and keep rule-change audit history.
- Support idempotent quote requests.

## 5. inventory-service

### Functional Requirements

- Track stock per SKU and location.
- Reserve inventory for checkout.
- Release or commit reservations after checkout result.
- Publish stock change events.

### Non-Functional Requirements

- Prevent overselling under concurrent reservations.
- Use reservation TTL and cleanup jobs.
- Target p95 reserve/release latency under 150ms.
- Provide strong consistency for reservation writes.

## 6. search-service

### Functional Requirements

- Serve search results with filters, sorting, and facets.
- Consume product, price, and stock events to update index.
- Support autocomplete or suggestions.

### Non-Functional Requirements

- Elasticsearch is read model only, never source of truth.
- Support reindex operations without downtime.
- Keep indexing lag bounded (target under 2 minutes).
- Target p95 query latency under 250ms.

## 7. cart-service

### Functional Requirements

- Add, update, remove items in customer cart.
- Return cart snapshot for pricing and checkout.
- Support saved-for-later behavior.

### Non-Functional Requirements

- Use MongoDB document model for mutable carts.
- Target p95 write latency under 120ms.
- Handle concurrent cart updates safely.
- Support cart expiration/cleanup for stale carts.

## 8. payment-service

### Functional Requirements

- Authorize, capture, void, and refund payments.
- Integrate with one or more payment providers.
- Publish payment status events.

### Non-Functional Requirements

- Require idempotency keys for all money operations.
- Apply strict timeout, retry, and circuit-breaker policy per provider.
- Minimize PCI scope and never store raw card data.
- Keep full audit trail of all payment state transitions.

## 9. order-service

### Functional Requirements

- Create immutable order snapshot from successful checkout.
- Store and expose order status history.
- Publish order lifecycle events.

### Non-Functional Requirements

- Use PostgreSQL for durable transactional writes.
- Order snapshot totals and lines are immutable after creation.
- Guarantee event publication for new orders and status changes.
- Target p95 order read latency under 150ms.

## 10. checkout-service

### Functional Requirements

- Own checkout session state and lifecycle.
- Orchestrate cart snapshot, re-pricing, inventory reserve, payment auth, and order create.
- Handle retries, duplicate requests, and compensation on failures.
- Return safe retry status to clients.

### Non-Functional Requirements

- Enforce API idempotency with idempotency keys.
- Keep durable state transitions for each checkout step.
- Apply strict timeout budgets for downstream calls.
- Provide compensating actions for partial failures.

## 11. fulfillment-service

### Functional Requirements

- Consume order placed events and create shipment tasks.
- Integrate with warehouse and carrier updates.
- Publish shipment created and shipment status events.

### Non-Functional Requirements

- Process events idempotently with retry and dead-letter handling.
- Keep shipment processing async and decoupled from checkout path.
- Handle duplicate carrier webhook callbacks safely.
- Keep order-to-shipment creation lag bounded (target under 5 minutes).

## 12. notification-service

### Functional Requirements

- Consume order and shipment events.
- Push real-time updates via WebSocket.
- Support email/SMS notifications as asynchronous channels.

### Non-Functional Requirements

- Support at-least-once internal delivery with deduplication by event ID.
- Target near real-time push latency (target under 3 seconds from event).
- Track delivery status by channel.
- Apply retry policy and fallback per channel provider.

## Recommended Implementation Order

Implement in dependency order, but deliver in vertical slices:

1. `catalog-service`
2. `pricing-service`
3. `inventory-service`
4. `search-service`
5. `cart-service`
6. `customer-service`
7. `payment-service`
8. `order-service`
9. `checkout-service`
10. `fulfillment-service`
11. `notification-service`
12. `edge-api` hardening and final external API consolidation

Notes:

- Build a minimal `edge-api` skeleton from day one, then expand it per slice.
- This order follows flow dependencies: discovery -> cart/pricing -> checkout -> post-order async.

## API-First Recommendation

Yes, defining APIs first is a good idea for this architecture.

Recommended approach:

1. Define contract for one vertical slice, not all services in full detail at once.
2. Include REST endpoints, request/response schema, error model, and idempotency behavior.
3. Define event contracts (Kafka topics, payload versions, keys) together with APIs.
4. Generate or enforce contract checks in CI before implementation.
5. Implement stubs/mocks so dependent teams can integrate early.

Avoid:

- Big upfront API design for every edge case across all services before any running slice.
- Publishing unstable contracts without versioning rules.

Best balance:

- Contract-first by slice, then implement and validate quickly.
