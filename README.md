# ReputIQ AI — AI-Powered Reputation Management & Local Business Growth SaaS

ReputIQ AI is a production-grade, secure, scalable, multi-tenant SaaS application designed for digital marketing agencies, freelancers, and multi-location local businesses.

---

## 1. System Architecture Overview

```
                      ┌──────────────────────────────────────────────┐
                      │              React Frontend SPA              │
                      │   (React 18, TypeScript, Vite, Tailwind CSS) │
                      └──────────────────────┬───────────────────────┘
                                             │ HTTPS / JWT
                                             ▼
                      ┌──────────────────────────────────────────────┐
                      │          Spring Boot Core Backend            │
                      │  (Java 17, Spring Boot 4.0.6, Spring Sec 6)  │
                      └───────┬──────────────┬──────────────┬────────┘
                              │              │              │
           ┌──────────────────┘              │              └──────────────────┐
           ▼                                 ▼                                 ▼
┌──────────────────────┐          ┌──────────────────────┐          ┌──────────────────────┐
│   PostgreSQL / Neon  │          │   FastAPI AI Service │          │ Google Business API  │
│  (Flyway Migrations, │          │ (Python 3.11, Gemini │          │ (OAuth 2.0, Reviews, │
│  Tenant RLS/Filters) │          │  Flash & Guardrails) │          │ Replies, Locations)  │
└──────────────────────┘          └──────────────────────┘          └──────────────────────┘
```

### Architectural Principles
* **Spring Boot Authoritative Rule:** Spring Boot controls authentication, authorization, multi-tenant data access, business workflows, audit logs, and outbound Google Business Profile publishing.
* **FastAPI Stateless AI Engine:** FastAPI executes NLP sentiment analysis, risk categorization, contextual reply drafting, and marketing poster generation. AI services never access the production database directly or independently publish to Google.
* **Anti-Review Gating Guarantee:** Full compliance with Google Review policies. All customers scanning QR codes receive the identical direct Google Review URL regardless of rating (1–5 stars) or private sentiment.
* **Multi-Tenant Boundary Isolation:** Tenant boundaries are strictly validated at both the repository and service layers. Agencies and businesses cannot read or modify resources belonging to other tenants.

---

## 2. Core Modules Implemented

| Module | Description | Backend Engine | Frontend Route |
| :--- | :--- | :--- | :--- |
| **Module 1: Authentication & RBAC** | JWT access/refresh token rotation, BCrypt password hashing, role-based authorization (`SUPER_ADMIN`, `AGENCY_OWNER`, `AGENCY_STAFF`, `BUSINESS_OWNER`, `BUSINESS_STAFF`). | Spring Security 6, JJWT | `/login`, `/register` |
| **Module 2: Multi-Tenant Architecture** | Agency workspaces, staff invitation with role management, client business onboarding, and strict tenant isolation. | `TenantSecurityService`, `AgencyService` | `/agency/settings`, `/businesses` |
| **Module 3: QR Code Generator** | Cryptographic URL-safe touchpoint tokens, ZXing vector SVG and PNG rendering, custom brand palettes, and downloadable assets. | `QrCodeService`, ZXing | `/qr-codes` |
| **Module 4: Customer Feedback (Anti-Gating)** | Mobile-responsive touch rating experience (`/q/{token}`). Dual-action design: saves private feedback while simultaneously presenting official Google Review link to all users. | `CustomerFeedbackService` | `/q/:token`, `/feedback` |
| **Module 5: Google Business Profile Integration** | OAuth 2.0 authorization code exchange, AES-256-GCM encrypted token storage, GBP account and storefront location discovery, and link orchestration. | `GoogleIntegrationService`, GBP Client | `/google/integration` |
| **Module 6: AI Review Agent** | Sentiment analysis, concern classification, hallucination prevention via approved facts, escalation for high-risk complaints (food poisoning, legal threats), and Human-in-the-Loop review approval. | `AiDecisionEngine`, `GoogleReplyPublisher` | `/reviews`, `/ai-agent` |
| **Module 7: AI Analytics Dashboard** | Dual-channel reporting strictly distinguishing public Google metrics from private QR feedback. Star distribution, sentiment tracking, topic frequency, and actionable growth recommendations. | `AnalyticsService` | `/analytics` |
| **Module 8: AI Poster Generator** | Occasion-based marketing poster synthesis (`PROMOTIONAL`, `FESTIVAL`, `DISCOUNT`, `EVENT`), brand color palettes, multi-aspect ratios (1:1, 9:16, 16:9), and gallery management. | `PosterService`, FastAPI Poster Generator | `/posters` |
| **Module 9: White-Label SaaS & Custom Domains** | Agency custom branding (logo, favicon, brand colors, custom CSS). Domain registration with automated TXT/CNAME verification challenge and unauthenticated public theme resolution. | `DomainService`, `PublicBrandingController` | `/agency/settings` |
| **Module 10: Subscription & Billing Management** | SaaS tiers (`STARTER`, `PROFESSIONAL`, `AGENCY`, `ENTERPRISE`), usage metering for AI replies & posters, and strict quota enforcement (HTTP 429). | `BillingService`, `BillingController` | `/billing` |

