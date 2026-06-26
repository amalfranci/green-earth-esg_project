# 🌿 Green Earth ESG — Sustainability Reporting Platform

A production-grade, multi-tenant SaaS backend for ESG (Environmental, Social, Governance) data collection, management, and regulatory reporting, with a focus on BRSR compliance aligned with SEBI guidelines (India).

---

## 🚀 Tech Stack

**Backend:** Node.js (ES Modules), Express.js, Prisma ORM  
**Database:** PostgreSQL (multi-schema — auth, brsr, emissions, materiality, topics, resources, audit, evidence, bulk_upload)  
**Architecture:** Microservices (10 services) · API Gateway · pnpm Workspaces · Turborepo  
**Auth:** JWT + Refresh Tokens · MFA via OTP · RBAC · Google SSO  
**Infra:** Docker · AWS EC2 / RDS · Nginx  
**Other:** Puppeteer (PDF generation) · Multer · Nodemailer · Pino · Axios (S2S)

---

## ⚙️ Key Features

### 🏗️ 1. Microservices Architecture

10 independent Node.js/Express services, each with its own Prisma schema and migrations:

| Service | Responsibility |
|---|---|
| gateway | Reverse proxy, rate limiting, CORS, auth routing |
| auth | Users, companies, JWT, MFA, RBAC, SSO |
| brsr-service | BRSR sections A/B/C, report aggregation, PDF export |
| emissions-service | Carbon accounting, fuel factors, calculations |
| resource-service | Water, waste, and environmental resource tracking |
| materiality | ESG issue prioritization matrix |
| topics | ESG topic/indicator catalogs |
| evidence-service | Proof document links for auditability |
| audit | Compliance audit trail across all services |
| bulk-upload-service | CSV mass import pipeline (energy/water/social data) |

### 🏢 2. Multi-Tenant Isolation

`companyId` is embedded in the JWT at login and scoped at the query level (`where: { companyId }`) across all services. `SYS_ADMIN` bypasses tenant scoping; all other roles are strictly isolated.

### 📊 3. BRSR Report Aggregation Pipeline

Full regulatory PDF generation coordinating 6 microservices:
1. Load base BRSR data (sections A/B/C) from the `brsr` schema
2. Parallel HTTP calls to auth, emissions, resource, materiality, topics, audit
3. Hydration layer merges remote data into the BRSR structure
4. Puppeteer renders HTML → PDF bytes returned to client

### 🔐 4. Auth & Access Control

- Three roles: `SYS_ADMIN` · `COMPANY_ADMIN` · `TEAM_MEMBER`
- MFA via email OTP (mandatory for company users)
- JWT access tokens + refresh token persistence
- Google OAuth2 SSO

### 📂 5. Bulk CSV Import

Async upload pipeline supporting modules: ENERGY · CARBON · WATER · WASTE · SOCIAL · GOVERNANCE. Validates, stages, and routes rows to the relevant domain service.

---

## 🧠 Architecture Highlights

- Single PostgreSQL instance with schema-per-domain isolation
- API gateway as the only public entry point (all services are internal)
- Shared packages (`@green-earth/shared`, `@green-earth/logger`) for DRY middleware, error handling, and validators
- Turborepo for parallel builds and caching across 10+ services
- Kubernetes manifests included for EKS deployment (`RUN_MODE=container` for inter-service DNS)

---

## 🛠️ Local Setup

1. Install PostgreSQL and create database `green_earth_esg`
2. `pnpm install` (from repo root)
3. Configure `.env` per service (`DATABASE_URL`, `JWT_SECRET`, etc.)
4. `pnpm run prisma:generate && pnpm run prisma:migrate` per service
5. `pnpm dev` — starts all services via Turborepo

Prisma Studio (all schemas): `pnpm run studio:all` (ports 5555–5559)

---

## 👤 Roles

| Role | Access |
|---|---|
| SYS_ADMIN | Platform-wide; manages all companies and users |
| COMPANY_ADMIN | Onboarding, frameworks, business units, team members |
| TEAM_MEMBER | Data entry, evidence upload, BRSR sections |
