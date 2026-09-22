# PatchPilot

Evidence-first GitHub Actions failure analysis. PatchPilot receives failed `workflow_run` webhooks, identifies the first meaningful failure, redacts secrets, and posts a concise diagnosis to related pull requests.

## Run locally

Requires Node.js 20+.

```bash
npm install
npm run build
npm test
npm start
```

Copy `.env.example` to `.env`. Set `GITHUB_APP_ID`, `GITHUB_PRIVATE_KEY`, and `GITHUB_WEBHOOK_SECRET`; use `DRY_RUN=true` to analyze without posting comments.

## MVP behavior

- Verifies `X-Hub-Signature-256` webhook signatures.
- Handles completed failed `workflow_run` events.
- Deduplicates `repository + run_id + run_attempt`.
- Retrieves failed job logs and ignores cleanup noise.
- Redacts common credentials before analysis.
- Posts evidence-based PR comments and never modifies source automatically.

The current store is in-memory and processing is synchronous. Replace it with PostgreSQL plus a queue before production scale, then add Marketplace entitlement metering and retention controls.
