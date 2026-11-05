## What & why

<!-- What does this change do, and why is it needed? Link the spec/issue if one exists. -->

## Test plan

<!-- How was this verified? Commands run, environments checked (local/QA), edge cases considered. -->

## Rollback plan

<!-- If this breaks something in production, how does it get undone? Cloud Run revision
     rollback, Firebase hosting:rollback, revert migration, flip a feature flag off — name it. -->

## Checklist

- [ ] `verify` CI is green (whatever the repo runs: typecheck, lint, tests, build, gitleaks, semgrep; trivy on Cloud Run image builds)
- [ ] Tested on QA (Api, Web) or the PR preview channel (Firebase SPAs), not just locally, if this touches a deployable
- [ ] No secrets, credentials, or PII committed
- [ ] Migrations (if any) tested against a QA database copy
