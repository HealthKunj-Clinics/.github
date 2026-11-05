# Contributing — HealthKunj Engineering Policy

This is the canonical, org-wide source for how code moves from a laptop to production across
**all** HealthKunj repositories. It lives here, in `HealthKunj-Clinics/.github`, so it applies
by default to every repo in the org that doesn't define its own override — one copy, not nine
drifting ones.

Two rules underpin everything else here:

1. **Nothing goes to `main` without a PR, review, and passing scans.**
2. **Nothing goes to production without being tested and approved on a test environment.**

**A load-bearing fact:** `HealthKunj-Clinics` is on the GitHub Free plan. Free does not support
branch protection, required status checks, required reviewers, or required-reviewer
environment gates on **private** repositories — and every HealthKunj repo is and should stay
private (health data). So none of the above two rules are enforced by GitHub itself. They're
enforced by the mechanisms below instead. If that ever changes (upgrading to GitHub Team,
~$4/seat/month, unlocks all of this natively), swap the mechanism, keep the rule.

---

## 1. How "nothing goes to `main` without PR, review, scanning" actually works

| Control | Mechanism (Free-tier compatible) |
|---|---|
| No direct push to `main`/`master` | A `pre-push` git hook, committed to every repo at `.githooks/pre-push` and enabled via `git config core.hooksPath .githooks`, refuses a push whose target ref is `main`/`master` unless `ALLOW_DIRECT_PUSH=1` is set. It's local and bypassable — real friction, not a hard wall — which is the honest ceiling on Free. |
| Review before merge | A `CODEOWNERS` file in every repo. GitHub auto-requests review from the owner on a PR even without branch protection; still a nudge, not a block, until the org has more than one engineer or upgrades plan. |
| Required checks pass | Every repo runs a `verify` job (typecheck, lint, build, and unit tests where a repo has them) on PRs. Where a deploy fires automatically, the deploy job declares `needs: verify` in the workflow file, so **a failing check structurally cannot reach the deploy step** — a real, unbypassable gate, achieved through the GitHub Actions dependency graph rather than a GitHub policy feature. Api and Web production deploys are instead manual releases of a commit already verified on its PR and checked on QA (§2). |
| Secret scanning | The `gitleaks` **CLI** (Apache-2.0, free) run directly as a CI step — not the `gitleaks/gitleaks-action` marketplace wrapper, which now requires a paid license for organization accounts (individual accounts are still free). Fails the build on a detected secret. This exists because `healthkunj-321414` was already suspended once for a leaked credential — the cheapest control on this list, non-negotiable. |
| SAST | `semgrep` as a CI step (free tier, no GHAS needed) — used instead of CodeQL, which requires GitHub Advanced Security. |
| Container scanning | `trivy` as a CI step against the built Docker image, before it's pushed to Artifact Registry. Applies to the Cloud Run repos only; the Firebase SPAs and Mobile ship no container. |
| Dependency scanning | Dependabot alerts — already enabled org-wide, genuinely free regardless of plan. New dependencies introducing a critical/high vulnerability should block merge by convention; existing backlog needs a separate remediation pass with an explicit SLA, not just a gate on new code (see §5). |

**Review model.** Engineering today is one human (Naval) plus Claude. Classic two-engineer
review doesn't map onto that. The practical equivalent: Claude self-reviews every change with
`/code-review` before opening a PR — the objective, mechanical pass — and Naval is the required
human approver, whose approval doubles as the production-readiness signal in §2. As the team
grows, move to a real second reviewer via `CODEOWNERS` rather than resting on the founder for
every PR forever.

## 2. How "nothing goes to production without tested + approved on test" actually works

Three tiers: **local → QA → production.** A change reaches production only after passing QA —
and passing QA is a separate event from being deployed to production.

- **Cloud Run services (Api, Web): push to `qa` → auto-deploy to QA.** Automatic, no approval
  needed — QA is meant to be cheap to update. Merging to `main` deploys nothing.
