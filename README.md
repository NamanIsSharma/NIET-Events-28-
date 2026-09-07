# NIET Events

NIET Events is a centralized college event management platform for the Noida Institute of Engineering and Technology. It gives students one place to discover and register for campus events while giving organizers and administrators tools to publish events, manage capacity, verify attendance, monitor venue occupancy, and issue certificates.

This is a Java major project designed to demonstrate a complete full-stack system using Spring Boot, Spring MVC, JSP, Servlets, Spring Security, raw JDBC, and Oracle Database.

## Problem statement

College events are often announced through separate WhatsApp groups, social media posts, notice boards, and forms. This creates three practical problems:

- Students miss events, updates, and venue changes.
- Organizers do not have one reliable registration or attendance record.
- Organizers cannot see venue occupancy early enough to manage crowd and safety risks.

NIET Events addresses these issues through an awareness hub, smart registration and certificates, and a live crowd management dashboard.

## Core features

### Awareness hub

- Browse all published events from one feed.
- Filter events by technical, cultural, sports, workshop, or seminar category.
- Search by title, department, date, or organizer.
- Surface popular events using registration counts.
- Display active announcements and important event updates.

### Registration and attendance

- Register for an event through a single platform.
- Show available capacity before registration.
- Prevent duplicate registrations and overbooking.
- Generate a unique QR token for every registration.
- Allow organizers to check in students by scanning or entering the token.
- Record attendance with time, venue, and organizer information.

### Certificates and notifications

- Generate a certificate after valid attendance is confirmed.
- Store a unique certificate number for verification.
- Notify students about registration, approval, changes, and certificate availability.

### Live crowd management

- Support multiple venues for one event.
- Calculate occupancy from confirmed check-ins.
- Display occupied seats, capacity, percentage, and crowd status.
- Expose live JSON endpoints for dashboard polling.
- Mark venues as comfortable, busy, or critical based on configurable thresholds.

### Administration

- Manage users, roles, departments, events, venues, and announcements.
- Review organizer-created events before publication.
- View registration, attendance, and occupancy reports.
- Keep an audit trail for sensitive actions.

## Technology stack

| Area | Technology | Purpose |
|---|---|---|
| Language | Java 25 | Application development |
| Backend | Spring Boot 3.5 | Application bootstrapping and dependency management |
| Web MVC | Spring MVC | Controllers, routing, and request handling |
| Views | JSP, JSTL, Bootstrap 5 | Server-rendered web pages |
| Legacy web API | Jakarta Servlets | Raw servlet endpoints where required for demonstration |
| Security | Spring Security | Authentication, authorization, roles, and CSRF protection |
| Persistence | Spring JDBC and raw JDBC | Prepared SQL and DAO-based database access |
| Database | Oracle Database or Oracle XE | Relational data storage |
| Charts | Chart.js | Organizer and administrator analytics |
| Live data | `fetch()` polling | Refresh crowd data without a full page reload |
| Build | Maven | Dependency management, compilation, testing, and packaging |

## Architecture

```text
Student / Organizer / Admin browser
        |
        | JSP pages, forms, and fetch() JSON requests
        v
Presentation layer
  Spring MVC Controllers and Servlets
        |
        v
Application layer
  Services, validation, transactions, authorization checks
        |
        +--> Notification adapter
        +--> QR adapter
        +--> Certificate/PDF adapter
        |
        v
Persistence layer
  JDBC DAOs, PreparedStatements, ResultSet mapping
        |
        v
Oracle Database
```

The application is intentionally a modular monolith for the first version. All domains run in one Spring Boot application, but each domain keeps its own controller, service, DAO, model, and DTO classes. This provides clear boundaries without the deployment and networking complexity of microservices.

## Project structure

