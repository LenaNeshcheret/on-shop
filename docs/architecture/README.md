# Architecture Flow Set

This folder contains the end-to-end architecture flows for the on-shop platform.

Current technology assumptions:

- `pricing-service` is implemented in Go.
- `search-service` is implemented in Java/Spring Boot.

## Files

- [flow-discovery.md](./flow-discovery.md): product discovery and search indexing flows
- [flow-cart-pricing.md](./flow-cart-pricing.md): add-to-cart and pricing refresh flow
- [flow-checkout.md](./flow-checkout.md): checkout and order creation flow
- [flow-fulfillment-tracking.md](./flow-fulfillment-tracking.md): fulfillment, shipment, and live tracking flow
- [checkout-sequence.puml](./checkout-sequence.puml): standalone PlantUML source for the detailed checkout sequence

Each flow markdown file links to its own standalone `.puml` source files for the diagrams it references.

Service-specific summaries and sequence diagrams now live in `services/*/README.md`.

## Recommended Reading Order

1. Read the end-to-end flows in this order:
   `flow-discovery.md` -> `flow-cart-pricing.md` -> `flow-checkout.md` -> `flow-fulfillment-tracking.md`
2. Use [checkout-sequence.puml](./checkout-sequence.puml) when you need the standalone checkout sequence source.
