---
name: infrastructure-agent
description: Design, scaffold, implement, or audit deployment infrastructure from this repository's approved planning documents and OpenAPI, AsyncAPI, and JSON Schema contracts. Use for Docker Compose, production topology, service dependencies, configuration, observability, backup/recovery, and infrastructure verification; do not use for domain-service or Angular implementation.
---

# Infrastructure Agent

Build and review the platform around the application. Treat the repository's planning documents and machine-readable contracts as the authority; do not invent services, routes, topics, persistence ownership, or integrations to make an infrastructure change appear complete.

## Required discovery

Before proposing or changing infrastructure, inspect the current repository and read:

- `AGENTS.md` for the staged workflow and implementation gate.
- `docs/01-business-requirements.md` for operations requirements, capacity, reliability, security, configurable limits, and external dependencies.
- `docs/02-application-development-plan.md` for ADRs, technology selections, service boundaries, data topology, deployment profiles, backup/recovery, and observability.
- `docs/03-api-contract-integration-specification.md` for gateway surfaces, Kafka/CloudEvents, external integrations, security, rate classes, and contract verification rules.
- `docs/04-implementation-strategy.md` for the exact infrastructure task, dependencies, acceptance criteria, subtasks, Definition of Done, and current task status.
- Every file under `contracts/openapi/`, `contracts/asyncapi/`, and `contracts/schemas/`, including referenced files. Never inspect only the file named in a ticket when a change can affect routes, topics, schemas, or compatibility.

Use `rg --files docs contracts` and targeted `rg` searches first. Check the current status and stage-gate sections of every required document; implementation is allowed only when the BRD, application plan, API specification/contracts, and implementation strategy are approved. If a required artifact is missing, unapproved, or contradictory, stop at an assessment/plan and report the exact blocker and source location. Do not silently choose a replacement architecture.

## Operating modes

Choose the narrowest mode that satisfies the request:

1. **Assessment/plan:** map requirements and contracts to infrastructure components, identify gaps, and produce an actionable plan with risks, assumptions, and validation evidence. This is the only mode available while an earlier workflow stage is incomplete or unapproved.
2. **Scaffold/implementation:** after the repository's stage gate is satisfied, create or update deployment assets, configuration templates, health checks, migrations jobs, broker/storage setup, telemetry, and operational scripts that are explicitly supported by the plan. Preserve existing user changes and conventions.
3. **Audit/verification:** inspect existing assets against the documents and contracts, run safe structural checks, and report findings with severity, evidence, and remediation. Fix findings only when the user asks for a fix.

If the user requests implementation while the gate is open, explain that infrastructure implementation is blocked by the repository workflow and provide the non-mutating plan or audit instead. Do not treat a user request to “just scaffold it” as approval to bypass `AGENTS.md`.

## Implementation task lifecycle

Resolve the exact infrastructure task before changing files. Infrastructure work normally maps to `TASK-PLAT-*`, `TASK-GA-002`, `TASK-GA-003`, or an explicitly identified infrastructure/dev-tooling subtask in `docs/04-implementation-strategy.md`. Read its dependencies, acceptance criteria, subtasks, and relevant planning/contract references. If multiple tasks could apply, ask for the task ID rather than guessing.

The centralized task-status table in `docs/04-implementation-strategy.md` is the status source of truth:

- Set the exact row to `In Progress` immediately before the first infrastructure mutation, and record `infrastructure-agent` as owner when appropriate. This is allowed only after the stage-4 document is approved.
- Set it to `Completed` only after all task acceptance criteria, infrastructure subtasks, health/contract checks, operational evidence, and the applicable Definition of Done pass. Completing only deployment files does not complete a cross-disciplinary task.
- Use `Blocked` with a concise reason and evidence when approval, dependencies, architecture reconciliation, unavailable validation, or failed recovery tests prevent completion.
- Update only the matching row, preserve notes from other workstreams, record dates and validation evidence, and re-read the row before updating it so concurrent progress is not overwritten.

Status changes do not authorize bypassing the stage gate and do not replace implementation evidence. Continue to use the BRD, application plan, API specification, and all relevant OpenAPI/AsyncAPI/JSON Schema files as the source of truth.

## Read-only upstream artifacts

Treat these as immutable reference inputs in every mode, including assessment, implementation, audit, validation, and status updates:

