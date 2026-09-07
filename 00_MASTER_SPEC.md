# NIET Events: Master Project Specification

**Project:** NIET Events

**Institution:** Noida Institute of Engineering and Technology, Greater Noida

**Project type:** College major project

**Status:** Initial architecture and implementation baseline

**Repository:** `NamanIsSharma/NIET-Events-28-`

This document is the single source of truth for the NIET Events project. It records the problem, scope, requirements, architecture, technology responsibilities, data model, workflows, API plan, security rules, development process, and delivery criteria.

## 1. Vision

NIET Events is a centralized platform for discovering, registering for, and safely managing technical, cultural, sports, workshop, and seminar events on campus.

The platform replaces scattered announcements, separate registration forms, paper attendance, manual capacity counting, and delayed certificate creation with one connected system.

## 2. Problem statement

NIET departments, societies, clubs, and the Student Council conduct many events, but students and organizers do not have one shared system.

### Current problems

1. **Awareness gap:** Event information is distributed through WhatsApp, Instagram, notice boards, and word of mouth.
2. **Registration chaos:** Every club may use a separate form, with no unified registration or capacity record.
3. **Attendance difficulty:** Paper registers are slow to reconcile and easy to lose or falsify.
4. **Crowd and safety risk:** Organizers cannot see venue occupancy while an event is running.
5. **Certificate delay:** Certificates are created manually after the event.
6. **Change visibility:** Students may miss venue, schedule, or cancellation updates.

## 3. Objectives

- Provide one searchable feed for all campus events.
- Support event categories, departments, organizers, venues, and schedules.
- Allow students to register with capacity protection.
- Generate a unique QR check-in pass for each registration.
- Let organizers verify attendance in real time.
- Track occupancy for every venue attached to an event.
- Generate attendance certificates automatically.
- Provide role-based dashboards for students, organizers, and administrators.
- Demonstrate enterprise Java concepts through a clean layered implementation.

## 4. Users and roles

### Student

- Register and log in.
- Browse, search, and filter published events.
- View schedule, venue, capacity, and announcements.
- Register or cancel according to event rules.
- View QR passes, attendance, and certificates.

### Organizer

- Create and edit event drafts.
- Add one or more venues with capacities.
- Submit events for approval and publish approved events.
- View registrations and export attendance.
- Check in students using QR tokens.
- Monitor live occupancy and crowd warnings.
- Post event-specific updates where permitted.

### Administrator

- Manage users, roles, departments, and event categories.
- Approve, publish, cancel, or archive events.
- Manage site-wide announcements.
- View system-wide registrations, attendance, occupancy, and reports.
- Review audit events and security-sensitive actions.

## 5. Functional requirements

### FR-01: Authentication and authorization

- Users can log in and log out.
- Passwords are stored only as secure hashes.
- Every authenticated user has one or more controlled permissions through a role.
- Student, organizer, and admin routes are separated.
- Unauthorized users receive a safe access-denied response.

### FR-02: Event management

- Organizers can create drafts with title, description, category, department, dates, and organizer information.
- An event can contain multiple venues.
- Each venue has a positive capacity.
- Events have controlled states: `DRAFT`, `PUBLISHED`, `CANCELLED`, and `COMPLETED`.
- Only published events appear in the student feed.
- Administrators can approve or reject publication.

### FR-03: Event discovery

- Students can see upcoming published events.
- Students can filter by category, department, date, and organizer.
- Students can search by event title and description.
- Events can be ordered by date and registration popularity.
- Active announcements are visible on relevant pages.

### FR-04: Registration

- A student can register for a published event.
- The same student cannot register twice for the same event.
- A cancelled registration cannot be used for check-in.
- Registration is rejected when the selected venue is full.
- Capacity checks and registration insertion occur in one transaction.
- A successful registration receives a unique QR token.

### FR-05: QR check-in

- An organizer can scan or enter a QR token.
- The system validates the token, event, venue, and registration status.
- Unknown, cancelled, or already-used tokens are rejected.
- A valid check-in stores the checker, venue, and timestamp.
- A successful check-in changes registration status to `ATTENDED`.

