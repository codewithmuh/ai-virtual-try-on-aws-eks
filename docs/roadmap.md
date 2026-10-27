# Roadmap

Built in public, phase by phase. Every phase ends with measured results, not claims. See [scope.md](scope.md).

## Phase 0: the app, locally (one GPU, or API-only mode)

- [x] Repository from the production template: CI, scans, signed releases, FastAPI plumbing (probes, metrics,
      JSON logs, graceful shutdown)
- [ ] ADRs: try-on model (quality, GPU memory, speed, license), SQS for jobs, API-only fallback, safety checks
- [ ] API: presigned uploads, try-on jobs, status events, results; photos deleted after 24 hours (rule R1)
- [ ] Safety checks on uploads and results (rule R2)
- [ ] GPU worker: takes a job, runs the model, saves the result; runs on a local GPU or in API-only mode
- [ ] Stylist: Claude picks two catalog items that complete the look
- [ ] Web: upload, catalog, live job stages, result, share card

## Phase 1: GPUs on EKS

- [ ] Terraform: VPC, EKS, Karpenter, SQS with a dead-letter queue, S3, ECR, Pod Identity
- [ ] GPU NodePool: Spot first, on-demand fallback, tainted, with a cap on GPUs
- [ ] NVIDIA device plugin (or the GPU Operator; ADR)
- [ ] KEDA scales workers on queue depth from zero; Karpenter removes the empty GPU node
- [ ] Model weights mirrored to S3; Argo CD for everything

## Phase 2: measured

- [ ] Cold start: queue → GPU node → model loaded → first image, before and after optimising
- [ ] Throughput per GPU and cost per 1,000 try-ons, Spot and on-demand
- [ ] Drill: Spot interruption mid-job; the job is retried and none are lost

## Phase 3: v1.0

- [ ] All 11 areas of the production-grade checklist
- [ ] Results table in the README, demo video and write-up
