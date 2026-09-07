# JDBC and Oracle

This folder represents the persistence workstream for NIET Events.

## Responsibilities

- Oracle schema and seed data.
- DAO interfaces and JDBC implementations.
- Prepared SQL statements and `ResultSet` mapping.
- Connection pooling through the Spring datasource.
- Transactional capacity checks and attendance updates.
- Database indexes, constraints, and reporting queries.

## Main database domains

```text
users -> events -> event_venues
users -> registrations -> check_ins
registrations -> certificates
users -> announcements
```

Never concatenate user input into SQL. Every DAO query must use parameters, and capacity-sensitive operations must be protected by a service transaction.

## Branch

Develop database and DAO work on `feature/jdbc-oracle/*`, then integrate through `feature/fullstack`.
