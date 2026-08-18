---
name: logging-wide-events
description: Designs and reviews structured operational logging around canonical wide events. Use when adding or reviewing logs, instrumenting requests, jobs, consumers, or workflows, defining log schemas or correlation, or fixing noisy, sparse, unsafe, duplicated, or expensive logging.
---

# Logging Wide Events

Build operational logs around one **wide event** per **unit of work**: an authoritative structured record, enriched during execution and emitted when that unit completes. Add other events only when they represent independently meaningful occurrences. Logs complement traces and metrics; ordinary operational logs are not a durable audit system.

## Vocabulary

**Unit of work** — one operation whose outcome matters: an HTTP request, queue message, scheduled job, CLI invocation, batch operation, workflow step, or streaming session.

**Wide event** — the unit's canonical completion record. It carries enough safe, bounded context to query the unit by outcome, timing, deployment, correlation identifiers, and relevant domain dimensions.

**Owner** — the boundary responsible for the unit's lifecycle. The owner initializes, enriches, and emits its wide event. Inner layers return or annotate errors instead of logging the same failure again.

**Independent event** — a separately queryable occurrence with meaning beyond narrating the unit's implementation steps: a security decision, externally visible state transition, retry attempt operators act on, progress checkpoint, circuit-breaker transition, or process lifecycle event.

## Process

### 1. Establish the local contract

Before changing logs, inspect the repository's:

- logger construction, serializers, redaction, transports, and shutdown behavior;
- field naming, event schemas, severity policy, and existing tests;
- tracing or OpenTelemetry setup and context-propagation APIs;
- privacy, retention, tenant-access, and data-classification rules;
- logging backend limits, indexing behavior, and sampling configuration.

Use the existing logger and context mechanism. Repository policy overrides this generic discipline. If the safety or retention policy does not settle whether a field is allowed, omit it and ask the user rather than guessing.

#### Fallback: establish a minimum viable logging boundary

Assess only the affected units and their nearest shared logging boundary. That boundary is usable only if the affected paths can emit structured fields, carry per-unit context, serialize errors safely, and apply redaction. If it is absent, string-only, inconsistently configured for those paths, or unsafe, correct it directly when doing so affects only its current callers. If correcting it would change unrelated callers or output and transport contracts, leave those callers alone: establish the smallest project-owned boundary for the affected units, or surface the decision when a second boundary would conflict with project architecture.

Keep the foundation proportional to the work being done:

1. Prefer the framework or runtime's structured logger, then a mature logger already installed in the project. Add a dependency only when the local stack cannot provide structured serialization, redaction, and contextual fields without rebuilding a logging library.
2. Expose one project-owned logger boundary that accepts a severity and structured event object. Callers pass objects; the boundary owns final serialization, redaction, and transport.
3. When no log service or vendor transport exists, application services emit newline-delimited JSON to the runtime's conventional logging stream and use the platform collector when one exists. CLI operational logs go to stderr and never contaminate contractual stdout. Reusable libraries accept the host application's logger or logging callback rather than choosing a process transport.
4. Supply safe defaults at the boundary: UTC timestamp, severity, event name, and service or operation identity derived from existing application metadata. Omit unknown deployment fields rather than inventing values. Centralize bounded value handling and safe error serialization there.
5. At each affected unit owner, add the smallest framework middleware, handler wrapper, or job harness that records monotonic duration, binds context, and emits completion. Continue an inbound trace or correlation ID when present; otherwise generate a unit ID at ingress and propagate it to outbound work.
6. Verify the fallback through its captured stream: each line parses as one JSON object, representative expected and unexpected terminal outcomes produce the canonical event, correlation survives the affected boundary, and representative secrets and oversized values are redacted or bounded.

Establish only the shared foundation the affected units need. Do not turn a logging change into an unrelated codebase-wide migration.

### 2. Name the unit and its owner

Identify the unit's entry point and every terminal outcome in its contract: success, expected rejection or skip, handled failure, timeout, cancellation, retry scheduling, and abandonment where applicable. Put the wide event at the narrowest boundary that observes the authoritative terminal signal.

Each service hop owns its own unit. When one unit calls a dependency, record a bounded dependency summary or trace span; the receiving service records its own wide event. Do not make every internal function another logging boundary.

