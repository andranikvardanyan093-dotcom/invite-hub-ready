# InviteHub White-Label Multi-Tenant Architecture

## Overview

InviteHub is a multi-organization SaaS. Every user receives a **personal organization** on signup (and existing users are backfilled by migration `012`). All tenant data is scoped by `organization_id` with Postgres RLS.

## Apply migrations

After `012_enterprise_platform.sql`:

1. `database/migrations/013_white_label_tenancy.sql` — schema, helpers, backfill
2. `database/migrations/014_white_label_rls.sql` — RLS + RPCs

Reuses enterprise `audit_logs`, `api_keys`, and `webhook_endpoints` (org FKs + vault `secret_ref`).

## Backfill behavior

`provision_personal_organization(user_id)`:

- Creates `organizations` row (`is_personal = true`)
- Adds owner membership, branding, settings, SMTP/SMS stubs, usage limits, subscription
- Creates default workspace
- Attaches existing `invitations`, `payments`, `subscriptions`, email/SMS/activity logs

New signups call the same function from `handle_new_user()`.

## RLS approach

| Helper | Purpose |
|---|---|
| `current_org_ids()` | Active orgs for `auth.uid()` |
| `is_org_member(org_id)` | Membership check (+ super admin) |
| `has_org_permission(org_id, permission)` | Role matrix (owner → viewer) |
| `can_access_invitation(id)` | Owner, org member, or client portal |
| `owns_invitation(id)` | Owner **or** org `invitations.edit` |

No tenant can read another tenant’s rows unless they are a platform super admin.

## Custom domains & SSL (infra)

**App-controllable (implemented):**

- Store domains + verification tokens
- Auto-generated DNS instructions (TXT + CNAME)
- Subdomain field on organizations
- Resolve branding via `resolve_organization_by_host` / `resolve_organization_by_slug`
- UI under Organization → Domain

**Platform / infrastructure (not in app DB):**

1. Edge/DNS worker verifies TXT `invitehub-verify=<token>`
2. Provision TLS certificates (e.g. Caddy / Traefik / Cloudflare / Supabase custom domains)
3. Route hostname → frontend with org resolution
4. CNAME target currently documented as `edges.invitehub.am` — update for your edge

SSL status column (`ssl_status`) is informational; certificate issuance is infra-owned.

## Secrets pattern

SMTP passwords, SMS API keys, webhook signing secrets, and OAuth tokens are **never** stored plaintext in Postgres or returned to the browser.

- Store only `*_secret_ref` vault/Edge secret names
- Edge Functions resolve secrets at send/connect time
- Client services strip `password` / `api_key` fields before upsert

## App routes

| Route | Purpose |
|---|---|
| `/app/organizations` | List / create orgs |
| `/app/organization` | Org settings (branding, domain, SMTP, SMS, team, billing, API, webhooks, integrations, audit) |
| `/app/client-portal` | Client review / approve / comment |
| `/o/:orgSlug/login\|register\|forgot-password` | Branded auth |
| `/invite-team/:token` | Accept team invite |
| `/admin/organizations` | Super-admin tenant management |

## Storage paths

Prefer `org/{organization_id}/...` (see `org_storage_prefix()` and `organization_files` quota table).
