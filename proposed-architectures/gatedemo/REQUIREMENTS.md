# gatedemo Requirements

## Overview
`gatedemo` is a realtime IoT telemetry pipeline. It must ingest sensor readings over HTTP, validate them, publish valid readings to Kafka, compute a rolling average per sensor, and stream aggregates to browsers over websocket. The design must reuse the platform's existing realtime ComponentDefinitions and include the platform-mandated identity topology for all externally exposed services.

## Architecture Scope
The proposed architecture consists of:
- `gatedemo-identity` using `auth0-idp` as the single identity component.
- `gatedemo-platform` using `realtime-platform` for shared realtime infrastructure and topic declarations.
- `gatedemo-ingest` using `realtime-service` as the externally exposed HTTP ingest endpoint.
- `gatedemo-processor` using `realtime-service` as the internal stream processor.
- `gatedemo-gateway` using `realtime-service` as the externally exposed websocket gateway.

## Responsibilities by Service

### gatedemo-identity
Provides the single identity provider binding required by the platform for externally exposed services. Exposes connection material for authentication and JWT validation to the ingest and gateway components.

```acceptance
service: gatedemo-identity
criteria:
  - id: ac-gatedemo-identity-1
    statement: "Exactly one auth0 identity component is present for the application"
    kind: test
    given: "the gatedemo OAM Application is rendered"
    when: "the component list is inspected"
    then: "there is exactly one component of type auth0-idp named gatedemo-identity"
```

### gatedemo-platform
Provides the shared realtime substrate for the pipeline, including Kafka-backed topic declarations for validated readings and sensor aggregates, plus processor intent for rolling-average computation.

```acceptance
service: gatedemo-platform
criteria:
  - id: ac-gatedemo-platform-1
    statement: "The realtime platform declares the two Kafka topics needed by the pipeline"
    kind: test
    given: "the gatedemo OAM Application is rendered"
    when: "the gatedemo-platform properties are inspected"
    then: "topics gatedemo.validated.readings and gatedemo.sensor.aggregates are both declared"
```

### gatedemo-ingest
Receives sensor readings over HTTP, validates them, and publishes validated readings to the Kafka topic `gatedemo.validated.readings`. Because it is externally exposed, it must reference the single `gatedemo-identity` component in both component properties and exposure trait configuration.

```acceptance
service: gatedemo-ingest
criteria:
  - id: ac-gatedemo-ingest-1
    statement: "The ingest service is externally exposed over HTTP and publishes validated readings"
    kind: test
    given: "the gatedemo OAM Application is rendered"
    when: "the gatedemo-ingest component is inspected"
    then: "it has role ingest, exposes /ingest publicly, sets identity to gatedemo-identity, and produces gatedemo.validated.readings"
```

### gatedemo-processor
Consumes validated readings from Kafka and computes a rolling average per sensor, then publishes aggregate results to `gatedemo.sensor.aggregates`. It is internal-only and therefore does not require an identity binding.

```acceptance
service: gatedemo-processor
criteria:
  - id: ac-gatedemo-processor-1
    statement: "The processor consumes validated readings and emits per-sensor aggregates"
    kind: test
    given: "the gatedemo OAM Application is rendered"
    when: "the gatedemo-processor component is inspected"
    then: "it has role processor, consumes gatedemo.validated.readings, and produces gatedemo.sensor.aggregates"
```

### gatedemo-gateway
Consumes sensor aggregate events from Kafka and streams them to browser clients over websocket. Because it is externally exposed, it must reference the single `gatedemo-identity` component in both component properties and exposure trait configuration.

```acceptance
service: gatedemo-gateway
criteria:
  - id: ac-gatedemo-gateway-1
    statement: "The gateway exposes websocket streaming for aggregate updates"
    kind: test
    given: "the gatedemo OAM Application is rendered"
    when: "the gatedemo-gateway component is inspected"
    then: "it has role gateway, enables websocket behavior, exposes /ws publicly, sets identity to gatedemo-identity, and consumes gatedemo.sensor.aggregates"
```

## Acceptance Criteria
- The application reuses existing ComponentDefinitions only: `auth0-idp`, `realtime-platform`, and `realtime-service`.
- The application contains exactly one identity component of type `auth0-idp` named `gatedemo-identity`.
- Every externally exposed component sets `identity: gatedemo-identity` in component properties.
- Every externally exposed component sets `identity: gatedemo-identity` on its exposure trait.
- The ingest path is externally exposed over HTTP and publishes validated readings to `gatedemo.validated.readings`.
- The processor consumes `gatedemo.validated.readings` and produces rolling-average aggregates on `gatedemo.sensor.aggregates`.
- The websocket gateway consumes `gatedemo.sensor.aggregates` and exposes a browser-facing websocket stream.
- The OAM Application validates successfully with dry-run.

## Non-Goals
- Implementing application source code for ingest validation, rolling average logic, or websocket broadcasting.
- Deploying the application to the cluster.
- Defining browser UI behavior beyond websocket aggregate streaming.
