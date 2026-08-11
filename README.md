# NBA Task Queue Deployment

Historical design sketch for a Celery/RabbitMQ NBA task service.

## Project status

This design is **superseded and not deployable**. The repository contains no runnable worker,
broker configuration, dependency definition, tests, task contracts, or operations controls. Its
empty script placeholders should not be treated as an implementation.

The current NBA API intentionally uses a request-driven runtime without Celery or RabbitMQ. Keep
interactive reads synchronous and serve precomputed predictions. Run ingestion, transformation,
training, and evaluation as bounded offline jobs. Introduce a durable asynchronous API only after a
measured workload and an explicit reliability contract justify it.

See [`docs/architecture-investigation.md`](docs/architecture-investigation.md) for the evidence,
replacement architecture, and reactivation criteria.

## Decision summary

- Do not deploy this repository.
- Do not make an HTTP request wait for a worker result.
- Do not use a task result backend as the canonical model-output store.
- Do not add RabbitMQ/Celery merely because a task is described as “data-heavy.”
- Recommend archiving the GitHub repository after this decision is merged and linked from active
  project documentation. Archiving is a separate owner action, not part of this pull request.

## Replacement approach

1. Precompute model predictions and publish versioned outputs to PostgreSQL.
2. Run acquisition and training as scheduled, bounded commands outside the API request path.
3. Keep the first job implementation with the active API repository so code, migrations, tests, and
   releases stay aligned.
4. If a user-triggered operation genuinely cannot finish within the latency budget, return
   `202 Accepted` with a durable job identifier and status resource.
5. Add a worker/queue only when rate, duration, concurrency, retry, and ownership evidence chooses
   it over a simpler mechanism.

The active service is
[`awesome-nba-data-api`](https://github.com/JovaniPink/awesome-nba-data-api). Historical notebooks
and data artifacts live in [`nba-data`](https://github.com/JovaniPink/nba-data), which is also not a
runtime dependency.

## If this design is ever reactivated

The implementation must define idempotency, at-least-once delivery behavior, acknowledgements,
timeouts, bounded retries with backoff, dead-letter handling, cancellation, result retention,
observability, deployment ownership, and recovery tests before it can be described as reliable.

## Contributing

This repository accepts corrections to the historical decision record. New runtime code belongs in
the active service until the reactivation criteria establish a justified independent service.

## License

[MIT](https://choosealicense.com/licenses/mit/)
