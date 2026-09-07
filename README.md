# NIET Events

A centralized event platform for NIET students, organizers, and administrators.

## First milestone

Build one complete vertical slice before adding every feature:

1. Student/admin login with role-based access.
2. Organizer creates and publishes an event.
3. Student browses an event and registers while capacity is available.
4. Organizer checks in the registration using its QR token.
5. The registration becomes attended and a certificate can be generated.

The crowd dashboard, announcements, email, and analytics should build on this working flow.

## Architecture

```text
Browser
  JSP + JSTL + Bootstrap + Chart.js
       |
       | HTML form requests and fetch() JSON requests
       v
Web layer
  Controller / Servlet -> Service -> DAO -> Oracle
                              |
                              +-> notification, QR, certificate adapters
```

Use Spring MVC controllers for normal pages and small JSON endpoints for live data. Keep raw JDBC in DAO classes; controllers must not contain SQL.

## Package and folder layout

```text
src/main/java/com/niet/events/
  NietEventsApplication.java
  config/              Security, MVC, datasource configuration
  auth/                Login, roles, user session
    controller/ service/ dao/ model/ dto/
  event/               Event and venue management
    controller/ service/ dao/ model/ dto/
  registration/        Capacity checks, QR, check-in, certificates
    controller/ service/ dao/ model/ dto/
  crowd/               Occupancy calculations and JSON endpoints
    controller/ service/ dao/ dto/
  announcement/        Site-wide announcements
    controller/ service/ dao/ model/
  notification/        Email notification port and implementation
  certificate/          PDF certificate generation
  common/               Exceptions, validation, API response helpers

src/main/resources/
  application.properties
  db/schema.sql
  db/seed.sql
  static/css/ static/js/ static/images/
  templates/            JSP files under WEB-INF/views/
    auth/ events/ organizer/ admin/ student/ error/

src/test/java/com/niet/events/
  event/ registration/ crowd/
```

## Layer responsibilities

| Layer | Responsibility | Must not do |
|---|---|---|
| Controller / Servlet | HTTP input, validation boundary, view or JSON response | SQL or business rules |
| Service | capacity, role rules, registration state changes, transactions | JSP concerns |
| DAO | Prepared SQL, mapping `ResultSet` to models | authorization or HTTP |
| Model / DTO | Database/domain data and request/response shapes | database calls |
| Config | Security rules, datasource, MVC/JSP setup | feature logic |
| View | Render data and submit actions | deciding capacity or permissions |

## Database relationships

- `users` has one role: `STUDENT`, `ORGANIZER`, or `ADMIN`.
- `events` belongs to its organizer and has a category and status.
- `event_venues` stores one or more venues and capacity per event.
- `registrations` joins a student to an event and stores a unique QR token and attendance state.
- `check_ins` records who checked in, when, and at which venue.
- `announcements` stores site-wide notices.
- `certificates` stores the generated certificate number and file location.

Capacity must be enforced in a transaction. Lock or re-check the event/venue row before inserting a registration so two simultaneous requests cannot overbook it.

## Request flow examples

### Student registration

```text
POST /events/{eventId}/register
  EventController
    -> RegistrationService.register(studentId, eventId)
      -> EventVenueDao.findAvailableVenueForUpdate(...)
      -> RegistrationDao.insert(...)
      -> NotificationService.sendRegistrationConfirmation(...)
```

### Live crowd monitor

```text
GET /api/crowd/events/{eventId}
  CrowdRestController
    -> CrowdService.getOccupancy(eventId)
      -> CheckInDao.countByVenue(...)
      -> returns { venueName, capacity, occupied, percentage, status }
```

The JSP dashboard can poll that endpoint every 5-10 seconds with `fetch()` until a WebSocket or push-notification solution is justified.

## Local setup

Prerequisites:

- JDK 25
- Maven 3.9+
- Oracle Database or an Oracle XE container
- A database user with permission to create the NIET tables

After installing Java and Maven:

```powershell
mvn clean test
mvn spring-boot:run
```

Open `http://localhost:8080/`.

Do not commit real database passwords. Put local values in an ignored profile such as `application-local.properties`, or use environment variables.

## Suggested implementation order

1. Create the schema and seed one admin, organizer, student, event, and venue.
2. Add datasource properties and verify one DAO query.
3. Implement login and role-based routes.
4. Implement event listing and organizer event CRUD.
5. Implement registration with capacity enforcement.
6. Add QR token generation and check-in.
7. Add attendance and PDF certificates.
8. Add crowd REST endpoints and the organizer dashboard.
9. Add announcements, email, analytics, audit logging, and tests.

## Security baseline

Hash passwords with a password encoder, use prepared statements, validate all IDs and form fields, enforce authorization in services as well as URL rules, protect state-changing forms with CSRF, and never trust a QR token without checking that the registration belongs to the event being checked in.

## Git workflow

The project is organized by technology workstreams, similar to the Honey Chain project:

```text
main
├── feature/springboot/*       Spring Boot, MVC, REST, configuration
├── feature/jsp-servlets/*     JSP, JSTL, Servlets, Bootstrap, browser flows
├── feature/jdbc-oracle/*      JDBC DAOs, SQL, Oracle schema and transactions
├── feature/security/*         Spring Security, roles, validation, audit rules
└── feature/fullstack/*        End-to-end integration of all workstreams
```

Branches are collaboration areas, not separate applications. Each branch should be merged into `main` through a pull request after its tests pass. Use the full-stack branch for integrated features such as registration, QR check-in, and the live crowd dashboard.

Suggested commit format:

```text
feat: add event registration service
fix: prevent venue overbooking
docs: update Oracle setup
test: cover QR check-in flow
```
