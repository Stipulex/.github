<div align="center">

<img src="https://i.postimg.cc/tJytcc7L/stipulex-changelog.gif" width="640" alt="Stipulex" />

# High-Fidelity Logic. Engineered in Redwood City.

*Hard data for soft skills. Deep analysis that scales, without the billable hours.*

</div>

---

## What We Build

Stipulex is a contract intelligence engine — not an AI tool. It combines a proprietary jurisdiction compliance database, a deterministic cross-reference engine, and multi-model AI analysis to surface risk flags, a proprietary Fairness Score, and a plain-English executive intelligence brief in under 90 seconds.

The defensible asset is the **data + engine + validation architecture**. AI models are one input — validated and cross-checked against curated statutory and regulatory data covering California and Federal jurisdictions at launch, expanding to all 50 states.

> Contract intelligence. Not legal advice — just the most logical precursor to it.

---

## Engineering Standards

Stipulex is built for the compliance requirements ahead — SOC 2 Type II, HIPAA BAA, and FedRAMP Moderate — without gold-plating controls the current phase doesn't need. Every architectural decision has a seam for the next requirement.

**Infrastructure**
- Distributed rate limiting via Redis sorted sets — sliding-window algorithm, atomic pipeline, shared across all server processes, survives restarts
- Compliance data cached in Redis with 5-minute TTL — fail-open: Redis outage falls through to Postgres transparently, no analysis blocked
- PDF reports rendered once and persisted as binary — zero re-render cost on subsequent access; backwards-compatible fallback for pre-cache records
- Query performance tracked via `pg_stat_statements` — every query measured from day one, no instrumentation required at investigation time

**Semantic Search**
- 98 jurisdiction compliance rules embedded with 1536-dimensional vectors stored in Postgres via `pgvector`
- Dual-path matching: keyword engine + vector similarity running in parallel, results merged — neither path removable without dropping recall
- Paraphrased clause language caught by semantic similarity; standard legal language caught by keyword; both required

**Observability**
- Structured NDJSON logging throughout — PM2, Datadog, and CloudWatch compatible natively
- PII and document content redacted at the serialization layer — `documentText`, `prompt`, `completion`, `apiKey`, `email`, `phone`, and all known-sensitive fields never appear in logs
- All AI provider connections managed through a single central module — DPA enforcement, zero-training flags, vendor audit hooks, and failover all have one seam

**Type Safety & Correctness**
- TypeScript strict mode end-to-end — zero `any` suppressions, all hook dependency arrays explicit and correct
- Zod runtime validation at every engine boundary — impossible states are unrepresentable
- Boot-time environment validation — missing configuration surfaces as a single descriptive error at startup, not a cryptic crash mid-analysis
- 90-second hard SLA enforced at the engine level

**Stack**