---

## 3. Technology Stack

- **Backend Core:** Java 17, Spring Boot 4.0.6, Spring Security 6, Spring Data JPA, Hibernate 7, JJWT 0.12.6, ZXing 3.5.3, Flyway 10, Maven.
- **AI Microservice:** Python 3.11, FastAPI 0.110+, Pydantic v2, Google GenAI SDK (Gemini 1.5/2.0 Flash), Uvicorn.
- **Database & Cache:** Neon PostgreSQL (production) / H2 in-memory test profile, Redis 7 Alpine.
- **Frontend SPA:** React 18, TypeScript, Vite, Tailwind CSS, TanStack Query, Axios, Lucide Icons, React Router v6.
- **Containerization:** Docker multi-stage builds, Docker Compose, Nginx reverse proxy.

---

## 4. Quick Start with Docker Compose

To launch the full ReputIQ AI platform with all dependent services:

```bash
# 1. Clone repository & configure environment variables
git clone https://github.com/your-org/reputiq-ai.git
cd reputiq-ai
cp .env.example .env

# 2. Build and launch all containers
docker-compose up -d --build

# 3. Check container status
docker-compose ps
```

The application will be accessible at:
* **Frontend Web Application:** `http://localhost:80` (or `http://localhost:3000`)
* **Core REST Backend API:** `http://localhost:8080/api/v1` (Swagger UI at `http://localhost:8080/swagger-ui.html`)
* **FastAPI AI Service:** `http://localhost:8000` (OpenAPI Docs at `http://localhost:8000/docs`)
* **PostgreSQL Database:** `localhost:5432` (`reputiq_db`)
* **Redis Cache:** `localhost:6379`

---

## 5. Local Development Setup

### Prerequisites
- JDK 17+ (`java -version`)
- Python 3.11+ (`python --version`)
- Node.js 18+ & npm (`node -v`)

### 1. Core Backend (Spring Boot)
```bash
cd backend-core
./mvnw clean compile
./mvnw spring-boot:run
```

### 2. AI Microservice (FastAPI)
```bash
cd ai-service
python -m venv venv
# Windows:
venv\Scripts\activate
# Linux/macOS:
source venv/bin/activate

pip install -r requirements.txt
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload
```

### 3. Frontend (React + Vite)
```bash
cd frontend
npm install
npm run dev
```

---

## 6. Running Automated Verification Tests

### Backend Unit & Integration Tests (21 Test Suites)
```bash
cd backend-core
./mvnw test
```
*Tests cover JWT authentication, multi-tenant agency isolation, QR token generation, anti-gating review parity, Google OAuth AES-256 token encryption, GBP review synchronization, AI decision engine approval workflows, dual-channel analytics, AI poster creation, custom domain DNS verification, and SaaS subscription quota enforcement.*

### Python AI Service Tests (10 Test Cases)
```bash
cd ai-service
pytest -v
```
*Tests cover secret header authentication, sentiment scoring, anti-hallucination guardrails, mixed sentiment handling, critical complaint safety detection, and promotional poster prompt synthesis.*

### Frontend Production Build
```bash
cd frontend
npm run build
```
*Compiles all TypeScript source modules, runs typecheck with `tsc`, and outputs optimized production assets via Vite.*

---

## 7. Security & Compliance Highlights

1. **AES-256-GCM OAuth Credential Storage:** Google OAuth refresh tokens and API credentials are encrypted with Galois/Counter Mode (GCM) using a 256-bit AES master key with 96-bit randomized initialization vectors.
2. **Deterministic High-Risk Escalation:** AI will never automatically respond to reviews alleging food poisoning, injuries, safety violations, discrimination, or legal litigation. High-risk reviews are automatically placed in the supervisor approval queue.
3. **Strict Multi-Tenant Verification:** Every write and read operation confirms that the requesting authenticated principal belongs to the active tenant owning the target business, location, review, or poster.
4. **Anti-Review Gating Parity:** The customer feedback endpoint `/q/{token}` prominently and unconditionally provides the direct Google Business Profile review write link to 100% of customers, strictly upholding Google Maps user-generated content policies.
