# End-to-End Flow: Checkout And Order Creation

The standalone PlantUML source for the detailed checkout sequence is available here:

- [checkout-sequence.puml](./checkout-sequence.puml)

## Core Checkout Path

Standalone PlantUML source:

- [flow-checkout-core-path.puml](./flow-checkout-core-path.puml)

## Async Side Effects

Standalone PlantUML source:

- [flow-checkout-async-side-effects.puml](./flow-checkout-async-side-effects.puml)

Checkout owns orchestration and temporary failure handling. Order owns durable history after creation.
