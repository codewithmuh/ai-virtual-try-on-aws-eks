# Kisekae (着せ替え): AI Virtual Try-On on AWS EKS

[![ci](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks/actions/workflows/ci.yml/badge.svg)](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks/actions/workflows/ci.yml)
[![release](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks/actions/workflows/release.yml/badge.svg)](https://github.com/codewithmuh/ai-virtual-try-on-aws-eks/actions/workflows/release.yml)
[![License: learning only, no commercial use](https://img.shields.io/badge/License-learning%20only%2C%20no%20commercial%20use-orange.svg)](LICENSE)

**See yourself in it before you buy it.** Kisekae is an AI virtual try-on for fashion stores: a photo of you
plus a garment gives a photo of you wearing it, and an AI stylist suggests what goes with it. The try-on model
runs on your own GPUs in Amazon EKS, and you pay for them only while someone is trying something on.

> **Your photos stay private.** Uploads are deleted after 24 hours and never used for training. The try-on only
> swaps clothes over clothes; nudity and photos of children are rejected.

**Status: phase 0, building in public.** The repository, CI and the API's plumbing are done; choosing the try-on model is next.
See the [roadmap](docs/roadmap.md).

## Part 3 of AI on EKS

Five production AI apps on Amazon EKS, one industry each. Every part adds one new layer of Kubernetes on top of
the parts before it.

| Part | Project | Industry | New on Kubernetes |
|---|---|---|---|
| 1 | [Heya](https://github.com/codewithmuh/ai-virtual-staging-aws-eks): AI virtual staging | Real estate | EKS basics: ALB Ingress with HTTPS, HPA, Pod Identity, then Argo CD |
| 2 | [Karute](https://github.com/codewithmuh/ai-medical-scribe-aws-eks): AI medical scribe | Healthcare | Security: NetworkPolicies, VPC endpoints, External Secrets, WebSockets |
| **3** | **Kisekae: AI virtual try-on (this repo)** | **Fashion** | **GPUs: Karpenter, KEDA on SQS, Spot, scale to zero** |
| 4 | [Fukikae](https://github.com/codewithmuh/ai-video-dubbing-aws-eks): AI dubbing in your own voice | Media | GPU pipelines: Argo Workflows, GPU sharing, RBAC |
| 5 | [Mimamori](https://github.com/codewithmuh/ai-cctv-video-search-aws-eks): AI CCTV video search | Security & retail | Operators (CRD), batched model serving, GPU metrics, SLOs |

## The demo

1. Upload a photo of yourself.
2. Pick a garment from the demo catalog, or upload a product photo.
3. Seconds later you are wearing it. The first request starts a GPU node from zero, and the page shows it happen.
4. The stylist picks two more items to complete the look and renders them.
5. Share the result as a card.

Then the platform demo: the queue empties, the workers scale to zero and Karpenter removes the GPU node.

## What it proves

| Question an interviewer asks | How Kisekae answers it |
|---|---|
| "How do you run GPUs on EKS without paying for idle?" | Karpenter adds a GPU node when jobs arrive and removes it when the queue is empty |
| "How do you scale workers on a queue?" | KEDA scales the try-on workers on SQS queue depth, from zero |
| "What about Spot interruptions?" | Spot GPUs with on-demand fallback; jobs stay in SQS until the result is saved, proven in a drill |
| "How long is a cold start?" | Measured: queue → GPU node → model loaded → first image, then reduced (image size, cached weights) |
| "What does one try-on cost?" | GPU seconds per image and cost per 1,000 try-ons, Spot vs on-demand, published |

The full design, rules and the numbers it will publish are in [docs/scope.md](docs/scope.md).

## The stack

| Layer | Choice |
|---|---|
| AI | Open-source try-on model on your own GPU (chosen in an ADR); Claude (vision) as the stylist; an API-only mode for accounts without GPU quota |
| App | Next.js web, FastAPI API, Python GPU worker, SQS (with a dead-letter queue), S3 |
| Kubernetes | Karpenter (CPU and GPU NodePools), NVIDIA device plugin, KEDA, taints and tolerations, Argo CD, External Secrets |
| AWS | GPU instances (g5 or g6, Spot first), SQS, S3, ECR, Pod Identity, all in Terraform |

## Before you start

- GPU quota. New AWS accounts often start at 0 vCPUs for GPU instances. Request the "Running On-Demand G and VT
  instances" quota and the matching Spot quota (8 vCPUs covers two g5.xlarge or g6.xlarge nodes). Ask early:
  approval can take days.
- An Anthropic API key, and a Hugging Face account for the model weights.
- No GPU quota yet? API-only mode runs the whole app with an image-editing API instead of the GPU model.
- Every phase publishes its measured cost per hour and ends with a teardown that leaves nothing running.

## Quick start

Requirements: [uv](https://docs.astral.sh/uv/), [pre-commit](https://pre-commit.com/) and Docker, or just open
the **Dev Container**.

```bash
make setup    # install dependencies and git hooks
make check    # lint, type-check and test (same as CI)
make up       # the API on :8000

curl localhost:8000/healthz          # {"status":"ok"}
curl localhost:8000/readyz           # {"status":"ready"}

make down
```

Run `make` with no arguments to list every command.

## Repository layout

```
services/api/      FastAPI service: src/api (config, logs, probes, metrics), tests/, Dockerfile. The
                   try-on endpoints go here as they are built.
docs/              scope, roadmap, architecture, threat model, ADRs, runbooks
compose.yaml       local run with production-like hardening
Makefile           every command you need
```

The web app, Terraform and Kubernetes manifests arrive with their phases in the [roadmap](docs/roadmap.md).

## How changes flow

1. Open a pull request with a [Conventional Commit](https://www.conventionalcommits.org/) title.
2. CI must pass: lint, types, tests, secret and vulnerability scans, image build and smoke test.
3. release-please keeps a Release PR open. Merging it tags the version and publishes a signed image to GHCR with an
   SBOM.

## Production-grade checklist

Every item must pass before v1.0.

- [ ] Code quality: linters, formatters, type checks and pre-commit pass
- [ ] Tests: unit + integration, ≥ 80% coverage, a smoke test and a load test with published numbers
- [ ] CI: every PR runs lint, tests, build and scans; `main` is protected
- [ ] Security: no secrets in git, least privilege, scanned and signed images, SBOM, threat model
- [ ] Infrastructure as code: remote state, `make up` / `make down`, cost estimate, clean teardown
- [ ] Releases: semantic versions, changelog, images in a registry
- [ ] Observability: logs, metrics, traces, dashboards as code, alerts linked to runbooks
- [ ] Reliability: probes, graceful shutdown, timeouts/retries/idempotency, tested backups, written SLOs
- [ ] Docs: README with quick start, architecture, ADRs, runbooks, contributing, license
- [ ] Developer experience: one-command setup, Dev Container or Compose, seed data
- [ ] Demo: screenshots, a short demo video, a write-up

## License

**Learning only. Commercial use is not allowed.** You may clone, run, modify and deploy this project to learn, use
it to teach at a school or in free workshops, and show it in your portfolio with credit. You may not sell it, run it
as a service, use it in company or client work, or use it in paid or monetised courses and videos without written
permission. The full terms are in [LICENSE](LICENSE).

For a commercial license, or if you are unsure whether your use is allowed: **contact@codewithmuh.com**

© 2026 Muhammad Rashid ([CodeWithMuh](https://youtube.com/@codewithmuh)), Senior Platform Engineer ·
[codewithmuh.com](https://codewithmuh.com) · [GitHub](https://github.com/codewithmuh) ·
[LinkedIn](https://www.linkedin.com/in/muhammad-rashid-daha/)
