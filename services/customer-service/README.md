# customer-service

`customer-service` owns customer profile data, saved addresses, and customer preferences. It should not own authentication credentials when identity is provided by an external identity provider.

## Main Info

- Runtime: Java / Spring Boot
- Modules: `api` for the public Java contract marker, `impl` for the Spring Boot runtime
- Storage: PostgreSQL
- Primary callers: `api-gateway`, `checkout-service`, `order-service`
- Primary downstreams: PostgreSQL
- Owns: customer profiles, addresses, preferences
- Does not own: identity-provider credentials or checkout/order workflow state

## Primary Sequence

```plantuml
@startuml
hide footbox
actor Client
participant "api-gateway" as Edge
participant "customer-service" as Customer
database "Postgres" as Pg

Client -> Edge: GET /customers/me
Edge -> Customer: loadProfile(customerId)
Customer -> Pg: select profile, addresses, preferences
Pg --> Customer: customer aggregate
Customer --> Edge: customer response
Edge --> Client: normalized API response
@enduml
```
