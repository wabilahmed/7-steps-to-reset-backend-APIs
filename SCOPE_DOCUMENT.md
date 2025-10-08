# 7 Steps to Reset – Project Scope Document

## 1. Project Overview
- **Vision**: Deliver a privacy-first B2B wellness platform that helps organizations track and improve employee wellbeing across seven lifestyle domains while keeping individual scores confidential.
- **Business Model**: Subscription-based access for organizations, delivered through a simplified single-server deployment shared across all tenants.
- **Products in Scope**:
  - **Backend API** (NestJS/TypeScript) powering data, scoring, authentication, notifications, and reporting.
  - **Admin Portal** (Next.js/TypeScript) for Super Admin and Business Admin workflows.
  - **Mobile App** (Flutter) for end-user wellness logging and insights on iOS, Android, and Web.

## 2. Objectives & Success Metrics
- **Primary Objectives**:
  - Launch an end-to-end wellness tracking solution that supports employees logging daily habits, organizations accessing aggregated insights, and platform operators managing the ecosystem.
  - Maintain strong privacy controls so organizations only see anonymized/aggregated employee data.
- **Technical KPIs**: API p95 < 200 ms, uptime > 99.9%, mobile crash rate < 0.5%, automated test coverage ≥ 80%.
- **Business KPIs**: ≥ 70% user activation, ≥ 60% weekly active users, ≥ 90% organization retention, ≥ 4.5/5 user satisfaction.

## 3. Stakeholders & Target Users
- **Super Admin (Platform Owner)**: Manages all organizations, system settings, analytics, content, and notification templates.
- **Business Admin (Organization Admin)**: Manages organization roster, departments, invitations, aggregated analytics, and report exports.
- **End User (Employee)**: Accesses mobile app to log wellness activities, view scores, assessments, and personalized insights.

## 4. In-Scope Capabilities
### 4.1 Wellness Program Foundations
- Seven domains with domain-specific scoring: Family & Friends (max 20), Mental Practice (21), Nutrition (21), Physical Activity (14), Work-Time Balance (7), Sleep (7), Leisure (10).
- Rolling 7-day score computation, Reset Circle visualization, weekly/monthly/quarterly trends, historical data store, color-coded feedback (Red/Yellow/Green).
- Warwick-Edinburgh Mental Wellbeing Scale assessments with categorized outcomes (Low/Moderate/High).

### 4.2 User & Organization Management
- Registration via invitations or organization codes, onboarding questionnaire, JWT auth with refresh tokens, password reset, email verification, optional MFA.
- Organization hierarchy: organizations → departments; roles (SUPER_ADMIN, BUSINESS_ADMIN, USER); soft deletes; audit logging.
- User preferences (notification channels, frequency, quiet hours, locale, theme).

### 4.3 Reporting & Analytics
- Individual reports: weekly summaries, trend analysis, downloadable PDF/CSV.
- Organization-level reports: department comparisons, participation metrics, aggregated scores, scheduled monthly exports.
- Super Admin analytics: platform-wide adoption, system health dashboards.

### 4.4 Notifications & Engagement
- Email (SendGrid/AWS SES) templates for welcomes, reminders, reports, password resets.
- Push notifications via Firebase Cloud Messaging, multi-device token management.
- In-app notifications, daily reminders (9 AM/6 PM), inactive user alerts, expiring invitation alerts.

### 4.5 Scheduling & Background Jobs
- Bull/Redis-based job queues for email, push, and report processing.
- Cron schedules: daily reminders, wellness score recalculation (2 AM), weekly user reports (Mondays), monthly organization reports (1st), inactive user nudges, invitation cleanup, automated database backups.

### 4.6 Compliance & Security
- GDPR-aligned features: data export, deletion, consent tracking, audit trails.
- Rate limiting, Helmet security headers, input validation, throttling, session management, token revocation, encrypted sensitive data.
- Privacy guardrails ensuring Business Admins only see aggregated/anonymized data.

### 4.7 Documentation & Tooling
- Swagger/OpenAPI docs, database ERD, design system tokens, deployment runbooks, incident response plan, user/admin guides, onboarding videos or FAQs.

## 5. Out-of-Scope (Phase 1)
- AI-driven coaching, wearable integrations, white-label theming, partner APIs, advanced gamification beyond basic streaks and badges, multi-region data residency.

## 6. Technical Architecture
- **Backend**: NestJS (TypeScript), TypeORM, PostgreSQL (prod) / SQLite (dev), Redis for cache/queues, Bull, Swagger, Jest, PM2, Docker, CI/CD via GitHub Actions.
- **Admin Portal**: Next.js App Router, TypeScript, Tailwind CSS + shadcn/ui, Zustand/React Query, NextAuth with JWT, Recharts, Axios-based API client.
- **Mobile App**: Flutter 3, Dart, Clean Architecture with BLoC or Riverpod, Dio networking, secure storage, Hive/SQLite cache, fl_chart, Firebase Analytics & FCM, support for iOS/Android/Web.
- **Infrastructure**: Single server (EC2-class) hosting backend and admin portal, S3 (or equivalent) for file storage, Nginx reverse proxy, SSL certificates (Let’s Encrypt/ACM), Dockerized services, GitHub Actions pipelines, application-level logging via Winston in staging with ELK/New Relic and automated WAL backups deferred to production rollout.

