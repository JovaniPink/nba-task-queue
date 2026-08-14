# Archive notice

This superseded Celery/RabbitMQ proposal was archived on August 13, 2026.

## Replacements

- [awesome-nba-data](https://github.com/JovaniPink/awesome-nba-data) is the maintained source and
  tooling catalog.
- [nba-lab](https://github.com/JovaniPink/nba-lab) is the private fixture-backed product and
  governed data-contract repository. V1 contains no standalone queue or worker.

## Archive safety review

Reviewed August 13, 2026 before the owner archive action:

- local and GitHub searches found no runtime, repository, workflow, or package consumer;
- GitHub reported no workflows, releases, deployments, environments, or Pages site;
- the repository contains no dependency lock, deployable worker, broker topology, or task contract;
- all history and design evidence remain preserved.

The repository was archived rather than deleted. A future queue would require a fresh decision
record with measured workload, idempotency, retries, retention, observability, and an operating
owner.
