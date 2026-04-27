<div align="center">

<img src="https://i.postimg.cc/tJytcc7L/stipulex-changelog.gif" width="640" alt="Stipulex" />

# High-Fidelity Logic. Engineered in Redwood City.

*Hard data for soft skills. Deep analysis that scales, without the billable hours.*

</div>

---

## What We Build

Stipulex is a contract intelligence engine — not an AI tool. It combines a proprietary jurisdiction compliance database, a deterministic cross-reference engine, and multi-model AI analysis to surface risk flags, a Fairness Score, and a plain-English executive intelligence brief in under 90 seconds.

> Contract intelligence. Not legal advice — just the most logical precursor to it.

---

## Engineering Standards

Stipulex is built with the compliance requirements ahead in mind — without gold-plating controls the current phase doesn't need. Every architectural decision has a seam for the next requirement.

**Infrastructure**
- Distributed rate limiting via Redis — shared across all server processes, survives restarts
- Compliance data cached in Redis with a short-lived TTL — fail-open: Redis outage falls through to Postgres transparently, no analysis blocked
- PDF reports rendered once and persisted as binary — zero re-render cost on subsequent access; backwards-compatible fallback for pre-cache records
- Query performance tracked via `pg_stat_statements` — every query measured from day one, no instrumentation required at investigation time

**Semantic Search**
- Hundreds of jurisdiction compliance rules with high-dimensional semantic embeddings for similarity-based clause matching
- Paraphrased clause language and standard legal language are both matched — semantic similarity and direct statutory matching work in tandem

**Authentication**
- Full in-house authentication stack — no third-party auth library dependencies
- Industry-standard password hashing, one-time token flows, and server-side session management
- Role-based rate limiting tied to account identity, not IP alone
- Brute-force protection across all credential endpoints
- Email-driven account setup, recovery, and admin invite flows

**Observability**
- Structured NDJSON logging throughout — PM2, Datadog, and CloudWatch compatible natively
- PII and document content redacted at the serialization layer — sensitive fields never appear in logs
- All AI provider connections managed through a single central module — DPA enforcement, zero-training flags, vendor audit hooks, and failover all have one seam

**Type Safety & Correctness**
- TypeScript strict mode end-to-end — zero `any` suppressions, all hook dependency arrays explicit and correct
- Zod runtime validation at every engine boundary — impossible states are unrepresentable
- Boot-time environment validation — missing configuration surfaces as a single descriptive error at startup, not a cryptic crash mid-analysis
- 90-second hard SLA enforced at the engine level

**Stack**