## 7. Data Model Scope
- Core tables: organizations, departments, users, user_preferences, user_onboarding, wellness_logs, wellness_scores, mental_wellbeing_assessments, notifications, user_sessions, organization_reports, user_reports, organization_subscriptions, invitation_tokens, notification_templates, content_tips, audit_logs.
- All primary keys UUIDv4, timestamps (created_at/updated_at), soft deletes for critical entities, JSONB for flexible domain payloads, indexes for FK and frequent filters.

## 8. Integrations & External Services
- **Email**: SendGrid or AWS SES (configurable driver via env vars).
- **Push**: Firebase Admin SDK (service account stored securely, path configurable).
- **Storage**: Local uploads (staging) with S3 switch via `FILE_STORAGE_DRIVER`.
- **Auth Support**: bcrypt hashing, JWT secrets, optional NextAuth providers.
- **CI/CD**: GitHub Actions for lint/test/build/deploy, PM2 for process management.

## 9. Assumptions & Dependencies
- Stakeholders provide finalized brand assets, copy, and legal content before UI polish.
- Infrastructure budgets cover required AWS (or equivalent) services.
- Third-party services (SendGrid, Firebase) accounts and credentials provisioned by project sponsor.
- Product decisions (MVP vs stretch) validated before development of optional features.
- Privacy/legal review available for GDPR implementation validation.

## 10. Constraints
- Must maintain single shared environment per architecture update (no per-tenant deployments).
- Individual user data cannot be exposed to Business Admins.
- Support offline-ready mobile experience with secure local caching.
- Ensure accessibility compliance (WCAG AA) in Admin Portal and mobile UI.
- Timeline target from kick-off to production launch: ≤ 22 weeks.

## 11. Implementation Phases & Milestones (Fresh Build)
1. **Phase 1 – Foundations (Weeks 1-2)**
   - Repository setup, CI/CD pipeline, coding standards, environment provisioning, design system tokens, base data models.
2. **Phase 2 – Core Backend (Weeks 3-5)**
   - Auth, organizations, departments, invitations, user management, wellness logging APIs, scoring services, database migrations.
3. **Phase 3 – Core Features (Weeks 6-8)**
   - Reports, notification pipeline, scheduler jobs, analytics endpoints, GDPR workflows, Swagger docs.
4. **Phase 4 – Admin Portal (Weeks 9-11)**
   - Auth integration, dashboards, user/org management, reports UI, notifications center, export tooling.
5. **Phase 5 – Mobile App (Weeks 12-19)**
   - Authentication, onboarding, daily log flows, scoring visualizations, reports, notifications, offline cache, accessibility polish.
6. **Phase 6 – Notifications & Engagement (Weeks 20-21)**
   - Push/email templates, preference management, real-time updates, socket integration.
7. **Phase 7 – Quality & Launch (Weeks 22+)**
   - Automated tests, performance tuning, security audit, staging/production deployment, training, launch readiness checklist.

## 12. Deliverables
- Production-ready backend service with documented API.
- Responsive Admin Portal with role-based UX and integration tests.
- Flutter mobile apps (iOS, Android, Web) with beta-ready builds, CI workflows, and store submission assets.
- Infrastructure scripts (Docker Compose, PM2 configs, deployment automation).
- Documentation package: API spec, ERD, design system, runbooks, onboarding guides.
- Test suites: backend unit/integration, admin UI unit/e2e, mobile widget/integration.
- Monitoring & alerting setup with dashboards and incident response procedure.

## 13. Acceptance Criteria
- End-to-end user journey demonstrable across all roles with test data.
- All high-priority user stories accepted by stakeholders.
- Security, performance, and accessibility benchmarks met.
- Documentation reviewed and approved (technical + user-facing).
- Production environment deployed with success logs, monitoring, and rollback plan.

## 14. Risks & Mitigations
- **Data Privacy Breach**: Enforce RBAC, audit logs, and penetration testing before launch.
- **Delayed Integrations**: Identify backup providers (e.g., backup email vendor) and mock services for development.
- **Mobile Adoption**: Incorporate UX research, onboarding optimization, and engagement notifications early.
- **Infrastructure Limits**: Load-test single-server design; plan for horizontal scaling roadmap if KPIs exceeded.
- **Regulatory Changes**: Schedule periodic compliance reviews; modularize data export/deletion services.

## 15. Next Steps
1. Validate scope with stakeholders and sign off on requirements.
2. Finalize product backlog with prioritized user stories per phase.
3. Prepare infrastructure accounts (AWS, Firebase, SendGrid) and secrets management.
4. Kick off Phase 1 activities with cross-functional team alignment.
