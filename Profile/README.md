# HealthKunj Clinics

**Technology-native homeopathic healthcare.**

We build the clinical infrastructure, patient experience, and digital ecosystem powering HealthKunj — a homeopathic medicine clinic in Kharadi, Pune, operated as a technology company from day one. Every consultation, prescription, payment, communication, and clinical outcome is owned, tracked, and automated through software we design and ship ourselves.

[![NestJS](https://img.shields.io/badge/NestJS-11-E0234E?logo=nestjs&logoColor=white)](https://nestjs.com)
[![Next.js](https://img.shields.io/badge/Next.js-16-000000?logo=next.js&logoColor=white)](https://nextjs.org)
[![Expo](https://img.shields.io/badge/Expo-SDK%2054-000020?logo=expo&logoColor=white)](https://expo.dev)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://postgresql.org)
[![GCP](https://img.shields.io/badge/GCP-Cloud%20Run-4285F4?logo=google-cloud&logoColor=white)](https://cloud.google.com)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?logo=typescript&logoColor=white)](https://typescriptlang.org)

---

## What We Build

HealthKunj is a homeopathic clinic treating chronic conditions — PCOS, thyroid, asthma, psoriasis, migraine, arthritis, and many more — under the clinical leadership of **Dr. Meera Thakur, BHMS**. Our platform serves every role: patients booking appointments, doctors conducting consultations and writing remedy protocols, admins managing operations, and the clinic itself growing through digital channels.

The codebase spans a family of products, all served by a single backend. No fragmented tools. No third-party EHR. One platform, built to our own clinical and operational requirements.

---

## Repositories

| Repository | Purpose | Stack | Hosted on |
|---|---|---|---|
| [`Healthkunj-Api`](https://github.com/HealthKunj-Clinics/Healthkunj-Api) | Unified backend — REST, auth, DB, AI and notifications for all products | NestJS 11 · TypeORM · PostgreSQL 16 | GCP Cloud Run |
| [`healthkunj-web`](https://github.com/HealthKunj-Clinics/healthkunj-web) | Marketing site + Admin / Doctor / Patient / HR / Super-Admin portal | Next.js 16 App Router · Tailwind v4 | GCP Cloud Run |
| [`HealthKunj-Mobile`](https://github.com/HealthKunj-Clinics/HealthKunj-Mobile) | Patient and doctor app for iOS and Android | Expo SDK 54 · React Native · NativeWind v4 | Not yet store-submitted |
| [`HealthKunj-Shop`](https://github.com/HealthKunj-Clinics/HealthKunj-Shop) | Online medicine and remedy shop — [shop.healthkunj.com](https://shop.healthkunj.com) | Vite · React · Razorpay | Firebase Hosting |
| [`HealthKunj-Case-forms`](https://github.com/HealthKunj-Clinics/HealthKunj-Case-forms) | Structured clinical intake forms per condition — [forms.healthkunj.com](https://forms.healthkunj.com) | Vite · React (static SPA, submits to the API) | Firebase Hosting |
| [`Healthkunj-Apps`](https://github.com/HealthKunj-Clinics/Healthkunj-Apps) | Kunji health calculators ([apps.healthkunj.com](https://apps.healthkunj.com)) + HoSTs suitability tests ([hosts.healthkunj.com](https://hosts.healthkunj.com)) | Vite · React · AI via the API | Firebase Hosting |
| [`healthkunj-digital`](https://github.com/HealthKunj-Clinics/healthkunj-digital) | Strategy docs, feature specs, constitution, internal dashboard, Playwright E2E suite | Markdown · React · Playwright | Internal |
| [`HealthKunj-Platforms`](https://github.com/HealthKunj-Clinics/HealthKunj-Platforms) | IT/platform branch — SaaS multi-tenancy + ABDM for other AYUSH clinics | NestJS | Scaffolded |
| [`HealthKunj-Academy`](https://github.com/HealthKunj-Clinics/HealthKunj-Academy) | Learning modules and academic initiatives — [academy.healthkunj.com](https://academy.healthkunj.com) | Next.js 16 | GCP Cloud Run |

---

## Architecture

**One API. One database. One auth layer.**

Every product — web portal, mobile app, shop, case forms, calculators — talks to `Healthkunj-Api`. There is no separate backend per product. The API is the single source of truth for every patient, appointment, order, prescription, and clinical record, and the only place AI models are called from.

```
┌──────────────────────────────────────────────────────────────────────┐
│                         healthkunj.com                               │
│                 Next.js 16 App Router · GCP Cloud Run                │
│     Marketing  ·  Admin Portal  ·  Doctor Portal  ·  Patient Portal  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │  /api/* proxy (never exposes backend)
            ┌──────────────────┼──────────────────┐
            │                                     │
     ┌──────▼──────┐                   ┌──────────▼──────────┐
     │ Mobile App  │                   │  Shop · Case Forms  │
     │ iOS/Android │                   │  Kunji · HoSTs      │
     │ Expo SDK 54 │                   │  Firebase Hosting   │
     └──────┬──────┘                   └──────────┬──────────┘
            │                                     │
            └──────────────────┬──────────────────┘
                               │  JWT + httpOnly refresh cookie
                               │  X-Client: mobile → JSON refresh body
┌──────────────────────────────▼───────────────────────────────────────┐
│                         Healthkunj-Api                               │
│                   NestJS 11 · GCP Cloud Run                          │
│   33 modules  ·  multi-tenant  ·  RBAC  ·  MFA  ·  audit logging     │
│   Razorpay  ·  ABDM  ·  notifications  ·  Gemini AI  ·  GST          │
└──────────────┬───────────────────────────────────────────────────────┘
               │
   ┌───────────┼─────────────────┐
   │           │                 │
┌──▼─────────┐ ┌─▼──────────┐ ┌─▼──────────────┐
│ Cloud SQL  │ │ Memorystore│ │ Cloud Storage  │
│ Postgres16 │ │ Redis 7    │ │ Docs · PDFs    │
│            │ │ private VPC│ │ Prescriptions  │
└────────────┘ └────────────┘ └────────────────┘
```

---

## Backend — `Healthkunj-Api`

**33 modules.** One deployed Cloud Run service handling every product and every role.

| Domain | Modules |
|---|---|
| Auth & Identity | `auth` · `users` · `tenants` |
| Clinic | `appointments` · `consultations` · `doctor` · `blocked-dates` · `clinical-evaluations` |
| Patient | `patient` · `health` · `referrals` · `case-forms` · `consent-logs` |
| Commerce | `shop` · `billing` · `payments` · `subscriptions` |
| Growth | `subscribers` · `contact` · `blog` · `content` · `crm` · `n8n` |
| Operations | `admin` · `analytics` · `notifications` · `uploads` · `mailer` |
| Platform | `chat` · `corporate-wellness` · `inventory` · `abdm` |
| Infrastructure | `redis` |

### Key Platform Capabilities

**Authentication**
- JWT (15-min access token, 30-day httpOnly refresh cookie)
- Google OAuth via Passport.js
- MFA / TOTP enforcement for clinical staff (admin, doctor)
- Account lockout after failed attempts, rate-limited auth endpoints
- Per-device session management + revocation
- Keyless CI/CD via Workload Identity Federation

**Access Control**
- Roles: `user · doctor · admin · super_admin` (HR access is scoped by company, not a separate role)
- Feature flags via environment (`FF_SUBSCRIPTIONS`, `FF_CLINICAL_EVALUATIONS`, `FF_CORPORATE_WELLNESS`, `FF_SAAS_PLATFORM`, `FF_ABDM`)
- Multi-tenant data model with per-request tenant context

**Clinical**
- Appointment lifecycle (NEW → CONFIRMED → COMPLETED / CANCELLED)
- Telemedicine — Google Meet / Zoom meeting link generation
- Clinical evaluations — burden scoring (0–100), questionnaires, shared progress links
- ABDM / ABHA — ABHA ID on patient records, consent logging
- Miasmatic classification in consultation notes
- Prescription-required flag on medicines + prescription approval flow
- Doctor availability, slot algorithm, multi-doctor scheduling

**Commerce & Billing**
- Razorpay orders, subscriptions, and webhook signature verification
- Sequential GST-compliant invoices (HK-INV-YYYY-NNNN)
- Shop with categories, coupons, wishlists, reviews, subscribe-and-save
- Corporate wellness packages with employee credit ledger
- Care subscription programs with scheduled consultations

**AI**
- Google Gemini 2.5 Flash, called server-side only — Kunji calculators (`/chat/message`) and HoSTs assessments (`/chat/assess`)

**Compliance**
- DPDP — data deletion endpoint + email confirmation
- Consent logs for all ABDM operations
- Audit log interceptor on all mutations and sensitive GETs
- Signed GCS URLs (15-min expiry) for all medical documents — never in API responses

---

## Clinical Focus

Homeopathic treatment for India's most prevalent chronic conditions, led by **Dr. Meera Thakur, BHMS**, with 108 condition pages on [healthkunj.com](https://healthkunj.com):

PCOS · Thyroid · Asthma · Diabetes · Migraine · Autism · ADHD · Immunity · Psoriasis · Arthritis · Allergy · Cholesterol · Acne & Skin · Hair Loss · Obesity · Liver · Mental Health · Women's Health · Kidney Stones · Homeopathy for children

Clinical outcomes tracked per patient through structured evaluations. Progress shareable with patients via secure token links. Case forms submitted digitally before consultation.

---

## Communication & Automation

**Patient communication is owned by the API.** Appointment confirmations, reminders, order updates and other transactional or clinical messages go through the API's `NotificationsService` — email, WhatsApp Business API and FCM push — with scheduled jobs driven by Cloud Scheduler. Postgres is the source of truth for every message sent.

**n8n is the outreach layer only** — marketing campaigns, social publishing and lead nurture, never patient-care messaging. Its API contract (`/outreach/*`) is live in `Healthkunj-Api`; the self-hosted n8n instance is not currently running.

---

## Infrastructure

| Layer | Service | Detail |
|---|---|---|
| Cloud | Google Cloud Platform | Project `healthkunj-140624` · `asia-southeast1` |
| API | Cloud Run | Auto-scaling · 0→N · `api.healthkunj.com` · separate QA service |
| Web | Cloud Run | Next.js standalone build · custom domain · separate QA service |
| Academy · Platforms | Cloud Run | Production only for now |
| Database | Cloud SQL | PostgreSQL 16 · Unix socket via Auth Proxy sidecar |
| Cache | Memorystore | Redis 7 · private VPC · no public IP |
| Files | Cloud Storage | Medical documents · invoices · prescription images · logos |
| CDN | Firebase Hosting | Shop · Case Forms · Kunji · HoSTs |
| Containers | Artifact Registry | `asia-southeast1-docker.pkg.dev/healthkunj-140624/healthkunj/` |
| CI/CD | GitHub Actions | API + Web: QA on push to `qa`, production by manual release. Firebase SPAs: deploy on merge to `main`. Workload Identity Federation — zero SA keys in secrets |
| Outreach automation | Hostinger VPS | n8n self-hosted (not currently running) |

---

## Technology Stack

```
Language        TypeScript (strict in the web, mobile and SPA repos; NestJS repos use a partial set)
Backend         NestJS 11 · TypeORM · class-validator · Passport.js · bcryptjs
Frontend        Next.js 16 App Router · Tailwind CSS v4 · Lucide icons
Mobile          Expo SDK 54 · React Native · NativeWind v4 · Expo SecureStore
Database        PostgreSQL 16 · Redis 7
Payments        Razorpay (orders · subscriptions · webhooks · refunds)
Notifications   FCM push · WhatsApp Business API · Nodemailer SMTP · Cloud Scheduler
AI              Google Gemini 2.5 Flash, server-side (health calculators + suitability tests)
Compliance      ABDM / ABHA · DPDP · GST invoicing
Testing         Playwright E2E · smoke after every production deploy · full authenticated suite weekly on QA
```

---

## Multi-Tenant SaaS

`Healthkunj-Api` is built multi-tenant from the ground up: tenant-scoped tables carry a `tenant_id`, every request resolves a tenant context, and SaaS features sit behind the `FF_SAAS_PLATFORM` flag. A tenant-isolation audit (November 2026) found a short list of gaps, chiefly binding each login to its tenant, to close before a second clinic is onboarded.

**The same platform that runs HealthKunj can run any AYUSH clinic in India** — with their own branding, subdomain, doctor roster, and feature set. This is the foundation for licensing HealthKunj as clinic management infrastructure to homeopathic, Ayurvedic, and Unani practitioners nationally. `HealthKunj-Platforms` is scaffolded as its future home.

---

## ABDM Compliance Roadmap

| Phase | Scope | Status |
|---|---|---|
| Phase 1 | ABHA ID on patient records · ABHA consent logging · HFR/HPR clinic registration | In progress — ABHA built; HFR/HPR registration pending |
| Phase 2 | UHI Health Service Provider integration · real ABDM credentials | Planned |
| Phase 3 | NHCX sandbox integration | Planned |
| Phase 4 | HealthKunj as EUA on UHI · SaaS licensing to AYUSH clinics | Year 2–3 |

Our NestJS + PostgreSQL stack **is** the EHR — we do not use third-party EHR software.

---

## Status

| Product | Status |
|---|---|
| API + Web portal | Live — [healthkunj.com](https://healthkunj.com) |
| Online shop | Live — [shop.healthkunj.com](https://shop.healthkunj.com) |
| Clinical intake forms | Live — [forms.healthkunj.com](https://forms.healthkunj.com) |
| Health calculators + suitability tests | Live — [apps.healthkunj.com](https://apps.healthkunj.com) · [hosts.healthkunj.com](https://hosts.healthkunj.com) |
| HealthKunj Academy | Live — [academy.healthkunj.com](https://academy.healthkunj.com) |
| Marketing CRM | Live in the admin portal |
| Mobile app (iOS + Android) | In development — core screens built, not yet store-submitted |
| Corporate wellness module | Feature-flagged, ready for first client |
| SaaS multi-tenant platform | Feature-flagged in `Healthkunj-Api`; `HealthKunj-Platforms` scaffolded as its future home, migration not started |
| Outreach automation (n8n) | API contract live; n8n instance not yet running |

---

## Contact

**Clinic** — Kharadi, Pune, Maharashtra, India  
**Website** — [healthkunj.com](https://healthkunj.com)  
**Email** — contact@healthkunj.com

---

<sub>HealthKunj Clinics Private Limited &nbsp;·&nbsp; "House of Health" &nbsp;·&nbsp; Practising since 2011 · incorporated 14 June 2024 &nbsp;·&nbsp; Kharadi, Pune</sub>
