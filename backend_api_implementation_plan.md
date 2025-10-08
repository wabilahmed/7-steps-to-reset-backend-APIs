# Backend API Implementation Plan

## 1. Guiding Principles
- **Privacy by design**: enforce strict RBAC, anonymize aggregated analytics, and secure personal data at rest and in transit.
- **Modular NestJS architecture**: feature-based modules with clear boundaries and testability.
- **Automation first**: adopt CI/CD, automated testing (unit/integration/e2e), linting, and documentation generation from day one.
- **Observability and resilience**: structured logging, health checks, metrics, and graceful degradation for background jobs and third-party integrations.

## 2. High-Level Architecture
1. **Application Layer** (NestJS): Feature modules for auth, organizations, wellness logging, scoring, reporting, notifications, compliance, and admin tooling.
2. **Persistence Layer** (TypeORM): PostgreSQL in production, SQLite in development; use migrations for schema evolution, repository pattern, and transactional services.
3. **Caching & Queues**: Redis-backed cache (short-lived lookups, rate limits) and Bull queues for asynchronous work (emails, push notifications, report generation).
4. **External Integrations**: Email provider abstraction (SendGrid/AWS SES), Firebase Admin SDK for push, storage driver interface (local/S3).
5. **DevOps**: Dockerized local environment, GitHub Actions for lint/test/build/deploy, PM2 or systemd for process management, environment secrets via `.env` + vaulting solution.

## 3. Module Breakdown & Responsibilities

### 3.1 Core Foundations
- **AppModule**: global config, validation pipe, logging, exception filters, rate limiting guard, Swagger setup.
- **ConfigModule**: environment schema validation (Joi/Zod), typed configuration service.
- **DatabaseModule**: TypeORM connection, migrations, repositories. Provide seeding scripts for base data (domains, templates).
- **HealthModule**: `/health` endpoint with checks for DB, Redis, queues, external services.

### 3.2 Identity & Access Management
- **AuthModule**:
  - Features: Email/password login, refresh token rotation, access token issuance (JWT), MFA (TOTP), password reset, email verification.
  - Components: Local strategy, JWT strategy, refresh token strategy, MFA guard, session revocation service, rate limiting on sensitive endpoints.
  - Data Models: `users`, `user_sessions`, `invitation_tokens`, `audit_logs` (log auth events).
  - Integrations: Email service for verification/reset; optional NextAuth compat endpoints.
  - Tests: Unit tests for guards/strategies, integration tests for login flows.
- **UsersModule**:
  - CRUD for user profiles (self-service & admin-managed), onboarding questionnaire, preferences (locale, notifications, theme, quiet hours), soft delete/reactivation.
  - Ensure field-level validation, privacy filtering (only user or admins with privileges can access PII).
- **OrganizationModule**:
  - Entities: `organizations`, `departments`, `organization_subscriptions`.
  - Capabilities: Create orgs, manage departments, assign roles (SUPER_ADMIN/BUSINESS_ADMIN/USER), invite flow, audit logging.
  - Scoped guards to ensure admins only access their org data; enforce aggregated views for Business Admins via database views/materialized queries.

### 3.3 Wellness Logging & Scoring
- **WellnessLogsModule**:
  - Entities: `wellness_logs` capturing daily entries per domain with timestamps and optional notes.
  - API: CRUD for daily logs, bulk upsert for multi-day import, validation of domain score ranges, offline sync endpoint for mobile (idempotent tokens).
  - Background job to reconcile duplicates from offline submissions.
- **ScoringModule**:
  - Maintain `wellness_scores` table storing rolling 7-day computations and domain totals.
  - Services to recalculate scores on log change, scheduled nightly recompute at 2 AM via Bull queue.
  - Provide endpoints for daily/weekly/quarterly trend data, Reset Circle metrics, and color-coded feedback based on thresholds.
  - Implement Warwick-Edinburgh Mental Wellbeing assessments: scoring algorithm, category mapping (Low/Moderate/High), store in `mental_wellbeing_assessments`.

### 3.4 Reporting & Analytics
- **ReportsModule**:
  - User-level weekly summaries: aggregate logs, streaks, suggestions. Generate PDF/CSV via templating service (e.g., handlebars + puppeteer) queued on-demand or scheduled (Mondays).
  - Organization reports: department comparisons, participation metrics, aggregated domain scores; restrict Business Admin visibility to anonymized data (minimum cohort size enforcement).
  - Scheduled jobs for monthly exports (1st of month) with storage in S3/local and email notification to admins.
  - API endpoints for report retrieval, filters, and asynchronous job status tracking.
- **AnalyticsModule**:
  - Super Admin dashboards: platform-wide adoption, system health metrics, subscription status.
  - Provide data for Admin Portal charts via aggregated SQL views.

### 3.5 Notifications & Engagement
- **NotificationsModule**:
  - Entities: `notifications`, `notification_templates`, `user_preferences`.
  - Services: Template management, personalization variables, queueing for email/push/in-app channels.
  - In-app notifications API with read/unread state, pagination, and quiet hours compliance.
