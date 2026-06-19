# gatedemo Requirements

## Summary
`gatedemo` is a realtime IoT telemetry pipeline. It ingests sensor readings over HTTP, validates them, publishes accepted readings to Kafka, computes a rolling average per sensor, and streams aggregate updates to browser clients over WebSocket.

## Architecture
The proposal reuses the platform's existing realtime ComponentDefinitions:

- `realtime-platform` provides the Kafka-backed realtime substrate and declares the pipeline topics and processing context.
- `realtime-service` is reused three times:
  - `gatedemo-ingest` accepts HTTP sensor submissions and publishes validated readings to Kafka.
  - `gatedemo-processor` consumes validated readings and publishes rolling averages.
  - `gatedemo-gateway` consumes aggregates and serves browser-facing websocket streams.
- `pure-ingress` exposes the gateway externally at `/ws`.

## Service Responsibilities

### gatedemo-platform
Owns the shared realtime infrastructure for the application, including Kafka topic declarations and pipeline-level rolling-average processing intent.

```acceptance
service: gatedemo-platform
criteria:
  - id: ac-1
    statement: "The shared realtime substrate declares both raw and aggregate telemetry topics"
    kind: test
    given: "The gatedemo OAM application is rendered"
    when:  "The realtime platform claim is inspected"
    then:  "It declares Kafka topics named sensor-readings and sensor-aggregates"
```

### gatedemo-ingest
Accepts HTTP sensor readings, validates payloads, and publishes accepted events to the raw telemetry Kafka topic.

```acceptance
service: gatedemo-ingest
criteria:
  - id: ac-1
    statement: "Valid HTTP sensor readings are published to the raw telemetry topic"
    kind: test
    given: "The ingest service is running and a syntactically valid sensor reading is submitted over HTTP"
    when:  "The request is accepted by the service"
    then:  "A corresponding event is published to the sensor-readings Kafka topic"
```

### gatedemo-processor
Consumes validated telemetry events and emits per-sensor rolling averages.

```acceptance
service: gatedemo-processor
criteria:
  - id: ac-1
    statement: "The processor emits rolling averages per sensor"
    kind: test
    given: "The processor service is running and multiple validated readings for the same sensor exist on sensor-readings"
    when:  "The processor handles those events"
    then:  "It publishes aggregate events for that sensor to the sensor-aggregates Kafka topic"
```

### gatedemo-gateway
Consumes aggregate events and streams them to browser clients over WebSocket.

```acceptance
service: gatedemo-gateway
criteria:
  - id: ac-1
    statement: "Browser clients receive aggregate updates over WebSocket"
    kind: test
    given: "The gateway service is running and a browser client is connected to /ws"
    when:  "A new aggregate event appears on the sensor-aggregates Kafka topic"
    then:  "The connected browser client receives the aggregate update over the websocket connection"
```

## External Interface
- HTTP ingest endpoint: provided by `gatedemo-ingest`
- Kafka raw topic: `sensor-readings`
- Kafka aggregate topic: `sensor-aggregates`
- Browser websocket endpoint: `wss://gatedemo.example.com/ws`

## Non-Goals
- Defining the exact sensor payload schema beyond the requirement that validation is enforced
- Selecting production DNS, TLS, or authentication settings for the websocket hostname
- Defining persistence beyond Kafka topic retention
