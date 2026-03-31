# cart-service

`cart-service` owns mutable shopping carts and saved-for-later state. Cart state is indicative and user-editable; it is not the same thing as a durable order.

## Main Info

- Runtime: Java / Spring Boot
- Modules: `api` for the public Java contract marker, `app` for the Spring Boot runtime
- Storage: MongoDB
- Primary callers: `edge-api`, `checkout-service`
- Primary downstreams: MongoDB
- Owns: active cart documents, line selections, saved items
- Does not own: final pricing decisions, order history, or payment state

## Primary Sequence

```plantuml
@startuml
hide footbox
actor Client
participant "edge-api" as Edge
participant "cart-service" as Cart
database "MongoDB" as Mongo

Client -> Edge: POST /cart/items
Edge -> Cart: addItem(customerId, sku, qty)
Cart -> Mongo: upsert cart document
Mongo --> Cart: updated cart
Cart --> Edge: cart response
Edge --> Client: cart updated
@enduml
```
