# Security

This folder represents authentication, authorization, validation, and audit work.

## Responsibilities

- Spring Security login and logout.
- `STUDENT`, `ORGANIZER`, and `ADMIN` roles.
- Password hashing with a password encoder.
- URL and service-layer authorization.
- CSRF protection for state-changing forms.
- Input validation and safe error handling.
- Audit records for publishing events, check-ins, and certificate issuance.

QR tokens must be treated as credentials with limited purpose. A check-in must verify that the token belongs to the requested event and that it has not already been used.

## Branch

Develop security work on `feature/security/*`, then integrate through `feature/fullstack`.
