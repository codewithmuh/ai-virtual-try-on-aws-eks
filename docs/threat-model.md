# Threat model

Lightweight [STRIDE](https://learn.microsoft.com/en-us/azure/security/develop/threat-modeling-tool-threats) review.
Revisit it when you add an external integration, a data store, or a new trust boundary.

## Assets

- Shoppers' photos of themselves
- Generated images
- GPU capacity and the AI bill
- Service availability
- Build and release pipeline (images, signing identity)

## Threats and mitigations

| STRIDE | Threat | Mitigation | Status |
|---|---|---|---|
| Spoofing | Forged release images | Keyless cosign signatures tied to this repo's workflow | ✅ |
| Tampering | Compromised GitHub Action or dependency | Actions pinned to SHAs, Dependabot, Trivy scans, SBOM | ✅ |
| Repudiation | Unknown who changed what | Protected `main`, required reviews, signed release provenance | ✅ |
| Information disclosure | Secrets committed to git | gitleaks in pre-commit and CI; `.env` ignored | ✅ |
| Elevation of privilege | Container breakout | Non-root, read-only, no capabilities, minimal base image | ✅ |
| Denial of service | Request floods exhaust the service | Rate limits at the edge (AWS WAF on the ALB) and per user in the API | ⏳ planned |
| Denial of service | Abuse runs up the AI or GPU bill ("denial of wallet") | Per-user daily limits, a global daily budget, alerts at 80% | ⏳ planned |
| Tampering | Prompt injection through uploaded content | Model output is schema-validated and never triggers an action on its own | ⏳ planned |
| Tampering | The app is used to make sexual or non-consensual images | Rule R2: clothes over clothes only; safety checks on every upload and every result | ⏳ planned |
| Information disclosure | Shopper photos leak or are kept too long | Private bucket, presigned URLs that expire in minutes, deletion after 24 hours | ⏳ planned |
| Denial of service | A flood of jobs scales GPUs without limit | A cap on GPU nodes in the NodePool, per-user limits, a queue that absorbs bursts | ⏳ planned |
| Tampering | A malicious model file runs code on the worker | Weights in safetensors format from pinned revisions, mirrored to a private S3 bucket | ⏳ planned |

## Open risks

List accepted risks with an owner and a review date.
