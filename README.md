# CyberCase — Complaint & Investigation Management

A Java 21 / Spring Boot / Thymeleaf application for authenticated complaint intake, search, case workflow, investigation notes, audit activity, CSV reporting, and optional URLhaus domain checks. It starts with **no complaint fixtures or seeded fake data**.

## Requirements
- Java 21+
- Maven 3.9+
- H2 for local evaluation (default) or PostgreSQL for deployment

## Run locally
```bash
# From this directory
# Windows PowerShell — local demonstration accounts only
$env:SPRING_PROFILES_ACTIVE = "dev"
mvn spring-boot:run
```
Open **http://localhost:8080/login**. All three staff roles use this same login page; after signing in they are redirected to role-specific dashboard URLs within the same application. The `dev` profile provisions the following accounts only if each username does not already exist:

| Role | Username | Development password |
|---|---|---|
| Administrator | `admin.demo` | `AdminDemo!2026Secure` |
| Investigator staff | `staff.demo` | `StaffDemo!2026Secure` |
| Reviewer (third role) | `user.demo` | `UserDemo!2026Secure` |

These credentials are for local demonstration only. Change them before any shared or deployed use, and never activate the `dev` profile in production. Bootstrap does not overwrite existing accounts or reset an existing password. If you already have a database with these usernames, the existing credentials remain in effect. To provision fresh accounts outside the demo profile, set the matching `app.bootstrap.*` values through environment/configuration with unique passwords of at least 14 characters.

The default H2 database is file-backed under `./data/cybercase`. Flyway creates the schema. It is intentionally empty. Do not expose the H2 console in a deployed environment.

## PostgreSQL configuration
Create an empty database and configure:
```bash
export DATABASE_URL='jdbc:postgresql://localhost:5432/cybercase'
export DATABASE_USERNAME='cybercase_app'
export DATABASE_PASSWORD='replace-with-a-secret'
export DATABASE_DRIVER='org.postgresql.Driver'
```
Use a database account with only the privileges needed by the application. Back up the database and test recovery. For production, terminate TLS at a trusted reverse proxy, set secure cookie flags, restrict network access, manage secrets with a secret manager, and apply institutional retention/identity policies.

## Optional live threat-intelligence check
The Verification Center supports a domain-only check through the URLhaus API. Configure `URLHAUS_API_KEY` with an authorized API key according to the current URLhaus terms and API documentation: https://urlhaus.abuse.ch/api/ . Without the key, the UI returns `UNCONFIGURED` and explicitly states that no live lookup was performed. A source error returns `UNAVAILABLE`, never a successful verification. A match is an investigative lead only.

This does **not** integrate with the National Cyber Crime Reporting Portal or access individual police complaints. A public web page is not automatically a public API; the official portal's access and any case-level records require documented authorization. Do not scrape restricted systems or bypass controls. Add another adapter only after confirming the official interface, terms, data minimization rules, and authorization.

## Current implemented features
- One shared `/login` page for administrator, investigator, and reviewer roles; BCrypt password hashing, CSRF protection, session logout, profile/environment-based bootstrap provisioning.
- Case Constellation / Connection Intelligence page at `/intelligence`, grouping repeated complaint indicators as potential leads for human review. The browser-only analysis does not label a connection as proof of wrongdoing.
- Role-specific dashboard routes: `/admin/dashboard`, `/staff/dashboard`, and `/reviewer/dashboard`; the same `/login` authenticates all three roles.
- Innovation Engine at `/innovation`: data-grounded findings for urgent unassigned cases and repeated indicators, transparent priority heuristics, and an auditable assignment action for authorized staff. Outcome counts are measured snapshots, not causal claims.
- CSV export is restricted to administrators and reviewers.
- Dashboard counts from persisted records, empty states, recent records and audit events.
- Complaint registration with server-generated references, validated category/priority, search, filters, pagination and detail view.
- Status and assignment workflow updates; investigation notes; append-only through normal UI audit events.
- URLhaus verification adapter with explicit unconfigured/unavailable/match/no-match outcomes.
- CSV export with formula-injection mitigation.
- Responsive custom CSS, hover motion, entrance reveals, loading feedback, and reduced-motion support.

## Deliberate limitations — do not treat as production-certified
This is a runnable foundation, not a certified law-enforcement deployment. Before real operational use, implement and independently review: fine-grained role-specific method authorization and officer assignment scope; login throttling/lockout and centralized identity integration; append-only/externally retained audit protection; evidence upload/storage, malware scanning and chain-of-custody controls; case status history as a dedicated entity; tasks and related-incident decision persistence; export filters and permission checks; CSRF-aware rate limiting; robust URLhaus response parsing against its current schema; integration and security tests; backup/retention policy; and deployment hardening. Evidence upload is intentionally not exposed rather than offering unsafe storage. Role-based authorization remains a starter implementation and should be expanded into a complete per-action policy before production deployment. Do not deploy with sensitive real cases until these controls have been implemented and reviewed.

## Build and test
```bash
mvn test
mvn package
```
Tests have not been executed in this environment if Maven is unavailable. Review `src/test` and run the commands locally before treating the build as verified.


## Shared login and development demo accounts

All staff roles use the same URL: `http://localhost:8080/login`. The server determines permissions from the saved account role; the login form does not accept a role choice.

For a local demonstration only, activate the development profile in PowerShell before starting the app:

```powershell
$env:SPRING_PROFILES_ACTIVE = "dev"
mvn spring-boot:run
```

Development demo credentials (only created when the username does not already exist):

- Administrator — `admin.demo` / `AdminDemo!2026Secure`
- Investigator staff — `staff.demo` / `StaffDemo!2026Secure`
- Reviewer (third role) — `user.demo` / `UserDemo!2026Secure`

Passwords are encoded using BCrypt before persistence. The bootstrap process does not reset existing passwords. These credentials are development-only; change them before sharing the application and never enable the `dev` profile in production. The Innovation Engine uses transparent rules over persisted complaint data; its scores are prioritization heuristics, not AI predictions or evidence of guilt.
