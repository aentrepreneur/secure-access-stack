# Secure Access Stack Architecture

## Public Architecture

```text
                        +-------------------------+
                        |    Users / Clients      |
                        |  VPN / Web / Admin UI   |
                        +-----------+-------------+
                                    |
                            HTTPS / RADIUS
                                    |
                        +-----------v-------------+
                        |   Access Surface        |
                        |   Reverse Proxy / WAF   |
                        |   Auth Challenge Entry  |
                        +-----------+-------------+
                                    |
                        +-----------v-------------+
                        |   MFA Service Layer     |
                        |   Token Validation      |
                        |   Enrollment Flows      |
                        |   Policy Evaluation     |
                        +-----------+-------------+
                                    |
                        +-----------v-------------+
                        |   RADIUS / Directory    |
                        |   Network Access Policy |
                        |   LDAP / Identity Res.  |
                        |   Authorization Logic   |
                        +-----------+-------------+
                                    |
                        +-----------v-------------+
                        |   Data and Cache Layer  |
                        |   Persistent Store      |
                        |   Session Management    |
                        |   Policy and Audit Data |
                        +-----------+-------------+
                                    |
                        +-----------v-------------+
                        |   Operations Layer      |
                        |   Validation / Health   |
                        |   Monitoring / Alerts   |
                        |   Audit Trail / Logs    |
                        +-------------------------+
```

## Component Responsibility

| Component | Role | Key Integration |
|-----------|------|-----------------|
| Reverse Proxy | TLS termination, auth challenge entry, surface hardening | WAF rules, certificate management |
| MFA Service | Token validation, enrollment orchestration, policy evaluation | RADIUS, LDAP, token store |
| RADIUS Daemon | Network access policy enforcement, VPN integration | Directory, MFA service |
| LDAP / Identity Provider | Identity resolution, directory-backed auth flows | User directory, cache layer |
| Data Store | Persistent policy, token records, session metadata | Backend database, replicated storage |
| Cache Layer | Lightweight session state, rate-limiting counters | Memory / Redis |
| Operations Module | Deployment validation, health checks, audit collection | All layers |

## Request Flow (Simplified)

```text
Client -> Proxy (TLS, auth challenge)
  -> MFA Service (token validation, policy check)
    -> RADIUS/LDAP (identity resolve, authorization)
      -> Session Grant (token, cookie, or RADIUS accept)
        -> Audit Log (evidence record written)
```

## Execution Model

The stack separates two operational concerns:

- **Control plane**: configuration, policy, identity integration, deployment logic
- **Data plane**: authentication requests, token validation, session management, audit streams

Control plane changes do not interrupt data plane operations. This allows policy updates, identity source changes, and operational maintenance without authentication service disruption.

## Design Rationale

- productization means the architecture must support repeatable deployment, not only functional correctness
- RADIUS and LDAP integration is intentional: most SMB infrastructure already uses directory services and network access policies
- the cache and data separation allows the MFA layer to remain stateless for request processing
- audit and operations are treated as architectural layers, not afterthoughts

## Public Boundary

This document describes system structure and integration logic without exposing sensitive internal implementation details such as exact software versions, certificate paths, or deployment secrets.
