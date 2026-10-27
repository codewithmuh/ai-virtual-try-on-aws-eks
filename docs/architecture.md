# Architecture

> Keep this page current: update it in the same pull request that changes the architecture.
> The target design is in [scope.md](scope.md). This page describes what is **built** today.

## Context

Kisekae renders a shopper wearing a garment from two photos, and suggests what to wear with it. It is part 3
of AI on EKS and the first part with GPUs: the try-on model runs on GPU nodes that exist only while there
is work in the queue.

## Built so far

```mermaid
flowchart LR
    client([Client]) --> api[api<br/>FastAPI :8000]
    api -. /metrics .-> prom[(Prometheus)]
```

| Component | Responsibility | Tech | Status |
|---|---|---|---|
| `api` | HTTP API, runtime contract | Python 3.13, FastAPI, uvicorn | Plumbing done; choosing the try-on model next |

Planned components are in the [roadmap](roadmap.md).

## Runtime contract (every service)

| Concern | How |
|---|---|
| Liveness | `GET /healthz`: process is up; never checks dependencies |
| Readiness | `GET /readyz`: 503 until started and during shutdown |
| Metrics | `GET /metrics`: Prometheus `http_requests_total`, `http_request_duration_seconds` |
| Logs | One JSON line per event on stdout, with `request_id` |
| Shutdown | On SIGTERM: fail readiness, drain in-flight requests (≤ 20 s), exit 0 |
| Config | Environment variables only; secrets from the platform's secret manager |
| Security | Non-root user, read-only filesystem, no capabilities, minimal base image |

## Data

Planned: photos and results in S3 (shopper photos deleted after 24 hours), try-on jobs in SQS with a
dead-letter queue, job status in the API's store (chosen in an ADR). See [scope.md](scope.md).

## Deployment

Planned: Terraform for AWS; Helm charts synced by Argo CD from `main`. GPU workers ship as their own image.
Roll back by reverting the commit.

## Reliability

SLOs are written in phase 2 of the [roadmap](roadmap.md), each with an alert and the runbook it links to.

## Security

Trust boundaries and controls: see [threat-model.md](threat-model.md).

## Decisions

See [adr/](adr/).