### FR-06: Certificates

- A certificate is issued only after valid attendance.
- Each certificate has a unique certificate number.
- The certificate records the registration and issue time.
- Students can view or download their certificate.

### FR-07: Live crowd management

- An event can use multiple venues.
- Occupancy is calculated from valid check-ins.
- The dashboard shows capacity, occupied count, percentage, and status.
- The initial UI refreshes through JSON polling every 5 to 10 seconds.
- Status thresholds are configurable and should support `COMFORTABLE`, `BUSY`, and `CRITICAL`.

### FR-08: Notifications and announcements

- The system can send registration confirmation.
- The system can notify users about event changes.
- The system can notify a student when a certificate is available.
- Administrators can publish active site-wide announcements.

### FR-09: Reporting and audit

- Organizers can view registrations and attendance for their events.
- Administrators can view system-wide statistics.
- Important actions are recorded for audit purposes.
- Reports must not expose password hashes or unnecessary personal data.

## 6. Non-functional requirements

- Use a layered modular-monolith architecture.
- Keep controllers free of SQL and complex business rules.
- Use prepared JDBC statements for all user-controlled values.
- Protect capacity-sensitive operations with transactions.
- Validate input at controller and service boundaries.
- Keep credentials outside source control.
- Return safe, user-friendly errors without leaking stack traces.
- Design JSP pages for desktop and mobile browsers.
- Keep API responses stable and documented.
- Add automated tests for business rules and high-risk workflows.

## 7. Technology decisions

| Area | Decision |
|---|---|
| Language | Java 25, as configured in `pom.xml` |
| Application | Spring Boot 3.5 |
| Web layer | Spring MVC plus selected Jakarta Servlet endpoints |
| View layer | JSP, JSTL, Bootstrap 5 |
| Security | Spring Security |
| Persistence | Spring JDBC and handwritten JDBC DAOs |
| Database | Oracle Database or Oracle XE |
| Charts | Chart.js |
| Live refresh | Browser `fetch()` polling initially |
| Build | Maven |
| Packaging | WAR |

The first release is a modular monolith. Microservices, WebSockets, mobile clients, and AI recommendations are future scope, not first-release dependencies.

## 8. Architecture

```text
+-------------------------------------------------------------+
| Browser                                                     |
| JSP / JSTL / Bootstrap / JavaScript / Chart.js              |
+-----------------------------+-------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Web layer                                                   |
| Spring MVC controllers / REST controllers / Servlets        |
+-----------------------------+-------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Application layer                                           |
| Services / validation / authorization / transactions        |
+------------+----------------+----------------+--------------+
             |                |                |
             v                v                v
        QR adapter     Notification       Certificate/PDF
                              |
                              v
+-------------------------------------------------------------+
| Persistence layer                                           |
| JDBC DAOs / PreparedStatement / ResultSet mapping           |
+-----------------------------+-------------------------------+
                              |
                              v
+-------------------------------------------------------------+
| Oracle Database                                             |
+-------------------------------------------------------------+
```

### Layer rules

| Layer | Owns | Must not own |
|---|---|---|
| Controller / Servlet | HTTP input, validation boundary, view or JSON response | SQL or core business decisions |
| Service | Business rules, authorization-sensitive checks, transactions | JSP rendering |
| DAO | SQL, parameters, connection use, result mapping | HTTP, session, role policy |
| Model | Domain state | Database calls |
| DTO | Request and response contracts | Passwords or internal fields unnecessarily |
| View | Presentation and form submission | SQL, capacity, authorization decisions |
| Adapter | External capabilities such as email, QR, PDF | Core event state rules |

## 9. Package and repository structure