- **Messaging Integrations**:
  - Email driver pattern: interface + provider-specific implementations (SendGrid/SES) selected via env.
  - Push service: Firebase Admin integration, multi-device token storage, token cleanup job.
  - Daily reminders (9 AM/6 PM) and inactive user alerts via Bull cron queues with per-user throttle.

### 3.6 Compliance & Security
- **ComplianceModule**:
  - GDPR requests: data export (JSON/CSV) and deletion workflows with audit logging.
  - Consent tracking for onboarding and notifications.
  - Privacy guardrails for aggregated data (minimum cohort sizes, suppression of small groups).
- **Security Enhancements**:
  - Helmet middleware, CSRF mitigation for admin portal interactions, input validation (class-validator), global rate limits.
  - Encryption for sensitive fields (e.g., MFA secret, refresh tokens) using envelope encryption (KMS or libsodium).
  - Audit trail service capturing key events (logins, data exports, role changes).

### 3.7 Documentation & Tooling
- Auto-generate Swagger/OpenAPI from decorators; publish JSON spec for client consumption.
- Provide Postman collection or Insomnia export.
- Maintain database ERD (dbdocs or similar) and architecture diagrams.
- Create developer onboarding docs: setup scripts, environment variables, run/test instructions.

## 4. Data Model Planning
- Define base entities with UUID primary keys, timestamps, soft delete columns (`deleted_at`), and indexes.
- Use TypeORM migrations for initial schema:
  1. Users & auth tables.
  2. Organizations, departments, invitations, subscriptions.
  3. Wellness logs, scores, assessments.
  4. Reports, notifications, templates, audit logs.
- Introduce database views/materialized views for aggregated analytics (e.g., `org_domain_scores_view`).
- Add JSONB columns for flexible payloads (e.g., `wellness_logs.domain_payload`).

## 5. Background Jobs & Scheduling
- Configure Bull queues: `emails`, `push`, `reports`, `scoring`, `maintenance`.
- Define workers with concurrency controls and retries.
- Schedule cron jobs via BullMQ repeatable jobs:
  - Daily reminders (09:00 & 18:00).
  - Nightly score recalculation (02:00).
  - Weekly user report generation (Monday 07:00).
  - Monthly organization reports (1st 06:00).
  - Inactive user nudges (daily scan).
  - Invitation cleanup and token expiration (hourly).
  - Automated DB backup triggers (daily/weekly, integrate with infrastructure scripts).

## 6. Testing Strategy
- **Unit Tests**: Service logic, guards, pipes, utilities (≥ 80% coverage requirement).
- **Integration Tests**: API endpoints using in-memory SQLite or test Postgres, verifying RBAC, privacy, and data integrity.
- **E2E Tests**: Critical flows (login, logging wellness, report request) using Nest testing utilities.
- **Contract Tests**: Validate mobile and admin portal clients using generated OpenAPI schemas.
- Integrate tests into GitHub Actions pipeline with parallel jobs.

## 7. Observability & Operations
- Implement structured logging (Winston) with correlation IDs per request.
- Provide `/metrics` endpoint for Prometheus scraping (via `@willsoto/nestjs-prometheus`).
- Add distributed tracing hooks (OpenTelemetry) for future expansion.
- Configure alerting for queue failures, job delays, and error rates.
- Provide runbooks for incident response and on-call escalation.

## 8. Delivery Roadmap (Backend Focus)
1. **Weeks 1-2 (Phase 1)**: Repo bootstrap, base Nest structure, config management, database setup, CI pipeline, health checks.
2. **Weeks 3-5 (Phase 2)**: Implement Auth, Users, Organizations/Departments, Invitations, Preferences, initial migrations, seed data, unit tests.
3. **Weeks 6-8 (Phase 3)**: Wellness logging, scoring, reports MVP, notification pipeline scaffolding, scheduling infrastructure, GDPR endpoints, Swagger docs.
4. **Weeks 9-11 (Phase 4)**: Expand analytics endpoints, finalize report generation, refine notification templates, implement MFA and compliance features, load/perf testing.
5. **Weeks 12-14**: Hardening—security review, privacy audits, failover testing, finalize documentation, readiness checklist for Admin/Mobile integrations.

## 9. Risks & Mitigations Specific to Backend
- **Privacy breaches**: Enforce RBAC guards, anonymization, implement static analysis for PII leaks, and conduct penetration testing.
- **Job queue overload**: Implement back-pressure monitoring, configure Redis cluster readiness, and provide fallback to synchronous processing for critical notifications.
- **External provider outages**: Build provider abstraction with circuit breakers, retry policies, and alternative providers configured via env.
- **Single-server constraint**: Optimize for resource usage (connection pooling, queue worker scaling) and document scaling roadmap (horizontal shards, read replicas) for future phases.

## 10. Acceptance Checklist
- All core modules implemented with tests ≥ 80% coverage.
- Security review complete: MFA, rate limiting, audit logs, encrypted sensitive data.
- Swagger spec published and shared with Admin Portal & Mobile teams.
- Background jobs and schedules operational with monitoring dashboards.
- Compliance workflows validated (export/delete) with legal/privacy stakeholders.
- Production deployment documented with rollback plan and health/metrics dashboards.
