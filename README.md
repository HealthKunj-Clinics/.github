# HealthKunj Clinics

**Technology-native homeopathic healthcare.**

We build the clinical infrastructure, patient experience, and digital ecosystem powering HealthKunj — a homeopathic medicine clinic in Kharadi, Pune, operated as a technology company from day one. Every consultation, prescription, payment, communication, and clinical outcome is owned, tracked, and automated through software we design and ship ourselves.

[![NestJS](https://img.shields.io/badge/NestJS-10-E0234E?logo=nestjs&logoColor=white)](https://nestjs.com)
[![Next.js](https://img.shields.io/badge/Next.js-14-000000?logo=next.js&logoColor=white)](https://nextjs.org)
[![Expo](https://img.shields.io/badge/Expo-SDK%2054-000020?logo=expo&logoColor=white)](https://expo.dev)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-4169E1?logo=postgresql&logoColor=white)](https://postgresql.org)
[![GCP](https://img.shields.io/badge/GCP-Cloud%20Run-4285F4?logo=google-cloud&logoColor=white)](https://cloud.google.com)
[![TypeScript](https://img.shields.io/badge/TypeScript-Strict-3178C6?logo=typescript&logoColor=white)](https://typescriptlang.org)

---

## What We Build

HealthKunj is a homeopathic clinic treating chronic conditions — PCOS, thyroid, asthma, psoriasis, autism, diabetes, migraine, and 16 more — under the clinical leadership of **Dr. Meera Thakur, BHMS**. Our platform serves every role: patients booking appointments, doctors conducting consultations and writing remedy protocols, admins managing operations, and the clinic itself growing through digital channels.

The codebase spans seven interconnected products, all served by a single backend. No fragmented tools. No third-party EHR. One platform, built to our own clinical and operational requirements.

---

## Repositories

| Repository | Purpose | Stack | Hosted on |
|---|---|---|---|
| [`Healthkunj-Api`](https://github.com/HealthKunj-Clinics/Healthkunj-Api) | Unified backend — REST, auth, DB for all products | NestJS · TypeORM · PostgreSQL 16 | GCP Cloud Run |
| [`healthkunj-web`](https://github.com/HealthKunj-Clinics/healthkunj-web) | Marketing site + Admin / Doctor / Patient / HR / Super-Admin portal | Next.js 14 App Router · TypeScript | GCP Cloud Run |
| [`HealthKunj-Mobile`](https://github.com/HealthKunj-Clinics/HealthKunj-Mobile) | Patient and doctor app for iOS and Android | Expo SDK 54 · React Native · NativeWind v4 | App Store / Play Store |
| [`HealthKunj-Shop`](https://github.com/HealthKunj-Clinics/HealthKunj-Shop) | Online medicine and remedy shop | Vite · React · Razorpay | Firebase Hosting |
| [`HealthKunj-Case-forms`](https://github.com/HealthKunj-Clinics/HealthKunj-Case-forms) | Structured clinical intake forms per condition | Vite · React · Express | Firebase Hosting |
| [`HealthKunj-Apps`](https://github.com/HealthKunj-Clinics/HealthKunj-Apps) | Kunjis (health calculators) + HoSTs (suitability tests) | Vite · React · Gemini AI | Firebase Hosting |
| [`healthkunj-digital`](https://github.com/HealthKunj-Clinics/healthkunj-digital) | Digital command dashboards + 88 n8n automation workflows | React · n8n | Active |

---

## Architecture

**One API. One database. One auth layer.**

Every product — web portal, mobile app, shop, case forms, calculators — authenticates against `Healthkunj-Api`. There is no separate backend per product. The API is the single source of truth for every patient, appointment, order, prescription, and clinical record.

```
┌──────────────────────────────────────────────────────────────────────┐
│                         healthkunj.com                               │
│                 Next.js 14 App Router · GCP Cloud Run                │
│     Marketing  ·  Admin Portal  ·  Doctor Portal  ·  Patient Portal  │
└──────────────────────────────┬───────────────────────────────────────┘
                               │  /api/* proxy (never exposes backend)
     ┌─────────────────────────┼──────────────────────────┐
     │                         │                          │
┌────▼────────┐     ┌──────────▼──────────┐    ┌─────────▼──────────┐
│ Mobile App  │     │  Shop · Case Forms  │    │  n8n Automation    │
│ iOS/Android │     │  Kunjis · HoSTs     │    │  88 workflows      │
│ Expo SDK 54 │     │  Firebase Hosting   │    │  sochish.cloud     │
└────┬────────┘     └──────────┬──────────┘    └─────────┬──────────┘
     │                         │                          │
     └─────────────────────────┼──────────────────────────┘
                               │  JWT + httpOnly refresh cookie
                               │  X-Client: mobile → JSON refresh body
┌──────────────────────────────▼───────────────────────────────────────┐
│                         Healthkunj-Api                               │
│                   NestJS 10 · GCP Cloud Run                          │
│   32 modules  ·  multi-tenant  ·  RBAC  ·  MFA  ·  audit logging    │
│   Razorpay  ·  ABDM  ·  WhatsApp webhooks  ·  FCM push  ·  GST      │
└──────────────┬───────────────────────────────────────────────────────┘
               │
   ┌───────────┼─────────────────┐
   │           │                 │
┌──▼─────────┐ ┌─▼──────────┐ ┌─▼──────────────┐
│ Cloud SQL  │ │ Memorystore│ │ Cloud Storage  │
│ Postgres16 │ │ Redis 7    │ │ Docs · PDFs    │
│ Cloud SQL  │ │ private VPC│ │ Prescriptions  │
└────────────┘ └────────────┘ └────────────────┘
```

---

## Backend — `Healthkunj-Api`

**32 modules.** One deployed Cloud Run service handling every product and every role.

| Domain | Modules |
|---|---|
| Auth & Identity | `auth` · `users` · `sessions` · `tenants` |
| Clinic | `appointments` · `consultations` · `doctor` · `blocked-dates` · `clinical-evaluations` |
| Patient | `patient` · `health` · `referrals` · `case-forms` · `consent-logs` |
| Commerce | `shop` · `billing` · `payments` · `subscriptions` |
| Growth | `subscribers` · `contact` · `blog` · `content` · `n8n` |
| Operations | `admin` · `analytics` · `notifications` · `uploads` · `mailer` |
| Platform | `chat` · `corporate-wellness` · `inventory` · `abdm` |
| Infrastructure | `redis` |

### Key Platform Capabilities

**Authentication**
- JWT (15-min access token, 30-day httpOnly refresh cookie)
- Google OAuth via Passport.js
- MFA / TOTP enforcement for clinical staff (ADMIN, DOCTOR)
- Account lockout after failed attempts
- Per-device session management + revocation
- Keyless CI/CD via Workload Identity Federation

**Access Control**
- Role hierarchy: `USER · DOCTOR · ADMIN · SUPER_ADMIN`
- Feature flags per tenant (`FF_SUBSCRIPTIONS`, `FF_CLINICAL_EVALUATIONS`, `FF_CORPORATE_WELLNESS`, `FF_SAAS_PLATFORM`, `FF_ABDM`)
- Multi-tenant isolation — every query is scoped to `tenantId`

**Clinical**
- Appointment lifecycle (NEW → CONFIRMED → COMPLETED / CANCELLED)
- Telemedicine — Google Meet / Zoom meeting link generation
- Clinical evaluations — burden scoring (0–100), questionnaires, shared progress links
- ABDM / ABHA consent integration (Phase 1 — ABHA ID on patient records)
- Miasmatic classification in consultation notes
- Prescription-required flag on medicines + prescription approval flow
- Doctor availability, slot algorithm, multi-doctor scheduling

**Commerce & Billing**
- Razorpay orders, subscriptions, and webhook signature verification
- Sequential GST-compliant invoices (HK-INV-YYYY-NNNN)
- Shop with categories, coupons, wishlists, reviews, subscribe-and-save
- Corporate wellness packages with employee credit ledger
- Care subscription programs with monthly check-ins

**Compliance**
- DPDP — data deletion endpoint + email confirmation
- Consent logs for all ABDM operations
- Audit log interceptor on all mutations and sensitive GETs
- Signed GCS URLs (15-min expiry) for all medical documents — never in API responses

---

## Clinical Focus — 20 Conditions

Homeopathic treatment for India's most prevalent chronic conditions, led by **Dr. Meera Thakur, BHMS**:

PCOS · Thyroid · Asthma · Diabetes · Migraine · Autism · ADHD · Immunity · Psoriasis · Arthritis · Allergy · Cholesterol · Acne & Skin · Hair Loss · Obesity · Liver · Mental Health · Women's Health · Kidney Stones · Homeopathy for children

Clinical outcomes tracked per patient through structured evaluations. Progress shareable with patients via secure token links. Case forms submitted digitally before consultation.

---

## Automation — 88 n8n Workflows

All patient communications and operational triggers run through n8n. The API fires fire-and-forget webhooks; n8n owns the messaging layer — WhatsApp, email, SMS, and push.

| Domain | Workflows |
|---|---|
| Appointments | Booking confirmation · 24h reminder · 2h reminder · cancellation notice · rescheduling |
| WhatsApp | Patient onboarding · remedy reminders · refill alerts · broadcast campaigns (28 workflows) |
| Shop | Order placed · shipped · delivered · prescription decision · refill scheduling |
| Leads | Quiz completion · newsletter subscribe · referral tracking · case form · enquiry routing |
| Subscriptions | Enrolment · payment failed · renewal · check-in reminder |
| Ops | Feedback collection · SaaS tenant onboarding · billing alerts |
| Social | Facebook · Instagram · LinkedIn · X · Google Business Profile (23 workflows) |
| Website | SEO pings · blog distribution · sitemap update |

---

## Infrastructure

| Layer | Service | Detail |
|---|---|---|
| Cloud | Google Cloud Platform | Project `healthkunj-140624` · `asia-southeast1` |
| API | Cloud Run | Auto-scaling · 0→N · private ingress only |
| Web | Cloud Run | Next.js standalone build · custom domain |
| Database | Cloud SQL | PostgreSQL 16 · Unix socket via Auth Proxy sidecar |
| Cache | Memorystore | Redis 7 · private VPC · no public IP |
| Files | Cloud Storage | Medical documents · invoices · prescription images · logos |
| CDN | Firebase Hosting | Shop · Case Forms · Apps (sub-10ms global CDN) |
| Containers | Artifact Registry | `asia-southeast1-docker.pkg.dev/healthkunj-140624/healthkunj/` |
| CI/CD | GitHub Actions | Push to `main` → deploy. Workload Identity Federation — zero SA keys in secrets |
| Automation | Hostinger VPS | n8n self-hosted at sochish.cloud |

---

## Technology Stack

```
Language        TypeScript (strict mode across all repos)
Backend         NestJS 10 · TypeORM · class-validator · Passport.js · bcryptjs
Frontend        Next.js 14 App Router · Tailwind CSS · Lucide icons
Mobile          Expo SDK 54 · React Native · NativeWind v4 · Expo SecureStore
Database        PostgreSQL 16 · Redis 7
Payments        Razorpay (orders · subscriptions · webhooks · refunds)
Notifications   FCM push · WhatsApp Business API · Nodemailer SMTP
AI              Google Gemini (health calculators + suitability tools)
Compliance      ABDM / ABHA · DPDP · GST invoicing
Testing         Playwright E2E · smoke (every push) · full authenticated suite (nightly)
```

---

## Multi-Tenant SaaS

`Healthkunj-Api` is built multi-tenant from the ground up. Every table carries a `tenant_id`. Every query is scoped through `tenantWhere()`. Every feature is gated per tenant via `featuresEnabled` JSON in `tenant_configs`.

**The same platform that runs HealthKunj can run any AYUSH clinic in India** — with their own branding, subdomain, doctor roster, and feature set. This is the foundation for licensing HealthKunj as clinic management infrastructure to homeopathic, Ayurvedic, and Unani practitioners nationally.

---

## ABDM Compliance Roadmap

| Phase | Scope | Status |
|---|---|---|
| Phase 1 | ABHA ID on patient records · HFR/HPR clinic registration · ABHA consent on booking | Complete |
| Phase 2 | UHI Health Service Provider integration · real ABDM credentials | Planned (Month 4–5) |
| Phase 3 | NHCX sandbox integration | Planned (Month 9–10) |
| Phase 4 | HealthKunj as EUA on UHI · SaaS licensing to AYUSH clinics | Year 2–3 |

Our NestJS + PostgreSQL stack **is** the EHR — we do not use third-party EHR software.

---

## Status

| Product | Status |
|---|---|
| API + Web portal | Live — [healthkunj.com](https://healthkunj.com) |
| Mobile app (iOS + Android) | Code complete — pending App Store / Play Store review |
| Online shop | Live |
| Clinical intake forms | Live |
| Health calculators + suitability tests | Live |
| 88 n8n automation workflows | Staging — deploying to production |
| Corporate wellness module | Feature-flagged, ready for first client |
| SaaS multi-tenant platform | Feature-flagged, ready for pilot |

---

## Contact

**Clinic** — Kharadi, Pune, Maharashtra, India  
**Website** — [healthkunj.com](https://healthkunj.com)  
**Email** — contact@healthkunj.com

---

<sub>HealthKunj Clinics Private Limited &nbsp;·&nbsp; "House of Health" &nbsp;·&nbsp; Est. March 2026 &nbsp;·&nbsp; Kharadi, Pune</sub>
