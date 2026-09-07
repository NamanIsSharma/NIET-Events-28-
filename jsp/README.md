# JSP and Servlets

This folder represents the presentation and servlet workstream for NIET Events.

## Responsibilities

- JSP pages with JSTL for server-rendered screens.
- Bootstrap-based responsive layouts.
- Student, organizer, and administrator dashboards.
- HTML forms, validation messages, and navigation.
- Servlet endpoints for QR check-in and other explicitly servlet-based flows.
- JavaScript `fetch()` calls for live crowd JSON data.

## Planned view folders

```text
src/main/webapp/WEB-INF/views/
  auth/
  events/
  student/
  organizer/
  admin/
  error/
```

Views display data only. SQL, authorization decisions, and capacity rules belong in Java services and DAOs.

## Branch

Develop presentation work on `feature/jsp-servlets/*`, then integrate through `feature/fullstack`.