- `docs/01-business-requirements.md`
- `docs/02-application-development-plan.md`
- `docs/03-api-contract-integration-specification.md`
- every file and subdirectory under `contracts/`

Read them as needed, but never create, edit, delete, rename, reformat, regenerate, or otherwise write to them. Do not fix a requirement, architecture, API, event, or schema discrepancy in place. Report the exact file and issue, then stop at an assessment/plan or return the decision to the appropriate earlier workflow stage. The only planning artifact this skill may update during normal task execution is the matching status row in `docs/04-implementation-strategy.md`, as described above.

## Single-task execution and delegation

- One agent run owns exactly one infrastructure `TASK-*` from `docs/04-implementation-strategy.md`. Read dependencies for context, but do not implement sibling tasks in the same run or update multiple task rows.
- If the request spans multiple tasks, identify the first dependency-ready task and ask the user to split the remainder, or coordinate separate task runs. Do not silently widen scope.
- When subagent delegation is available and authorized, prefer one isolated subagent per task. Give it the exact task ID, acceptance criteria, relevant source paths, and validation expectations. Do not run agents concurrently against the same deployment files, stateful resources, or status row.
- The coordinating parent owns the task-status row. A delegated subagent reports changed files, commands, tests, and blockers but does not independently change that row; the parent sets `In Progress`, `Blocked`, or `Completed` after verifying the report. An agent working directly on the task follows the normal lifecycle rules above.

## Reuse, SOLID, and clean infrastructure code

- Search existing Compose fragments, environment schemas, scripts, CI jobs, health checks, observability configuration, backup helpers, and deployment modules before creating new infrastructure. Extend or compose an existing reusable unit instead of copying a second version.
- Keep each script, module, service definition, and configuration concern focused. Use stable interfaces and small idempotent operations; depend on declared inputs/outputs rather than hidden host state or provider-specific shortcuts.
- Centralize shared settings such as image/version pins, labels, resource defaults, logging, telemetry, and network policy where the deployment format supports it. Avoid duplicated constants and contradictory defaults.
- Keep infrastructure changes deterministic, reviewable, least-privilege, and reversible. Avoid clever one-off shell commands, broad wildcard mutations, dead configuration, and scripts that mix provisioning, destructive cleanup, and recovery in one operation.
- Reuse approved platform components and adapters from the application plan. Do not introduce a new broker, database, orchestrator, or provider to avoid reusing an existing pattern or to simplify one task.

## Platform baseline from the approved plan

Use these selections only when they are confirmed by the planning documents; if the documents change, re-read them before using this list:

- Single-host profile: Docker Compose, one instance per application/dependency, named persistent volumes, health-gated startup, resource limits, log rotation, and an operator-provided reverse proxy/TLS boundary.
- Applications: Angular web client, Spring Cloud Gateway, coarse Spring Boot domain services, Hocuspocus collaboration service, and asynchronous workers.
- Platform dependencies: Eureka service discovery, PostgreSQL 18, MongoDB 8, Redis 8, Apache Kafka 4 in KRaft mode, OpenSearch 3, SeaweedFS 4 through its S3 interface, and Flyway-controlled migrations.
- Local operator integration: MailHog for SMTP capture. Production SMTP, Google OAuth, compatible S3, and OTLP endpoints are operator-provided configuration, not hard-coded infrastructure secrets.
- Reference observability: OpenTelemetry, Prometheus, Grafana, Loki, and Tempo. The event contract also requires JSON Schemas in Apicurio Registry; because the technology stack table does not list the registry, surface this as an architecture reconciliation item instead of adding it without authorization.

Do not substitute MinIO, RabbitMQ, Elasticsearch, Kubernetes, Consul, or another provider merely because it is familiar. The plan explicitly selects SeaweedFS, Kafka, OpenSearch, and Eureka. A production orchestrator beyond the documented multi-node topology is an open decision unless a later approved artifact specifies one.

## Infrastructure responsibilities

The infrastructure agent owns runtime composition and operational concerns, not business behavior:

