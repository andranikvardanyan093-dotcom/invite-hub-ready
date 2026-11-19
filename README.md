# InviteHub

Commercial multi-tenant SaaS for creating digital invitation websites.

**Stack:** React (Vite) · React Router · Supabase Auth/DB/Storage · PostgreSQL · Vercel · Expo (iOS/Android)

## Features

- Multi-tenant accounts with strict data isolation (RLS)
- Invitation builder with templates, themes, SEO, and public URLs (`/invite/:code` or `/:slug`)
- Guests, RSVPs, gift registry, analytics
- Billing plans & subscription management
- Super Admin portal (users, templates, subscriptions, payments, settings)
- Production PostgreSQL schema, policies, storage buckets, and seed data
- **Mobile app** (`mobile/`) — Expo + TypeScript for iOS/Android/tablets, same Supabase auth & data, offline check-in, push, QR scanning (see `mobile/README.md`)

## Quick start (demo mode)

InviteHub runs fully in **demo mode** without Supabase credentials — ideal for local UI/product work.

```bash
cd invitehub
npm install
npm run dev
```

Demo logins:

| Email | Password | Role |
|---|---|---|
| `demo@invitehub.am` | `demo1234` | User |
| `admin@invitehub.am` | `demo1234` | Super Admin |

Sample public invite: [http://localhost:5173/andranik-wedding](http://localhost:5173/andranik-wedding) or `/invite/a8H72L`

## Production setup (Supabase)

1. Create a Supabase project.
2. Apply SQL in order (see `database/README.md` / `database/INSTALL.md`):
   - `001` … `010_app_contract.sql` (required for web builder)
   - `011_mobile_checkin_push.sql` (required for mobile check-in + push)
3. Enable Auth: Email + Password, confirm email, recovery emails.
4. Copy `.env.example` → `.env` and set `VITE_SUPABASE_*`.
5. Promote a super admin:

```sql
UPDATE public.profiles SET role = 'super_admin' WHERE email = 'you@example.com';
```

6. Deploy to Vercel.
## Architecture

```
src/
  components/   # UI kit + feature components
  pages/        # Route-level screens
  layouts/      # Public / App / Admin shells
  contexts/     # Auth + Toast
  services/     # Business logic (Supabase or demo store)
  hooks/        # Shared hooks
  lib/          # Supabase client + demo store
  constants/    # Routes, labels, defaults
  styles/       # Design system
database/       # Schema, RLS, seed, storage
```

## Security model

- Every invitation/guest/gift/analytics row is owned by a user
- PostgreSQL RLS enforces tenant isolation
- Public read only for **published** invitations and related RSVP/gift claim flows
- Admin helpers: `is_admin()`, `is_super_admin()`, `owns_invitation()`

## Scripts

| Command | Description |
|---|---|
| `npm run dev` | Local development |
| `npm run build` | Production build |
| `npm run preview` | Preview production build |
| `npm run lint` | Lint |

## License

Proprietary — InviteHub