```text
NIET Event Management/
├── pom.xml
├── README.md
├── .gitignore
├── src/
│   ├── main/
│   │   ├── java/com/niet/events/
│   │   │   ├── NietEventsApplication.java
│   │   │   ├── config/                 Security, MVC, and datasource configuration
│   │   │   ├── auth/                   Login, users, and roles
│   │   │   │   ├── controller/
│   │   │   │   ├── service/
│   │   │   │   ├── dao/
│   │   │   │   ├── model/
│   │   │   │   └── dto/
│   │   │   ├── event/                  Event and venue management
│   │   │   ├── registration/           Registration, QR, and check-in
│   │   │   ├── crowd/                  Occupancy calculations and JSON APIs
│   │   │   ├── announcement/            Site-wide announcements
│   │   │   ├── notification/            Email notification adapters
│   │   │   ├── certificate/             PDF certificate generation
│   │   │   └── common/                  Exceptions and shared validation
│   │   └── resources/
│   │       ├── application.properties
│   │       ├── db/schema.sql
│   │       ├── db/seed.sql
│   │       ├── static/css/
│   │       ├── static/js/
│   │       └── WEB-INF/views/           JSP pages
│   │           ├── auth/
│   │           ├── events/
│   │           ├── student/
│   │           ├── organizer/
│   │           ├── admin/
│   │           └── error/
│   └── test/java/com/niet/events/       Unit and integration tests
└── target/                              Generated Maven output, ignored by Git
```

## Domain model

The initial Oracle schema is available in [src/main/resources/db/schema.sql](src/main/resources/db/schema.sql).

```text
users
  |
  +--> events ------------------> event_venues
  |       |
  |       +---------------------- registrations
  |                                  |
  |                                  +--> check_ins
  |                                  +--> certificates
  |
  +--> announcements
```

Main tables:

- `users`: students, organizers, and administrators.
- `events`: title, description, category, schedule, organizer, and publication status.
- `event_venues`: one or more venues and capacity for each event.
- `registrations`: student-event relationship, selected venue, QR token, and status.
- `check_ins`: attendance time, checking organizer, and actual venue.
- `announcements`: notices displayed across the application.
- `certificates`: certificate number, issue time, and generated file location.

## Layer responsibilities

| Layer | Responsibility | Rule |
|---|---|---|
| Controller / Servlet | Read HTTP input and return JSP or JSON | No SQL and no complex business rules |
| Service | Implement business rules and transactions | Must enforce authorization-sensitive actions |
| DAO | Execute prepared SQL and map results | No HTTP or JSP dependencies |
| Model | Represent domain data | No database calls |
| DTO | Define request and response shapes | Do not expose unnecessary database fields |
| Configuration | Configure security, MVC, datasource, and JSP | No feature-specific logic |
| JSP view | Render data and submit user actions | No capacity decisions or authorization logic |

## Important request flows

### Student registration

```text
POST /events/{eventId}/register
  -> Event/RegistrationController
  -> RegistrationService.register(studentId, eventId)
  -> verify event is published and registration is not duplicated
  -> check venue capacity inside a transaction
  -> RegistrationDao.insert(...)
  -> generate QR token
  -> send confirmation notification
  -> redirect to registration confirmation page
```

Capacity must be checked transactionally. A simple count followed by an insert is not enough because two simultaneous requests could both observe one remaining seat. The DAO/service must lock or re-check the relevant venue row before committing the registration.

### QR check-in

```text
POST /organizer/check-in
  -> CheckInController or CheckInServlet
  -> CheckInService.checkIn(qrToken, organizerId)
  -> find registration and verify event/venue relationship
  -> reject cancelled, unknown, or already-attended registration
  -> insert check-in and mark registration ATTENDED
  -> create certificate record
```

### Live crowd monitor

```text
GET /api/crowd/events/{eventId}
  -> CrowdRestController
  -> CrowdService.getOccupancy(eventId)
  -> CheckInDao.countByVenue(eventId)
  -> JSON response for Chart.js and dashboard gauges
```

The dashboard can poll this endpoint every 5 to 10 seconds for the first version. WebSockets or push notifications can be added later if polling is no longer sufficient.

## Local setup

### Prerequisites

- JDK 25
- Maven 3.9 or newer
- Oracle Database, Oracle XE, or an accessible Oracle development instance
- An Oracle schema user with permission to create the application tables

### Configure the database

The default development URL is:

```text
jdbc:oracle:thin:@localhost:1521/XEPDB1
```

Set credentials with environment variables rather than committing passwords:

