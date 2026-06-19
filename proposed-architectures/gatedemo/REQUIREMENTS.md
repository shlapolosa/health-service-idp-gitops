# gatedemo Requirements

## Use Case
Build a reusable realtime IoT telemetry pipeline named `gatedemo` using existing platform realtime ComponentDefinitions. The pipeline must ingest sensor readings over HTTP, validate them, publish valid readings to Kafka, compute a rolling average per sensor, and stream aggregates to browser clients over WebSocket.

## Services and Responsibilities

### gatedemo-platform
Provides the shared realtime platform substrate for the application, including Kafka-backed topic provisioning and realtime connection material for dependent services.

### gatedemo-ingest
Accepts sensor readings over HTTP, validates payloads, and publishes validated readings to the Kafka topic `gatedemo.validated-readings`.

### gatedemo-processor
Consumes validated readings from Kafka, computes a rolling average per sensor, and publishes aggregates to `gatedemo.sensor-averages`.

### gatedemo-gateway
Consumes aggregate updates from Kafka and exposes a browser-facing WebSocket stream on `/ws`.

## Acceptance Criteria

```acceptance
- kind:test
  id: platform-topics-provisioned
  service: gatedemo-platform
  verify: The deployed platform exposes Kafka connection material and both topics `gatedemo.validated-readings` and `gatedemo.sensor-averages` are declared for the application.

- kind:test
  id: ingest-http-accepts-readings
  service: gatedemo-ingest
  verify: An HTTP POST of a syntactically valid sensor reading to the ingest service returns a success response from the service.

- kind:test
  id: ingest-publishes-only-validated-readings
  service: gatedemo-ingest
  verify: A valid sensor reading submitted over HTTP is published to Kafka topic `gatedemo.validated-readings`, and an invalid reading is rejected rather than published.

- kind:test
  id: processor-computes-rolling-average
  service: gatedemo-processor
  verify: After multiple validated readings for the same sensor are published, the processor emits an updated rolling average for that sensor on topic `gatedemo.sensor-averages`.

- kind:test
  id: gateway-streams-aggregates
  service: gatedemo-gateway
  verify: A browser-compatible WebSocket client connecting to `ws://gatedemo.example.com/ws` receives aggregate messages derived from `gatedemo.sensor-averages`.
```

## Non-Goals
- Defining service implementation code or container build pipelines.
- Introducing new ComponentDefinitions or custom traits.
- Adding persistence beyond the Kafka-backed realtime platform already provided by existing components.