- **Firebase SPAs: a PR gets a preview channel, merging to `main` deploys to production.** The
  preview channel is the test environment; review on the preview URL before merging. (Shop has
  no preview channel yet — see the open gaps below.)
- **Deploying to production is `workflow_dispatch`-only.** A separate job/workflow that a human
  must manually trigger, picking which QA-validated commit to promote. On Free, GitHub
  Environments' "required reviewers" gate isn't available for private repos — a manual-trigger
  step is the free substitute, and it's arguably clearer anyway: the act of triggering it *is*
  the approval, with no ambiguity about who clicked what.
- **The full, mutating E2E suite runs against QA, not production.** Production only gets the
  read-only smoke subset, post-deploy, as a verification signal — not a testing ground. Running
  the full suite against prod pollutes real data (this happened; see the 2026-07-17/18
  incident where MFA/session state churned on shared prod test accounts during E2E debugging).
- **Migrations are tested against a QA database copy before they ship.** `synchronize: true` is
  dev-only, never production.

### Repo status

| Repo | QA environment | Prod deploy trigger |
|---|---|---|
| `Healthkunj-Api` | Cloud Run QA service (`deploy-qa.yml`, on push to `qa`) | `workflow_dispatch` (since 2026-10-19) |
| `healthkunj-web` | Cloud Run QA service (`deploy-qa.yml`, on push to `qa`) | `workflow_dispatch` (since 2026-10-19) |
| `HealthKunj-Shop` | none — Firebase Hosting only | push to `main` |
| `HealthKunj-Case-forms` | Firebase preview channel per PR (`preview.yml`) | push to `main` (`needs: verify`) |
| `Healthkunj-Apps` | Firebase preview channel per PR (`preview.yml`) | push to `main` (`needs: verify`) |
| `HealthKunj-Mobile` | none — `verify.yml` only, no build/release workflow | n/a (not store-submitted) |
| `healthkunj-digital` | n/a — docs, dashboard and the E2E suite, not a deployed product surface | — |
| `HealthKunj-Platforms` | none — **production-only by decision** (QA later); prod not yet launched | `workflow_dispatch` |
| `HealthKunj-Academy` | none — **production-only by decision** (QA later) | `workflow_dispatch` |

**Decision (2026-10-18): `HealthKunj-Platforms` and `HealthKunj-Academy` are production-only.**
Neither runs a QA environment; one will be created later, once there is something worth
validating outside production. Each still has a manual-dispatch production gate, plus a
build-and-scan workflow on every push to `main`. This is a deliberate exception to the
two-environment rule above, not a gap to fix. Don't restore their QA pipelines unprompted.

**Production gate for Api and Web (2026-10-19).** Merging to `main` no longer deploys either
service. QA deploys on every push to `qa`, and a production release is a deliberate
`gh workflow run deploy.yml --ref main -f ref=<sha>` for a commit already checked on QA.
Dependabot auto-merges sync to `qa` and ship to prod with the next release.

**One open gap, called out honestly rather than implied fixed:**
- Shop has no test environment, and no `verify` job either: every push to `main` goes straight
  to production. The fix is the same one Apps and Case-forms already use — a `verify.yml`, a
  Firebase Hosting **preview channel** per PR, and `needs: verify` on the deploy job.

**Testing.** The Playwright E2E suite lives in `healthkunj-digital`. The read-only smoke subset
runs after every Api/Web production deploy (dispatched from `deploy.yml`) and on changes to the
tests; the full, mutating suite runs weekly against QA and on manual dispatch.

## 3. Rollback and traceability

- Every production deploy is tagged (git tag matching the deployed image tag) so a running
  revision traces back to an exact commit.
- Cloud Run rollback is a documented one-command action (traffic split to the previous
  revision), not something improvised during an incident. Firebase Hosting rollback
  (`firebase hosting:rollback`) documented the same way for the SPA repos.
- Post-deploy smoke verification should be a hard, alerting gate — a failing post-deploy smoke
  run needs to page/notify, not just sit green-or-red in an Actions tab nobody is watching.

## 4. Access and secrets

