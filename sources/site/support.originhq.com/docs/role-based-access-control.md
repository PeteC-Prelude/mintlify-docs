# Source: https://support.originhq.com/docs/role-based-access-control

# 

Role Based Access Control

Origin uses two built-in roles to control what users can do in the console: **Admin** and **Member**. Each user in a tenant is assigned exactly one role.

---

## 

Roles at a glance

| Role | Summary |
| --- | --- |
| **Admin** | Full administrative control over the tenant — manages users, agent provisioning, integrations, and tenant configuration. |
| **Member** | Read-only investigator — can view all telemetry and endpoint state, but cannot change configuration or manage users. |

---

## 

Admin

Admins have full control over the tenant. They can:

- **Manage users** — invite new users, remove users, and change a user's role.
- **Manage provisioning** — create and revoke agent provisioning tokens.
- **Configure integrations** — set up and modify the External LLM connection.
- **Manage the Proxy Root CA** — generate or rotate the certificate used by the agent's local proxy.
- **Operate ILO** — trigger In-Line Observer model rebuilds.
- **Edit tenant configuration** — change tenant-wide endpoint settings, including proxy behavior and telemetry settings.

In addition, Admins inherit every capability granted to Members.

---

## 

Member

Members are read-only investigators. They have full visibility into Origin's telemetry but cannot make changes to the tenant.

Members **can**:

- View all endpoints and endpoint metadata.
- Query events.
- Browse prompt sessions and AI activity.
- See telemetry status for each endpoint.
- List installers and download the agent.
- View the current Proxy Root CA.

Members **cannot**:

- Create or revoke provisioning tokens.
- Generate or rotate the Proxy Root CA.
- Change any tenant settings (External LLM connection, proxy behavior, telemetry, ILO).
- Invite, remove, or re-role users.

---

## 

Capability matrix

| Capability | Admin | Member |
| --- | --- | --- |
| View endpoints, events, prompt sessions, and telemetry | ✓ | ✓ |
| List installers and download the agent | ✓ | ✓ |
| View the current Proxy Root CA | ✓ | ✓ |
| Invite, remove, and re-role users | ✓ | — |
| Create and revoke provisioning tokens | ✓ | — |
| Configure the External LLM connection | ✓ | — |
| Generate or rotate the Proxy Root CA | ✓ | — |
| Trigger ILO model rebuilds | ✓ | — |
| Edit tenant endpoint configuration (proxy, telemetry) | ✓ | — |

---

 

Copy Page