![Next.js](https://img.shields.io/badge/Next.js_16-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript_6.0.3-3178C6?style=flat-square&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black)
![Postgres](https://img.shields.io/badge/PostgreSQL_16-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B1?style=flat-square&logo=zod&logoColor=white)
![Drizzle](https://img.shields.io/badge/Drizzle_ORM-C5F74F?style=flat-square&logo=drizzle&logoColor=black)

---

## Changelog &nbsp;·&nbsp; April 23, 2026

### TypeScript 6.0.3 + Authentication System &nbsp;·&nbsp; [PR #21](https://github.com/Stipulex/stipulex-demo/issues/21)

**TypeScript 6.0.3 Upgrade**
- Migrated from TypeScript 5.5 to 6.0.3
- Patched open CVEs in transitive dependencies
- Resolved all Next.js 16 compatibility requirements — middleware conventions, edge runtime compatibility, and build configuration updated to match current framework contracts

**Authentication System**

Replaced a third-party auth library (removed due to a CVE in its dependency tree) with a purpose-built authentication stack designed for Stipulex's compliance posture from the ground up.

- **Credential management:** Industry-standard password hashing; one-time tokens for account setup and password recovery; tokens are single-use and expire
- **Sessions:** JWT-based with Redis backing — sessions are server-side invalidatable on logout; algorithm and scope pinned
- **Rate limiting:** Per-identity enforcement tied to account tier, not IP alone; brute-force protection active on all credential endpoints
- **Account flows:** Registration (invite-gated), set-password, login, logout, forgot-password, reset-password, admin setup, and account suspension
- **Admin visibility:** Usage dashboard for account-level activity

**Database**
- Schema extended to support user accounts, authentication tokens, invite codes, and onboarding profiles
- All analyses now carry a user foreign key — no orphaned records
- Legacy schema artifacts dropped; database fully migrated on VPS
---

## Changelog &nbsp;·&nbsp; April 22, 2026

This update covers a significant infrastructure and accuracy sprint across the Stipulex engine. Changes span reliability, analysis correctness, performance, database architecture, and developer tooling.

A comprehensive internal security audit was completed prior to public access. All findings were resolved before launch.

---

### New Capabilities

**Contract-Type Aware Analysis**
The analysis engine now recognizes document-specific context and applies appropriate scoring logic for different agreement types — surfacing the signals that matter for each contract, not generic penalties that don't apply.

**Instant Report Delivery**
Reports generated during analysis are now stored and served instantly on subsequent access. Previously, every download triggered a full re-render of the annotated PDF. That render cost has been eliminated — the report is generated once during analysis and delivered from storage on every subsequent request. For demo and presentation scenarios, this means zero latency between "View Report" and the PDF appearing.

**Analysis Cache with Configurable Retention**
A document-level cache is available for development and testing workflows, gated behind an explicit configuration flag that is off by default in all environments. This preserves Stipulex's default zero-retention posture for live use while giving engineers a fast inner loop during development.

---

### Reliability & Infrastructure

**Distributed Rate Limiting**
Request rate limits are now enforced at the infrastructure level via Redis. The previous implementation was per-process and reset on every server restart — meaning a deployment or crash would reset everyone's window. The new implementation is shared across all server processes and survives restarts.

**Resilient Compliance Data Caching**
The jurisdiction compliance ruleset is now cached in Redis shared across all server processes. The previous per-process cache meant every new worker process hit the database cold. The new cache is fail-open: if Redis is unavailable, the engine falls through to the database transparently. No analysis is blocked by a cache outage.

**Semantic Search Infrastructure — Live**
Jurisdiction compliance rules now have semantic embeddings enabling vector similarity search against contract clauses — paraphrased clauses that don't use exact statutory keywords are now catchable by semantic similarity.

**Slow Query Visibility**
Query performance tracking is now active. Every query is measured automatically — when retrieval performance needs investigation, the data is already there.

---

### Fixes

**Development Tooling — Database CLI**
Database schema management commands were silently failing in terminal sessions. Fixed by explicitly loading the environment file at configuration time — `db:push`, `db:generate`, and `db:studio` now work correctly from any terminal without manual setup steps.

**TypeScript Strict Mode — Audit Pass**
Tightened remaining `any` type suppressions and corrected React hook dependency arrays causing unnecessary re-renders in the PDF viewer. All hooks now declare explicit, correct dependency arrays.

---

### Architecture

**Centralized AI Provider Management**
All external AI provider connections now flow through a single management module. Data processing agreement enforcement, zero-training flags, vendor audit hooks, and failover configuration all have one seam.

**Structured Logging**
All engine output now emits structured JSON to stdout/stderr, compatible with PM2, Datadog, and CloudWatch out of the box. Sensitive fields are redacted at the serialization layer before any log entry is written.

**Boot-Time Environment Validation**
Missing environment variables now surface as a single descriptive error at startup listing every missing key — not as cryptic crashes mid-analysis.

**Enterprise Folder Structure**
AI provider management, observability, configuration validation, compliance data, and database queries are each in dedicated, properly namespaced modules. The database query layer was split into domain-scoped modules with a barrel export — zero impact on existing callers.

---

<div align="center">

*Stipulex decodes the architecture of your agreements.*
*We provide the map; you provide the counsel.*
*Not legal advice — just the most logical precursor to it.*

</div>