- CI/CD auth uses Workload Identity Federation — no static service-account JSON keys in GitHub
  Secrets, in any repo, ever.
- CI secrets are scoped per-repo, not shared broadly across the org.
- Production secrets live in GCP Secret Manager, not `.env` files or GitHub Secrets, except
  where GitHub Secrets is the only available mechanism (e.g. E2E test credentials).
- Given DPDP-covered health data moves through this stack, whoever approves a production
  deploy (§2) shouldn't be the only identity that can also bypass every other control —
  separation of duties matters more here than in a typical project.

## 5. Vulnerability backlog

A merge gate stops *new* vulnerable dependencies; it doesn't retroactively fix what's already
there. Check Dependabot directly per-repo for current counts (they drift). Track remediation of
the existing backlog as a recurring task with an explicit SLA for criticals — a gate sitting on
top of an unaddressed pile is security theater, not security.

## 6. Feature flags

Land risky or large changes behind a flag (`FF_SUBSCRIPTIONS`, `FF_CLINICAL_EVALUATIONS`, etc.
— pattern already established in `Healthkunj-Api`), verify behind the flag in QA, then flip it
on in production as a separate, reversible step from the deploy itself. This decouples "shipped
the code" from "turned on the behaviour," which is most of what makes rollback fast when
something's wrong.

---

## Status

This policy was written 2026-07-18 as a target state. Where each repo actually stands, as of
2026-11-05:

| Repo | `pre-push` hook | `CODEOWNERS` | `verify` (gitleaks + semgrep + typecheck) | Deploy `needs: verify` | trivy | Prod deploys tagged |
|---|---|---|---|---|---|---|
| `Healthkunj-Api` | ✅ | ✅ | ✅ (+ unit tests, build; no lint) | n/a — manual release | ❌ | ❌ |
| `healthkunj-web` | ✅ | ✅ | ✅ (+ build; no lint) | n/a — manual release | ❌ | ❌ |
| `HealthKunj-Shop` | ❌ | ❌ | ❌ | ❌ | n/a | ❌ |
| `HealthKunj-Case-forms` | ✅ | ✅ | ✅ (+ lint, build) | ✅ | n/a | ❌ |
| `Healthkunj-Apps` | ✅ | ✅ | ✅ (+ lint, build) | ✅ | n/a | ❌ |
| `HealthKunj-Mobile` | ❌ | ❌ | ✅ (+ expo-doctor, advisory) | n/a — no release | n/a | n/a |
| `healthkunj-digital` | ❌ | ❌ | ❌ | n/a | n/a | n/a |
| `HealthKunj-Platforms` | ✅ | ✅ | ✅ (+ lint, unit tests, build) | ✅ | ✅ | ✅ |
| `HealthKunj-Academy` | ✅ | ✅ | ✅ (+ lint, build) | ✅ | ✅ | ✅ |

Done since the policy was written:

- [x] `verify` CI with gitleaks + semgrep added to Api, Web, Case-forms, Apps and Mobile
- [x] Manual-dispatch production gate for `Healthkunj-Api` and `healthkunj-web` (2026-10-19)
- [x] Preview-channel test environment for Case-forms and Apps
- [x] E2E full/mutating run pointed at QA; production gets post-deploy smoke only

Still open:

- [ ] `HealthKunj-Shop`: `verify`, preview channel, `needs: verify`, hook, `CODEOWNERS`
- [ ] `.githooks/pre-push` and `CODEOWNERS` in `HealthKunj-Mobile` and `healthkunj-digital`
- [ ] `trivy` image scan in the Api and Web deploy workflows
- [ ] A working lint step in Api and Web (Api has no eslint config; Web's `next lint` was
      removed in Next 16 and typescript-eslint doesn't support TS 7 yet)
- [ ] Git-tag production deploys in Api, Web, Shop, Case-forms and Apps (§3)
- [ ] Post-deploy smoke failures alert someone, rather than only showing red in Actions (§3)

Keep this table current as boxes get checked; the file should describe reality, not
aspiration.