![Next.js](https://img.shields.io/badge/Next.js_16-black?style=flat-square&logo=next.js)
![TypeScript](https://img.shields.io/badge/TypeScript_5.5-3178C6?style=flat-square&logo=typescript&logoColor=white)
![React](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black)
![Postgres](https://img.shields.io/badge/PostgreSQL_16-4169E1?style=flat-square&logo=postgresql&logoColor=white)
![pgvector](https://img.shields.io/badge/pgvector-semantic_search-4169E1?style=flat-square)
![Redis](https://img.shields.io/badge/Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![Zod](https://img.shields.io/badge/Zod-3E67B1?style=flat-square&logo=zod&logoColor=white)
![Drizzle](https://img.shields.io/badge/Drizzle_ORM-C5F74F?style=flat-square&logo=drizzle&logoColor=black)

---

## Changelog &nbsp;·&nbsp; Updated April 22, 2026

This update covers a significant infrastructure and accuracy sprint across the Stipulex engine. Changes span reliability, analysis correctness, performance, database architecture, and developer tooling.

---

### ✨ New Capabilities

**Contract-Type Aware Analysis**
The analysis engine now recognizes document-specific scoring context. Proprietary Information and Invention Assignment agreements (PIIAs) are a foundational legal instrument for startups — and they behave differently than commercial contracts. The engine now detects these documents and applies appropriate analysis logic: IP assignment is expected and not penalized, while the analysis focuses on what actually matters — compensation fairness, assignment scope, statutory compliance notices, and dual-party exposure where both signatories are making independent declarations. Fairness Scores for PIIAs now accurately reflect the signer's real position rather than applying commercial-contract penalties that don't apply.

**Instant Report Delivery**
Reports generated during analysis are now stored and served instantly on subsequent access. Previously, every download triggered a full re-render of the annotated PDF. That render cost has been eliminated — the report is generated once during analysis and delivered from storage on every subsequent request. For demo and presentation scenarios, this means zero latency between "View Report" and the PDF appearing.

**Analysis Cache with Configurable Retention**
A document-level cache is now available for development and testing workflows, gated behind an explicit configuration flag that is off by default in all demo and production environments. This preserves Stipulex's zero-retention posture for live use while giving engineers a fast inner loop during development — re-running the same contract skips API costs without affecting the demo experience.

---

### ⚡ Reliability & Infrastructure

**Distributed Rate Limiting**
Request rate limits are now enforced at the infrastructure level via Redis, using a sliding-window algorithm. The previous implementation was per-process and reset on every server restart — meaning a deployment or crash would reset everyone's window. The new implementation is shared across all server processes and survives restarts. 10 analyses per hour per IP, enforced consistently.

**Resilient Compliance Data Caching**
The jurisdiction compliance ruleset — the core data that powers Stipulex's analysis — is now cached in Redis with a 5-minute TTL shared across all server processes. The previous per-process cache meant every new worker process hit the database cold. The new cache is fail-open: if Redis is unavailable for any reason, the engine falls through to the database transparently. No analysis is blocked by a cache outage.

**Semantic Search Infrastructure — Live**
All 98 jurisdiction compliance rules now have semantic embeddings stored in the database, enabling vector similarity search against contract clauses. This is the foundation for the next generation of compliance matching — paraphrased clauses that don't use exact statutory keywords will now be catchable by semantic similarity rather than keyword matching alone. The schema additions supporting this capability are fully in place; the retrieval layer is next.

**Slow Query Visibility**
Query performance tracking is now active on the production database. Every query is measured automatically — when retrieval performance needs investigation, the data is already there. No instrumentation changes required at that point.

---

### 🔧 Fixes

**Development Tooling — Database CLI**
Database schema management commands were silently failing in terminal sessions because the CLI tool doesn't inherit shell environment variables. Fixed by explicitly loading the environment file at configuration time — `db:push`, `db:generate`, and `db:studio` now work correctly from any terminal without manual setup steps.

**TypeScript Strict Mode — Audit Pass**
A sweep of the codebase tightened all remaining `any` type suppressions and corrected React hook dependency arrays that were causing unnecessary re-renders in the PDF viewer component. ESLint suppression comments were removed; all hooks now declare explicit, correct dependency arrays.

---

### 🏗️ Architecture

**Centralized AI Provider Management**
All external AI provider connections are now managed through a single module. Previously each analysis stage instantiated its own SDK clients independently — five separate instantiation points across three files. This is now one place. Future requirements like data processing agreement enforcement, zero-training flags, vendor audit hooks, and failover configuration all have a single seam to attach to rather than requiring changes across the entire codebase.

**Structured Logging**
All engine output now emits structured JSON to stdout/stderr, compatible with PM2, Datadog, and CloudWatch out of the box. Sensitive fields — document content, API credentials, PII — are redacted at the serialization layer before any log entry is written. This is the observability foundation required for SOC 2 and HIPAA audit trails.

**Boot-Time Environment Validation**
Missing environment variables now surface as a single descriptive error at startup listing every missing key — not as cryptic crashes mid-analysis when a specific code path is first reached. The error tells you exactly what's missing and where to set it.

**Enterprise Folder Structure**
The codebase was reorganized to support the scale and compliance requirements ahead: AI provider management, observability, configuration validation, compliance data, and database queries are each in dedicated, properly namespaced modules. The database query layer was split from a single monolithic file into domain-scoped modules with a barrel export — zero impact on existing callers, significant improvement to maintainability as new query types are added.

---

<div align="center">

*Stipulex decodes the architecture of your agreements.*
*We provide the map; you provide the counsel.*
*Not legal advice — just the most logical precursor to it.*

</div>