Choose the unit and completion signal explicitly for the workload:

| Workload | Usual unit and authoritative completion | Critical caveat |
| --- | --- | --- |
| HTTP | One request-response exchange; response finished or aborted | Handler return is not completion for a streamed response; capture disconnect or abort. |
| Queue consumer | One delivery attempt; broker- or framework-confirmed ack, nack, requeue, or dead-letter settlement where observable | When confirmation is unavailable, emit after authoritative disposition handoff; classify settlement failure separately and retain identifiers for redelivery. |
| Scheduled or background job | One execution attempt; worker or scheduler terminal state | Distinguish completed, retry scheduled, cancelled, and permanently abandoned. |
| CLI | One invocation; predetermined final exit status | Determine status, emit completion to stderr, await logging flush, then exit with the predetermined status; preserve stdout as the command's data contract. |
| Replay-based workflow | The runtime-defined workflow run or activity; its replay-safe terminal hook | Use runtime-provided replay-safe logging, clock, and IDs; direct emission from deterministic replay code can duplicate events. |
| Stream | One session or one record, chosen explicitly; close, abort, or disconnect | Keep session context bounded and represent useful progress as bounded independent checkpoints. |

### 3. Design the wide event

Follow the repository's schema when one exists. Otherwise choose stable fields from the groups that answer real operational questions:

- **Identity** — event name, schema version, service, operation, unit kind.
- **Correlation** — trace and span IDs; request, message, job, or workflow IDs where useful and permitted.
- **Outcome** — a bounded value from the workload contract, such as `success`, `rejected`, `skipped`, `deduplicated`, `retry_scheduled`, `timeout`, `cancelled`, `abandoned`, or `failure`; duration; protocol status; retry count.
- **Deployment** — environment, version or commit, region, runtime, instance class.
- **Domain** — allowlisted identifiers, state, feature decisions, plan or product dimensions needed to explain the outcome.
- **Dependencies** — bounded counts, outcomes, and timings for important downstream work.
- **Error** — sanitized error type, stable code, retryability, and an approved safe message or stack representation.

There is no target field count. Include fields with diagnostic or operational value; keep every value safe and bounded.

Initialize the event at entry, enrich it when context becomes known, and emit it once from the owner's completion boundary. Pass a structured object to the logger rather than pre-stringifying JSON. Preserve application control flow when event construction or transport fails, using the project's centralized logging failure policy rather than scattering `try`/`catch` around calls.

```text
event = {
  event: "message_processing.completed",
  schema_version: 1,
  operation: "send_invoice",
  message_id: permitted_message_id,
  trace_id: active_trace_id
}
started = monotonic_clock()

try:
  result = process_message()
  event.outcome = outcome_for_result(result, workload_contract)
  event.invoice_type = result.invoice_type
catch error:
  terminal = classify_error(error, workload_contract)
  event.outcome = terminal.outcome
  event += terminal.safe_fields
  rethrow
finally:
  event.duration_ms = elapsed_since(started)
  existing_logger.write(level_for(event), event)
```

Adapt the shape and completion mechanism to the language and framework. A `finally` block covers ordinary control flow, not hard termination such as out-of-memory failure or `SIGKILL`.

### 4. Decide whether another event earns a line

For every additional log call, ask:

> Does this represent an independently queryable occurrence, or merely narrate execution of the wide event?

Keep independent events such as:

- security and compliance-relevant decisions sent through the approved sink;
- externally meaningful state transitions;
- retry attempts or dead-lettering when operators need attempt-level evidence;
- bounded progress checkpoints for long-running work;
- circuit-breaker, startup, shutdown, and crash events;
- temporary sampled diagnostics for a specific investigation.

Fold narrative milestones, duplicated errors, and incidental values into the wide event. Tag temporary diagnostic events so cleanup is a single search, and remove them when the investigation ends unless they proved durable operational value.

### 5. Preserve correlation

Continue the repository's existing trace context. When available, propagate W3C Trace Context across network and queue boundaries and attach active trace and span IDs to events. Preserve causal relationships through fan-out and retries using the tracing library's supported links or parentage.

Use request or correlation IDs as a fallback when distributed tracing is unavailable, or as stable identifiers humans and support systems exchange. Do not invent a second propagation mechanism beside one the system already uses.

