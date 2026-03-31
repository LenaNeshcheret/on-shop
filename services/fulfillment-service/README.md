# fulfillment-service

`fulfillment-service` owns shipment creation, shipment status, and tracking updates. It runs after order creation and coordinates warehouse-facing shipment work plus downstream shipment events.

## Main Info

- Runtime: Java / Spring Boot
- Modules: `api` for the public Java contract marker, `app` for the Spring Boot runtime
- Storage: PostgreSQL
- Primary callers: order events, carrier webhooks
- Primary downstreams: `inventory-service`, PostgreSQL, Kafka shipment events
- Owns: shipment records, shipment status, tracking updates, warehouse assignment
- Does not own: synchronous checkout flow or durable order truth

## Primary Sequence

```plantuml
@startuml
hide footbox
queue "Kafka" as Kafka
participant "fulfillment-service" as Fulfillment
participant "inventory-service" as Inventory
database "Postgres" as Pg
actor Carrier

Kafka -> Fulfillment: OrderPlaced
Fulfillment -> Fulfillment: create shipment and assign warehouse
Fulfillment -> Inventory: allocate reserved stock
Inventory --> Fulfillment: allocation confirmed
Fulfillment -> Pg: persist shipment
Pg --> Fulfillment: shipmentId
Carrier -> Fulfillment: tracking status update
Fulfillment -> Kafka: ShipmentStatusChanged
@enduml
```