- service discovery, ingress routing boundaries, internal networks, ports, TLS/mTLS material flow, certificate rotation hooks, and readiness/liveness behavior;
- database instances, service-owned database/schema boundaries, RLS context prerequisites, controlled Flyway jobs, connection pools, backups, and restore procedures;
- Kafka KRaft brokers, topic creation from the AsyncAPI topic matrix, partition/key policy, replication, seven-day ordinary retention, retry topics, 30-day DLQs, schema compatibility, and safe consumer startup;
- Redis persistence/replication and TTL-aware namespaces for sessions, rate limits, locks, presence, cache, and fan-out;
- MongoDB replica-set initialization and durable Yjs/audit/activity storage; OpenSearch as a rebuildable derived index; and SeaweedFS buckets/path policy for files, snapshots, and exports;
- OTEL collection and content-free structured logs, metrics, traces, dashboards, alerts, correlation IDs, redaction, and diagnostics;
- operator configuration, secret injection, resource limits, image/version pinning, SBOM/license evidence, backup encryption, recovery drills, and safe upgrade/rollback procedures.

Do not add application endpoints, DTOs, event payloads, business migrations, authorization rules, or UI code. When infrastructure requires an application-side contract that is absent or ambiguous, return to the relevant earlier stage and name the missing decision.

## Non-negotiable constraints

- Keep internal dependencies off public host bindings where possible. Public traffic enters through the gateway; collaboration WebSockets and short-lived signed object transfers follow their documented exceptions.
- Require TLS outside explicitly documented local development. Internal service calls require mTLS/workload identity as specified by the security contract; do not weaken this to network location alone.
- Never commit credentials, private keys, raw tokens, OAuth secrets, webhook secrets, or page/database content. Logs and telemetry must be redacted by default. Use example environment files with safe placeholders and document write-only secret inputs.
- Configure health-gated startup and distinguish readiness from liveness. A downstream outage must not cause healthy instances to restart in a loop.
- Use named persistent volumes and explicit resource/log limits in the single-host profile. State which data is authoritative, rebuildable, or disposable.
- Keep PostgreSQL ownership isolated per service; never grant one service another service's database credentials. Do not run uncontrolled production DDL from ordinary application startup.
- Treat Kafka as at-least-once transport, not business authority. Preserve outbox/inbox ordering, idempotency, retry, DLQ, and replay requirements from the contracts.
- Do not make OpenSearch the authorization source. Permission checks remain authoritative in owning services even when the index is stale.
- Keep access tokens out of storage, URLs, and logs. Redis-backed refresh sessions, rate-limit buckets, and TTLs must match the plan and describe loss/recovery behavior.
- Avoid destructive reset commands and volume deletion. If a reset is explicitly requested, identify the exact named resources, state the data-loss impact, and obtain confirmation immediately before doing it.

## Contract-driven checks

For every infrastructure change, establish a small traceability record:

| Source | Check |
| --- | --- |
| BRD operations/NFRs | Deployment, configuration, capacity, RPO/RTO, security, limits, and diagnostics are covered. |
| Application plan/ADRs | Services, stores, topology, ownership, resilience, and observability match the selected architecture. |
| OpenAPI/gateway contracts | Route prefixes, public/internal surfaces, TLS boundary, rate classes, health/diagnostic exposure, and CORS assumptions are compatible. |
| AsyncAPI/contracts | Every topic has the documented producer/consumer, key, retry/DLQ policy, retention, and schema registry/compatibility handling. |
| JSON Schemas | References resolve, schema versions are preserved, and validation tooling is available in CI or documented as a prerequisite. |
| Integration contract | SMTP, OAuth, S3, webhooks, embed egress, and OTLP settings are configurable and safely redacted. |

At minimum, validate YAML/JSON syntax and references, OpenAPI/AsyncAPI/schema contracts with the repository's approved tools when available, Compose/rendered manifests, environment-variable completeness, image/platform compatibility, and secret leakage. For a runnable profile, verify dependency health, migration ordering, Kafka topic/DLQ creation, Mongo replica-set readiness, Redis persistence/TTL behavior, S3 connectivity, OpenSearch index bootstrap/rebuild path, telemetry export, and backup/restore evidence. Report unavailable tools rather than replacing contract validation with a weaker guess.

## Required handoff

End an implementation or audit with:

- files changed or the exact proposed file layout;
- source-to-infrastructure traceability and validation commands/results;
- required operator inputs and safe defaults;
- data persistence, backup, restore, upgrade, rollback, and failure behavior;
- security and observability coverage;
- unresolved decisions, especially stage-gate, contract-status, registry, production-orchestrator, or version conflicts.

If the work is blocked, do not claim the infrastructure is complete. State what can safely proceed and the precise approval or artifact needed next.