```text
NIET Event Management/
├── 00_MASTER_SPEC.md
├── README.md
├── pom.xml
├── .gitignore
├── springboot/README.md
├── jsp/README.md
├── jdbc/README.md
├── security/README.md
├── fullstack/README.md
├── docs/README.md
├── src/main/java/com/niet/events/
│   ├── NietEventsApplication.java
│   ├── config/
│   ├── common/
│   ├── auth/
│   │   ├── controller/ service/ dao/ model/ dto/
│   ├── event/
│   │   ├── controller/ service/ dao/ model/ dto/
│   ├── registration/
│   │   ├── controller/ service/ dao/ model/ dto/
│   ├── crowd/
│   │   ├── controller/ service/ dao/ dto/
│   ├── announcement/
│   ├── certificate/
│   └── notification/
+├── src/main/resources/
│   ├── application.properties
│   ├── db/schema.sql
│   ├── db/seed.sql
│   ├── static/css/
│   ├── static/js/
│   └── WEB-INF/views/
│       ├── auth/ events/ student/ organizer/ admin/ error/
└── src/test/java/com/niet/events/
```

The technology folders at the repository root are documentation and workstream areas. Production Java source remains under `src/main/java`, so the Maven build keeps one coherent application.

## 10. Database specification

The initial schema is in [src/main/resources/db/schema.sql](src/main/resources/db/schema.sql).

### Entities

- `users`: identity, email, password hash, role, department, creation time.
- `events`: organizer, title, description, category, schedule, and status.
- `event_venues`: venue name and capacity for an event.
- `registrations`: student-event relationship, venue, QR token, status, and registration time.
- `check_ins`: registration, checker, actual venue, and check-in time.
- `announcements`: author, message, active period, and creation time.
- `certificates`: registration, certificate number, file location, and issue time.

### Relationships

```text
users 1 ---- * events
users 1 ---- * announcements
events 1 ---- * event_venues
events 1 ---- * registrations
users 1 ---- * registrations
event_venues 1 ---- * registrations
registrations 1 ---- 0..1 check_ins
registrations 1 ---- 0..1 certificates
```

### Database rules

- Email is unique.
- Event end time must be after start time.
- Venue capacity must be greater than zero.
- A student can have one registration per event.
- QR token is unique.
- A registration can have at most one check-in.
- A registration can have at most one certificate.
- Foreign keys must be enforced.
- Index event status/start time, registrations by event, and check-ins by venue.

### Capacity transaction

```text
BEGIN
  lock or re-check the selected venue capacity
  count active registrations/check-ins as defined by the business rule
  reject if capacity is exhausted
  insert registration
COMMIT
```

The exact Oracle locking strategy must be covered by an integration test before the feature is considered complete.

## 11. API and route contract

These routes are the planned initial contract. Implemented routes should preserve the same intent and should be documented when their request or response shape changes.

### Authentication

| Method | Route | Role | Purpose |
|---|---|---|---|
| `GET` | `/login` | Public | Show login page |
| `POST` | `/login` | Public | Authenticate user |
| `POST` | `/logout` | Authenticated | End session |

### Student and event discovery

| Method | Route | Role | Purpose |
|---|---|---|---|
| `GET` | `/events` | Public/Auth | Browse published events |
| `GET` | `/events/{eventId}` | Public/Auth | View event details |
| `POST` | `/events/{eventId}/register` | Student | Register for an event |
| `POST` | `/registrations/{id}/cancel` | Student | Cancel own registration |
| `GET` | `/student/registrations` | Student | View registrations and QR passes |
| `GET` | `/student/certificates` | Student | View issued certificates |

### Organizer

| Method | Route | Role | Purpose |
|---|---|---|---|
| `GET` | `/organizer/events` | Organizer | List owned events |
| `GET` | `/organizer/events/new` | Organizer | Show event form |
| `POST` | `/organizer/events` | Organizer | Create event draft |
| `POST` | `/organizer/events/{id}/publish` | Organizer/Admin | Submit or publish event |
| `POST` | `/organizer/check-in` | Organizer | Verify QR token |
| `GET` | `/organizer/events/{id}/registrations` | Organizer | View event registrations |

### Admin

