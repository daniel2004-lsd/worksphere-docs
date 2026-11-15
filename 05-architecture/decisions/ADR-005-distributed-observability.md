# ADR-005: Distributed Observability and Telemetry Strategy

- **Status:** Proposed (Pending Technical Approval by Daniel)
- **Date:** 2026-11-15
- **Decision Makers:** Daniel (`daniel2004-lsd`) — Software Architect & Technical Lead
- **Scope:** Universal across all WorkSphere microservices, databases, and message brokers

---

## 1. Context and Problem Statement

WorkSphere distributes business processes across multiple polyglot services and an asynchronous broker. When an approval takes 48 hours to complete, or when a task fails to unblock after an approval event, debugging the root cause across independent databases and logs without distributed correlation is error-prone and slow.

Requirements:
- End-to-end trace correlation from the web frontend through Kong Gateway, microservices, and RabbitMQ events.
- Centralized metric collection adhering to the Four Golden Signals.
- Structured log aggregation with mandatory `tenant_id` and `trace_id` injection.

---

## 2. Decision

We adopt an **OpenTelemetry-first telemetry architecture backed by the Grafana Observability Stack**:

1. **Instrumentation Standard:** OpenTelemetry (OTel) SDKs across all runtimes (Java OpenTelemetry Agent, .NET OpenTelemetry package, Go OTel).
2. **Context Propagation Standard:** W3C Trace Context (`traceparent`) forwarded in all HTTP headers and AMQP message properties.
3. **Collector & Backends:**
   - **OpenTelemetry Collector:** Single ingestion point for traces and metrics.
   - **Prometheus:** Metrics time-series engine.
   - **Grafana Tempo:** Distributed trace visualization.
   - **Grafana Loki:** Log aggregation via Docker daemon log driver / Promtail.
   - **Grafana:** Unified visualization dashboards.

---

## 3. Consequences

### Positive:
- Instant causality tracing: clicking an error in Loki directly surfaces the exact trace in Tempo and related service metrics in Prometheus.
- Vendor independence: OpenTelemetry allows switching backend storage without rewriting application instrumentation code.
- Operational bottleneck discovery: identifies exactly which service or approval step introduced latency.

### Operational Challenges & Mitigations:
- **Telemetry Overhead:** Trace sampling rate configured at 100% in local/QA environments, but reduced to 20% probabilistic sampling in production for high-volume read endpoints.