### 6. Keep context safe

Build an explicit allowlist of fields. Prefer bounded scalars, stable enums, approved opaque identifiers, counts, and timings. Normalize or truncate user-controlled strings before emission.

Apply centralized redaction before transport. Credentials, authorization headers, cookies, session material, access and refresh tokens, private keys, connection strings, payment credentials, and complete request or response bodies must not enter ordinary logs. Serialize errors through a safe formatter rather than passing arbitrary thrown objects to the logger.

Treat user IDs, tenant IDs, email addresses, IP addresses, device identifiers, free text, financial values, and health or identity data according to project policy. Confirm whether they are permitted, need pseudonymization, and are visible only to appropriate operators for an appropriate retention period. If any answer is unknown, leave the field out and surface the decision.

### 7. Keep the schema queryable and affordable

- Use static field names and one naming convention within the system.
- Record explicit units such as `duration_ms` instead of relying on convention.
- Version schemas when consumers rely on their shape.
- Bound strings, arrays, stack traces, and nested objects; replace large collections with counts and a small approved sample.
- Consider ingestion cost, event rate, payload limits, backpressure, and shutdown flushing.
- Match sampling to the operational contract. Document which errors, security events, or rare outcomes must bypass ordinary sampling.
- Put high-cardinality dimensions in event records only when the backend can query them economically. Do not copy uncontrolled high-cardinality values into metric labels.

### 8. Use severity as one dimension

Follow the repository's levels. If none are defined, use this restrained default:

| Level | Meaning |
| --- | --- |
| `debug` | Temporary or detailed diagnostic evidence |
| `info` | Expected lifecycle and terminal outcomes, including expected rejection, skip, or deduplication |
| `warn` | Degraded or unusual but handled behavior |
| `error` | Unexpected system failure or an outcome requiring operator attention |
| `fatal` | The process cannot continue |

Do not derive severity mechanically from a non-success outcome. Keep semantics in fields such as `event`, `outcome`, `error_code`, and `retryable`; severity alone should not encode the event's meaning.

## Review transformations

When reviewing existing logging, prefer these transformations:

| Instead of | Move toward |
| --- | --- |
| Scattered narrative strings | One enriched wide event at the owner |
| Logging the same stack at every layer | One authoritative failure plus propagated stable error context |
| Serializing whole requests, users, or errors | Explicitly allowlisted safe fields |
| Manually stringifying JSON | Passing a structured object to the configured logger |
| Dynamic field names | Static field name plus a bounded value |
| Large arrays or payloads | Counts, summaries, and small approved samples |
| A new ad hoc request ID | Existing trace context, with a correlation ID only where it adds value |
| Using logs as an audit ledger | A dedicated durable, access-controlled audit mechanism |

## Verification

Exercise the real logging boundary or its established test sink. Verify every applicable invariant:

- [ ] Each affected workload names its unit, owner, and authoritative terminal signal.
- [ ] Each execution emits exactly one wide event when it reaches any applicable terminal outcome.
- [ ] Expected rejection, skip, deduplication, or retry outcomes are not mislabeled as operational errors.
- [ ] Timeout, cancellation, abandonment, disconnect, ack/nack, and broker-settlement failure paths emit the correct outcome where applicable and observable.
- [ ] A CLI determines its exit status, emits completion, awaits logging flush, then exits with that same status.
- [ ] Required fields have stable names, types, and explicit units.
- [ ] Trace or correlation context survives the relevant downstream boundary.
- [ ] Sensitive fields are absent or redacted, including inside serialized errors.
- [ ] User-controlled and collection values remain bounded.
- [ ] Inner layers do not duplicate the owner's failure event.
- [ ] Logger or transport failure does not change application behavior, unless the project explicitly defines fail-closed audit semantics.
- [ ] Sampling and severity follow the repository's operational contract.
- [ ] If no usable logger existed, affected applications use one project-owned boundary whose captured output is valid newline-delimited JSON with safe defaults; CLIs preserve stdout and libraries use the host logger.

Assert the important fields and invariants rather than snapshotting an entire rendered log line. Report any path that hard termination or unavailable infrastructure makes impossible to verify.