| Method | Route | Role | Purpose |
|---|---|---|---|
| `GET` | `/admin/dashboard` | Admin | System overview |
| `POST` | `/admin/events/{id}/approve` | Admin | Approve publication |
| `POST` | `/admin/announcements` | Admin | Publish announcement |
| `GET` | `/admin/reports` | Admin | View reports and analytics |

### JSON endpoints

| Method | Route | Role | Response |
|---|---|---|---|
| `GET` | `/api/crowd/events/{eventId}` | Organizer/Admin | Occupancy for each venue |
| `GET` | `/api/analytics/events/{eventId}` | Organizer/Admin | Registration and attendance metrics |
| `GET` | `/api/announcements/active` | Authenticated | Active announcement data |

Example crowd response:

```json
{
  "eventId": 42,
  "venues": [
    {
      "venueId": 7,
      "venueName": "Main Auditorium",
      "capacity": 500,
      "occupied": 210,
      "percentage": 42,
      "status": "COMFORTABLE"
    }
  ],
  "updatedAt": "2026-09-08T12:00:00Z"
}
```

## 12. Key workflows

### Publish an event

```text
Organizer logs in
  -> creates draft
  -> adds venue and capacity
  -> submits for approval
  -> admin reviews
  -> event becomes PUBLISHED
  -> event appears in student feed
```

### Register for an event

```text
Student opens published event
  -> selects available venue if required
  -> service validates role, schedule, duplicate, and capacity
  -> DAO inserts registration in a transaction
  -> QR token is generated
  -> confirmation is shown and optionally emailed
```

### Check in and issue certificate

```text
Organizer scans QR token
  -> token and registration are validated
  -> registration is marked ATTENDED
  -> check-in record is inserted
  -> certificate record is generated
  -> student can download certificate
```

### Monitor crowd

```text
Check-in is recorded
  -> crowd service counts occupancy by venue
  -> organizer dashboard polls JSON endpoint
  -> gauge and status update
  -> critical venue can trigger organizer action
```

## 13. Security specification

- Use Spring Security for authentication and route protection.
- Use a strong password encoder; never store plaintext passwords.
- Use prepared statements for every SQL query.
- Validate all path variables, form fields, dates, capacities, and uploaded files.
- Enforce ownership checks: organizers may manage only their own events unless an admin acts.
- Keep CSRF protection enabled for browser state-changing requests.
- Treat QR tokens as limited-use credentials.
- Check event and venue ownership during check-in.
- Return generic authentication and database error messages.
- Store credentials only in environment variables or ignored local configuration.
- Avoid logging passwords, tokens, or sensitive student information.
- Record audit events for publication, cancellation, check-in, role changes, and certificate issuance.

## 14. Environment and setup

### Required tools

- JDK 25
- Maven 3.9+
- Oracle Database or Oracle XE
- SQL Developer, SQL*Plus, or another Oracle client
- Git

### Environment variables

```powershell
$env:NIET_DB_URL = "jdbc:oracle:thin:@localhost:1521/XEPDB1"
$env:NIET_DB_USERNAME = "niet_app"
$env:NIET_DB_PASSWORD = "your-local-password"
```

Never commit real values. The application defaults are in `src/main/resources/application.properties` and are intended for local development only.

### Commands

```powershell
mvn clean test
mvn spring-boot:run
mvn clean package
```

The application is expected at `http://localhost:8080/`.

## 15. Development branches

```text
main
├── feature/springboot/*       Bootstrapping, MVC, REST, configuration
├── feature/jsp-servlets/*     JSP, JSTL, Servlets, Bootstrap, browser flows
├── feature/jdbc-oracle/*      Oracle schema, SQL, DAOs, transactions
├── feature/security/*         Login, roles, validation, audit controls
└── feature/fullstack/*        Cross-workstream end-to-end features
```

### Branch rules

- Keep `main` stable.
- Create a focused branch from the relevant workstream branch.
- Do not put SQL in controllers or business rules in JSP files.
- Add or update tests with behavior changes.
- Merge through pull requests after review and validation.
- Integrate cross-cutting work on `feature/fullstack` before merging to `main`.

