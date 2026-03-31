# on-shop

`on-shop` is the parent workspace for the marketplace platform. The root project is no longer an application. It owns:

- parent Gradle configuration for all Java services
- shared architecture and design documentation
- deployment documentation and future manifests
- service layout and dependency rules

The root `src` application has been removed on purpose. Runtime code now lives in dedicated service modules under `services/`.

## Technology Baseline

- Java services: Spring Boot `4.0.3`
- Java toolchain target: `25`
- Gradle wrapper target: `9.4.0`
- Pricing service: planned as Go and kept outside the Java multi-module build

## Module Structure

Each Java service is split into two Gradle modules:

- `:services:<service-name>:api`
- `:services:<service-name>:app`

Rule:

- one service may depend on another service only through that other service's `:api` module
- direct dependencies on another service's `:app` module are forbidden by the build

Current Java services:

- `edge-api`
- `customer-service`
- `catalog-service`
- `search-service`
- `cart-service`
- `inventory-service`
- `checkout-service`
- `order-service`
- `payment-service`
- `fulfillment-service`
- `notification-service`

Planned non-Java service:

- `pricing-service` in Go under the same repository parent

Each service directory contains a local `README.md` with a service summary and a service-centered sequence diagram.

## Functional Requirements

### Marketplace Core

- expose API-only capabilities for clients and partner systems
- support browsing, search, cart, checkout, orders, payments, fulfillment, and notifications
- support customer profile and address management
- support product catalog management with flexible product attributes
- support order placement with final repricing and inventory reservation
- support real-time order and shipment updates

### Service Responsibilities

- `edge-api`: public REST entrypoint, versioning, auth propagation, request shaping
- `customer-service`: profiles, addresses, preferences
- `catalog-service`: product master data and category-driven attributes
- `search-service`: search query and indexing logic on the Java side
- `cart-service`: mutable shopping carts and saved items
- `inventory-service`: stock levels and reservation management
- `checkout-service`: orchestration, idempotency, temporary checkout state, compensation
- `order-service`: immutable order snapshots and durable order lifecycle
- `payment-service`: payment authorization, capture, refund, provider integration
- `fulfillment-service`: shipment creation, shipment status, warehouse coordination
- `notification-service`: WebSocket push and later message delivery channels
- `pricing-service` (Go): price calculation, promotions, coupon validation, quote generation

### Data and Integration Rules

- PostgreSQL is the transactional source of truth for customer, inventory, checkout, order, payment, and fulfillment domains
- MongoDB is used where the document model is a better fit, especially catalog and cart
- Elasticsearch is a read model for search only, never the system of record
- REST is the external API style
- gRPC is reserved for latency-sensitive internal calls
- Kafka is used for domain events and asynchronous workflows
- WebSockets are used for live client-facing status updates

## Non-Functional Requirements

### Architecture

- no shared application database tables across service boundaries
- only `:api` modules may be referenced across services
- root project stays free of runtime business code
- pricing remains isolated as a Go service instead of a Java module

### Reliability and Consistency

- checkout must be idempotent
- state-changing workflows must support compensation or retry
- Kafka consumers must be idempotent
- search and notification flows may be eventually consistent

### Security

- every public endpoint must be authenticated and authorized
- secrets must be externalized from source control
- internal service calls must propagate correlation and security context

### Operability

- every service must expose health and info endpoints
- structured logging and trace correlation are required
- metrics must be available for Kubernetes scraping
- deployment topology must support Docker Compose for local work and Kubernetes for runtime

### Performance and Scalability

- scale services independently
- keep synchronous request paths short and explicit
- move expensive side effects out of the checkout path
- treat search as a separately scalable read-heavy service

### Delivery and Maintainability

- services must be independently testable
- build conventions must be centralized in the parent workspace
- architecture documentation must live with the codebase
- deployment definitions must live in the root parent project

## Repository Layout

```text
on-shop/
├── build.gradle
├── settings.gradle
├── deployment/
├── docs/
│   └── architecture/
├── services/
│   ├── edge-api/
│   │   ├── api/
│   │   └── app/
│   ├── customer-service/
│   │   ├── api/
│   │   └── app/
│   ├── ...
│   └── pricing-service/
└── gradle/
```

## Notes

- the local machine currently has JDK `21`, while the build is configured for Java `25`
- Gradle toolchain auto-download is enabled, but downloading a JDK or Gradle distribution was not verified in this session
- the deployment directories are placeholders for the next step