```powershell
$env:NIET_DB_URL = "jdbc:oracle:thin:@localhost:1521/XEPDB1"
$env:NIET_DB_USERNAME = "niet_app"
$env:NIET_DB_PASSWORD = "your-local-password"
```

Run [schema.sql](src/main/resources/db/schema.sql) using Oracle SQL Developer, SQL*Plus, or another Oracle SQL client. Add seed data only to a local seed script; never commit real student data or credentials.

### Build and run

From the project root:

```powershell
mvn clean test
mvn spring-boot:run
```

Open `http://localhost:8080/` after the application starts. To package the WAR file:

```powershell
mvn clean package
```

## Implementation roadmap

1. Create the Oracle schema and local seed data.
2. Add datasource configuration and verify a first JDBC query.
3. Implement login, logout, password hashing, and role-based routes.
4. Build public event listing, search, category filters, and event details.
5. Build organizer event and venue CRUD with draft/published status.
6. Implement registration, duplicate protection, and capacity enforcement.
7. Generate QR tokens and implement organizer check-in.
8. Generate attendance certificates and notification messages.
9. Add crowd APIs and the organizer live dashboard.
10. Add announcements, reports, analytics, audit logging, and automated tests.

The first release should complete one vertical slice: login, event creation, event browsing, registration, QR check-in, and attendance. This creates a demonstrable working system before advanced features are added.

## Security baseline

- Hash passwords with Spring Security's password encoder.
- Use `PreparedStatement` parameters for every user-controlled SQL value.
- Validate IDs, dates, capacities, categories, and file inputs at the web and service boundaries.
- Enforce permissions in services as well as URL configuration.
- Protect state-changing forms with CSRF protection.
- Never trust a QR token without checking its registration, event, and venue relationship.
- Do not expose password hashes, internal database errors, or private student data in JSP or JSON responses.
- Store credentials in environment variables or ignored local configuration.
- Add audit records for event publication, registration cancellation, check-in, and certificate issuance.

## Testing strategy

- Unit tests for capacity checks, role rules, status transitions, and QR validation.
- DAO integration tests against an Oracle test schema when available.
- Controller tests for form validation, redirects, error messages, and JSON responses.
- Integration tests for the complete registration-to-certificate workflow.
- Security tests for unauthenticated access, role boundaries, CSRF, duplicate registration, and invalid QR tokens.
- Manual acceptance flow for student, organizer, and administrator dashboards.

## Git workflow

The repository uses technology workstreams, inspired by the Honey Chain project:

```text
main
├── feature/springboot/*       Spring Boot, MVC, REST, and configuration
├── feature/jsp-servlets/*     JSP, JSTL, Servlets, Bootstrap, and browser flows
├── feature/jdbc-oracle/*      JDBC DAOs, SQL, Oracle schema, and transactions
├── feature/security/*         Spring Security, roles, validation, and audit rules
└── feature/fullstack/*        End-to-end integration of all workstreams
```

Branches are collaboration areas, not separate applications. Each feature branch should be merged into `main` through a pull request after its tests pass. Use `feature/fullstack` for integrated features such as registration, QR check-in, and live crowd monitoring.

Suggested commit format:

```text
feat: add event registration service
fix: prevent venue overbooking
docs: update Oracle setup
test: cover QR check-in flow
refactor: separate crowd calculation service
```

## Future scope

- Native Android and iOS companion application.
- Push notifications for urgent venue and schedule changes.
- Personalized event recommendations.
- RFID or college ID-card check-in.
- Sponsor and event budget management.
- Alumni access for reunion and networking events.
- WebSocket-based crowd updates.
- Deployment with Docker and a CI pipeline.

## Team responsibilities

The work can be divided into three major areas:

- Backend and database: Spring Boot, service logic, JDBC DAOs, Oracle schema, transactions, and reports.
- Frontend and UI/UX: JSP, JSTL, Bootstrap, forms, dashboards, accessibility, and responsive layouts.
- System design and security: architecture, Spring Security, validation, testing, documentation, and deployment.

## License

This project is an academic major project for NIET. Add the final license and contribution policy before accepting external contributions.
