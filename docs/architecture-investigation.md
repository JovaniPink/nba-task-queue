# Architecture investigation

_Reviewed: 2026-08-11_

> **Archive decision completed August 13, 2026.** The replacements are
> [awesome-nba-data](https://github.com/JovaniPink/awesome-nba-data) and the private
> [nba-lab](https://github.com/JovaniPink/nba-lab). This document is a historical design record.

## Decision

Do not implement or deploy the proposed standalone Celery/RabbitMQ service. Mark this repository as
a superseded design record and recommend GitHub archival after the documentation decision is merged.

This is the one repository in the NBA project family that is genuinely overengineered for the
current goals: it commits to a distributed topology before defining the job, workload, correctness
semantics, or operating owner.

## Evidence from the repository

- The README described Celery, RabbitMQ, Flask subscription, and PostgreSQL results as a desired
  architecture rather than observed system behavior.
- `scripts/tasks/data_extract.py` and `scripts/tasks/etl.py` are empty.
- There is no dependency lock, Celery app, broker topology, container definition, database schema,
  task input/output contract, API integration, test, or deployment workflow.
- Installation, worker, scraping, ETL, example, and test sections were placeholders.
- No throughput, latency, concurrency, payload-size, failure-rate, or cost evidence selects a
  distributed broker.

The gap is not “unfinished code.” The proposed topology has no falsifiable requirement against
which it can be judged.

## Problems in the original interaction model

### The request waits for asynchronous work

The proposed web application starts a task and waits for the worker. That retains most of the
failure and latency coupling of synchronous execution while adding broker, worker, serialization,
result-backend, and monitoring dependencies.

If processing is accepted but not complete, HTTP defines `202 Accepted` for that state. A response
should identify a durable job/status resource; the client can poll or receive a separately designed
notification. The original HTTP connection should not be the job-lifetime boundary.

### Distribution does not create reliability

RabbitMQ acknowledgement and recovery flows can redeliver messages. Celery therefore recommends
idempotent tasks when late acknowledgement is used. Every side effect needs a stable idempotency key
and transactional record. “Celery handles it” is not a failure contract.

### A result backend is not a domain store

A task result says whether an invocation completed and may retain a serialized return value. A
prediction product needs canonical records with model version, feature cutoff, input snapshot,
calibration version, output schema, supersession/correction state, and access policy. Those records
belong in application tables even if Celery is eventually used for execution.

### Model training is not an interactive request

Acquisition, feature generation, training, and evaluation should be bounded offline jobs. The web
path should read a precomputed, versioned prediction or perform a deliberately small calculation.
Deep learning is not an architectural requirement; model choice follows validated data size,
baseline performance, calibration, latency, and operational cost.

## Simpler target

```text
scheduled/bounded batch command
  -> acquire approved snapshot
  -> validate and transform
  -> train/evaluate offline when required
  -> publish versioned predictions to PostgreSQL

web request
  -> read versioned prediction from API
  -> return immediately with freshness/model metadata
```

Keep bounded batch commands with NBA Lab. This gives them the same contracts, fixtures, CI, and
review without coupling execution to HTTP handlers.

## Escalation ladder

1. **Synchronous read or bounded calculation:** default for the UI.
2. **Scheduled process:** default for acquisition, transformation, and training.
3. **PostgreSQL job table:** consider for modest durable user jobs when one database and a small
   worker meet the measured workload. PostgreSQL's `SKIP LOCKED` can support queue-like consumers,
   but it provides an intentionally inconsistent view and still requires leases/retries/recovery.
4. **Managed queue or Celery broker:** consider when independent scaling, routing, scheduling,
   throughput, or isolation requirements exceed the database-backed design.
5. **Separate repository/service:** only after ownership and deployment cadence are independently
   justified.

This ladder is not a mandate to build every stage. Stop at the simplest one that meets the evidence.

## Reactivation criteria

Do not write worker code here unless a decision record supplies all of the following:

- named job and user outcome;
- typed, versioned input and output contracts;
- p50/p95/p99 duration, request rate, concurrency, and payload size;
- latency budget showing why precomputation or synchronous execution is insufficient;
- idempotency key and transaction boundary;
- at-least-once/at-most-once tradeoff for each side effect;
- acknowledgement, retry/backoff, timeout, cancellation, and dead-letter rules;
- task and canonical-result retention periods;
- authentication/authorization and tenant isolation if user-triggered;
- metrics, logs, traces, alerts, runbooks, and on-call owner;
- broker/result-store backup and recovery expectations;
- integration tests that exercise crash, duplicate delivery, retry, and stale-worker behavior;
- cost and operational comparison with a scheduled job, PostgreSQL queue, and managed queue.

If the evidence selects Celery and RabbitMQ, create the implementation from a current Python
baseline and current official documentation rather than reviving the empty placeholders.

## Cross-repository consequence

NBA Lab intentionally omits Celery and RabbitMQ. Reintroducing the same speculative dependency
through a separate repository would recreate coupling while splitting tests and ownership.

## Archive procedure

Completed August 13, 2026:

1. link the decision from the replacement documentation;
2. confirm no deployment, package, automation, or repository depends on this project;
3. preserve the repository history;
4. archive the GitHub repository through an explicit owner action.

Do not delete the repository or rewrite its history as part of the archive decision.

## Primary references

- [HTTP `202 Accepted` semantics (RFC 9110)](https://www.rfc-editor.org/rfc/rfc9110.html#name-202-accepted)
- [Celery task idempotency and acknowledgements](https://docs.celeryq.dev/en/stable/userguide/tasks.html)
- [Celery broker and result-backend overview](https://docs.celeryq.dev/en/stable/getting-started/backends-and-brokers/index.html)
- [RabbitMQ reliability guide](https://www.rabbitmq.com/docs/reliability)
- [RabbitMQ consumer acknowledgements and publisher confirms](https://www.rabbitmq.com/docs/confirms)
- [PostgreSQL `SELECT`, including `SKIP LOCKED`](https://www.postgresql.org/docs/current/sql-select.html)
