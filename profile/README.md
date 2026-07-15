# Aegis Identity Platform

**Aegis** is a multi-tenant, cloud-native Identity & Access Management (IAM) platform — an Okta-style identity cloud built as distributed Spring Boot microservices behind a single edge gateway, with a React admin console and infrastructure-as-code for local Docker, AWS and Azure.

## What it does

- **Per-tenant OAuth2 / OIDC issuers** — each organisation gets its own issuer (`https://<gateway>/{tenant}`) with its **own RSA signing key**, served through the edge gateway. Resource servers validate against an aggregate JWKS with a strict issuer allowlist.
- **Hosted, branded login** — password (Argon2id, lockout) with enforced MFA step-up: TOTP and WebAuthn/passkeys. Tokens cannot be minted while a second factor is pending.
- **Embedded customer auth** — passkey login inside a tenant's own web/mobile app via a PKCE-bound, single-use interaction-code exchange.
- **Enterprise federation** — SAML 2.0 IdP, social/external IdP brokering (OIDC), and SCIM 2.0 provisioning.
- **Machine-to-machine** — `client_credentials` with tenant-scoped clients and per-endpoint scope enforcement.
- **Zero-trust by construction** — every service validates JWTs itself (never the network), default-deny network policies, least-privilege workload identities, per-tenant cryptographic isolation.

## Repositories

| Repository | What it is |
|---|---|
| [aegis-authorization-server](https://github.com/AegisIdentity/aegis-authorization-server) | Spring Authorization Server — per-tenant issuers & signing keys, hosted login, MFA gate |
| [aegis-edge-gateway](https://github.com/AegisIdentity/aegis-edge-gateway) | Spring Cloud Gateway — the single public front door and per-tenant issuer routing |
| [aegis-identity-service](https://github.com/AegisIdentity/aegis-identity-service) | Users, credentials (Argon2id), lockout and authentication policy |
| [aegis-tenant-service](https://github.com/AegisIdentity/aegis-tenant-service) | Organisations/tenants, custom sign-in domains |
| [aegis-mfa-webauthn-service](https://github.com/AegisIdentity/aegis-mfa-webauthn-service) | TOTP and WebAuthn/passkey ceremonies (per-tenant relying parties) |
| [aegis-saml-idp-service](https://github.com/AegisIdentity/aegis-saml-idp-service) | SAML 2.0 identity provider |
| [aegis-social-broker-service](https://github.com/AegisIdentity/aegis-social-broker-service) | Social / external IdP brokering |
| [aegis-scim-provisioning-service](https://github.com/AegisIdentity/aegis-scim-provisioning-service) | SCIM 2.0 user provisioning |
| [aegis-admin-api-service](https://github.com/AegisIdentity/aegis-admin-api-service) | Administration & management APIs |
| [aegis-admin-console](https://github.com/AegisIdentity/aegis-admin-console) | React 19 + TypeScript admin console (tenant-customised docs & API explorer) |
| [aegis-platform-commons](https://github.com/AegisIdentity/aegis-platform-commons) | Shared library (security, events, web) |
| [aegis-platform-bom](https://github.com/AegisIdentity/aegis-platform-bom) | Maven BOM — one place for dependency versions |
| [aegis-platform-infra](https://github.com/AegisIdentity/aegis-platform-infra) | Docker Compose stack, Terraform (AWS + Azure, dev/test/stage/prod), Helm |
| [aegis-platform-docs](https://github.com/AegisIdentity/aegis-platform-docs) | Architecture, ADRs, and diagrams (app, sequence flows, infra for Docker/AWS/Azure) |

## Architecture at a glance

- **Stack**: Java 21 · Spring Boot · Spring Security / Spring Authorization Server · PostgreSQL (database-per-service) · Redis · Kafka · React 19 + TypeScript + Vite
- **Topology**: one public entry point (edge gateway); all services private behind it; events on Kafka; per-service databases — no shared tables, ever
- **Deployment**: Docker Compose locally; EKS/AKS via modular Terraform with two profiles — `cost-optimized` (dev/test) and `hardened` (stage/prod: private control planes, CMKs everywhere, private endpoints, least-privilege pod identities)

Diagrams and ADRs live in [aegis-platform-docs](https://github.com/AegisIdentity/aegis-platform-docs).

## Run it locally

```sh
git clone https://github.com/AegisIdentity/aegis-platform-infra
cd aegis-platform-infra/compose
docker compose up -d
```

- Admin console: http://localhost:3000
- Edge gateway (public API + per-tenant issuers): http://localhost:8080
- Authorization server (direct, dev only): http://localhost:9000

All credentials in the compose stack are clearly marked `dev-only` defaults for local development — replace everything before any real deployment.

## Status

Under active development — built spec-first with TDD across the services. Review the security posture and run your own assessment before production use.
