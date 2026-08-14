# NBA Task Queue Deployment

> [!IMPORTANT]
> **Archived on August 13, 2026.** The maintained repositories are
> [awesome-nba-data](https://github.com/JovaniPink/awesome-nba-data) and the private
> [nba-lab](https://github.com/JovaniPink/nba-lab). NBA Lab v1 has no standalone queue, worker, or
> public API. This repository remains only as a superseded design record.

Historical design sketch for a Celery/RabbitMQ NBA task service.

## Project status

This design is **superseded and not deployable**. The repository contains no runnable worker,
broker configuration, dependency definition, tests, task contracts, or operations controls. Its
empty script placeholders should not be treated as an implementation.

NBA Lab keeps request paths small and runs approved acquisition, transformation, training, and
evaluation as bounded offline jobs. No durable asynchronous API is part of v1.

See [`docs/architecture-investigation.md`](docs/architecture-investigation.md) for the evidence,
replacement architecture, and reactivation criteria.

## Decision summary

- Do not deploy this repository.
- Do not make an HTTP request wait for a worker result.
- Do not use a task result backend as the canonical model-output store.
- Do not add RabbitMQ/Celery merely because a task is described as “data-heavy.”
- Preserve and archive the GitHub repository after linking this decision from the replacement
  repositories. This owner action was completed August 13, 2026.

## Replacement approach

1. Precompute model predictions and publish versioned outputs to PostgreSQL.
2. Run acquisition and training as scheduled, bounded commands outside the API request path.
3. Keep bounded batch work with NBA Lab so code, contracts, fixtures, and tests stay aligned.
4. If a user-triggered operation genuinely cannot finish within the latency budget, return
   `202 Accepted` with a durable job identifier and status resource.
5. Add a worker/queue only when rate, duration, concurrency, retry, and ownership evidence chooses
   it over a simpler mechanism.

The active repositories are [awesome-nba-data](https://github.com/JovaniPink/awesome-nba-data) and
the private [nba-lab](https://github.com/JovaniPink/nba-lab). Historical notebooks and data remain
preserved in the archived [nba-data](https://github.com/JovaniPink/nba-data) repository.

## If this design is ever reactivated

The implementation must define idempotency, at-least-once delivery behavior, acknowledgements,
timeouts, bounded retries with backoff, dead-letter handling, cancellation, result retention,
observability, deployment ownership, and recovery tests before it can be described as reliable.

## Contributing

This archived repository accepts no new runtime work. A future queue requires a new, explicit
architecture decision backed by measured workload evidence.

## License

[MIT](https://choosealicense.com/licenses/mit/)