### Commit examples

```text
feat: add organizer event creation
feat: enforce venue capacity transaction
fix: reject duplicate QR check-in
docs: update Oracle setup
test: cover registration service
```

## 16. Implementation milestones

### Milestone 0: Foundation

- Maven project and Spring Boot entry point.
- Oracle schema and local seed data.
- Datasource connection.
- Health check and first DAO query.

### Milestone 1: Authentication

- Login and logout.
- Password hashing.
- Role-based route protection.
- Student, organizer, and admin landing pages.

### Milestone 2: Event discovery and management

- Organizer CRUD.
- Venue management.
- Approval and publication status.
- Student feed, filters, search, and details.

### Milestone 3: Registration

- Registration service and DAO.
- Duplicate and capacity protection.
- QR token creation.
- Student registration history.

### Milestone 4: Attendance and certificates

- QR check-in.
- Attendance state transition.
- Certificate generation.
- Student certificate page.

### Milestone 5: Crowd operations

- Occupancy service.
- Crowd JSON endpoints.
- Chart.js dashboard.
- Threshold alerts and announcements.

### Milestone 6: Quality and delivery

- Unit, DAO, controller, security, and integration tests.
- Audit logging and reports.
- Documentation and demo data.
- Packaging and deployment instructions.

## 17. Testing and acceptance criteria

### Unit tests

- Capacity and duplicate registration rules.
- Event status transitions.
- Role and ownership checks.
- QR token validation.
- Crowd percentage and status calculation.

### Integration tests

- Login to role dashboard.
- Organizer creates and publishes an event.
- Student registers successfully.
- Full venue rejects a new registration.
- Organizer checks in a valid QR token.
- Duplicate check-in is rejected.
- Attendance creates a certificate.
- Crowd endpoint reflects check-ins.

### Security tests

- Anonymous user cannot access protected dashboards.
- Student cannot access organizer actions.
- Organizer cannot modify another organizer's event.
- CSRF-protected state-changing requests reject invalid tokens.
- SQL injection input does not change query behavior.
- Invalid QR tokens reveal no private information.

### Demo acceptance flow

A successful demonstration should show:

1. Admin logs in.
2. Organizer creates an event with two venues.
3. Admin approves it.
4. Student discovers and registers.
5. The student receives a QR pass.
6. Organizer checks in the student.
7. Venue occupancy updates.
8. A certificate becomes available.
9. Admin sees the event and attendance report.

## 18. Future scope

- Native Android and iOS applications.
- Push notifications.
- AI-based event recommendations.
- RFID or college ID-card check-in.
- Sponsor and budget management.
- Alumni events and access.
- WebSocket live updates.
- Docker, CI/CD, and cloud deployment.

## 19. Ownership

Suggested project ownership:

- **Backend and database:** Spring Boot, service logic, JDBC DAOs, Oracle schema, transactions, and reports.
- **Frontend and UI/UX:** JSP, JSTL, Bootstrap, forms, dashboards, accessibility, and responsive layouts.
- **System design and security:** Architecture, Spring Security, validation, testing, documentation, and deployment.

Ownership can be reassigned, but every feature should have one primary owner and one reviewer.

## 20. Definition of done

A feature is complete when:

- Its requirements and route/data contracts are documented.
- Its controller, service, DAO, and view responsibilities are separated.
- Inputs and authorization are validated.
- Transactions are correct for state changes.
- Automated tests cover normal and failure paths.
- No credentials or sensitive data are committed.
- The feature works through the relevant JSP or JSON endpoint.
- The relevant workstream branch is reviewed and integrated.

## 21. Academic deliverables

The final project package should include:

- Problem statement and objectives.
- Requirements and user roles.
- System architecture diagram.
- Use-case and workflow diagrams.
- Database ER diagram and schema explanation.
- API and screen documentation.
- Security and testing strategy.
- Screenshots or live demonstration.
- Installation and user guide.
- Future scope and limitations.
