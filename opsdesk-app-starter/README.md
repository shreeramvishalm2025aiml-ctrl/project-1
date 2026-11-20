# OpsDesk — B2B Travel Operations CRM

A React + TypeScript + Vite operations console backed by Supabase Auth and PostgreSQL.

## Quick start

```bash
npm install
cp .env.example .env
# Set VITE_SUPABASE_URL and VITE_SUPABASE_PUBLISHABLE_KEY
npm run dev
```

Open http://localhost:5173. Production build: `npm run build`.

## Supabase

Project: `hdthpcppgrfjlujvmexh` · Mumbai (`ap-south-1`)

The database schema and demo seed data are provisioned in the project. The frontend uses the publishable key only. Never place a `service_role` or secret key in Vite/browser environment variables.

## Auth and demo identities

The `users` rows are seeded as application profiles. A database trigger links a newly created Supabase Auth identity to a seeded profile only when the email exactly matches. This does not create passwords or bypass email verification.

Create Auth users through Supabase Dashboard → Authentication → Users → Add user (or invite by email). Provision one identity for each desired demo email:
- `vishal@horizon.example` — requester / Horizon Travels
- `ops@desertstar.example` — requester / Desert Star Holidays
- `support@eurowings.example` — requester / EuroWings B2B
- `priya.sharma@tbo.example` — agent
- `arjun.menon@tbo.example` — agent
- `daniel.keller@tbo.example` — agent
- `maya.rao@tbo.example` — agent
- `omar.haddad@tbo.example` — escalation manager
- `sofia.laurent@tbo.example` — escalation manager
- `alex.morgan@tbo.example` — lead

For demo password access, set a password during Dashboard user creation and keep it in a secure password manager. Do not reuse a shared production password. Role switching is intentionally not a client-side dropdown: sign in as a different provisioned Auth identity to change roles.

## Implemented in this vertical slice

- [x] Supabase Postgres schema, seed agencies/users/bookings/tickets.
- [x] Auth session persistence and email/password + magic-link sign-in UI.
- [x] Auth identity ↔ seeded OpsDesk profile link trigger.
- [x] Queue, triage, my queue, escalation queue and basic filters.
- [x] Ticket creation, ticket detail, linked cases, message composer and timeline.
- [x] Public replies/internal notes visibility in UI and RLS.
- [x] Assignment/transfer and escalation RPC workflows.
- [x] Supplier side-conversation recipient list is separate from ticket assignment.
- [x] Lead-only insights and export screen.
- [x] Paged CSV/JSON export without an artificial 3,000-row cap.
- [x] Lead-only archival RPC enforces Closed + 90-day age.
- [ ] Automated routing scoring on ticket creation (current UI creates unassigned tickets; assignment RPC supports manual transfer).
- [ ] True server-streamed export for very large datasets (client paginates and downloads a Blob; use an Edge Function for 50K+ rows).
- [ ] Realtime subscriptions and full archive history browsing.
- [ ] Full demo Auth identities need to be provisioned in the Supabase Auth dashboard.

## Security and assumptions

- Single-tenant TBO operations organization.
- Supabase Auth establishes identity; `public.users` provides application role and agency membership.
- Requester agency isolation is enforced by PostgreSQL RLS, not UI filtering.
- Internal agent/manager/lead users are organization-wide; lead-only RPCs are gated by the authenticated profile role.
- No service-role key is present in this repository.
- Shifts and availability are seeded/simulated; live workforce scheduling is not integrated.
- Intent suggestion in this slice is a deterministic client-side keyword helper placeholder; the form currently uses the selected intent field.
- SLA values in the seed data are demonstration values. Production SLA calculation should be moved into a database function/Edge Function and recomputed as check-in/LCD approaches.
- Side conversation email delivery is not connected to an outbound email provider; the thread/recipient persistence is implemented.
- For 50K+ exports, use a server-side Edge Function with keyset pagination and streaming response; for near-realtime sync use Supabase Realtime plus an outbox/CDC pipeline.

## Demo walkthrough (3–5 minutes)

1. **Journey A — repeat refund:** Sign in as `vishal@horizon.example`, open OPS-1003, show the linked TBOH-984210 history.
2. **Journey B — routing:** Sign in as a TBO agent, open Triage, open OPS-1004 and transfer to an available MEA specialist; read the routing audit reason.
3. **Journey C — side conversation:** Open OPS-1014, show SC-12 recipients, transfer the ticket and confirm the recipient list remains unchanged.
4. **Journey D — SLA/escalation:** Open OPS-1007 or OPS-1008, show the Breached badge and escalation details. Escalate another urgent ticket with a reason.
5. **Journey E — insights:** Sign in as `alex.morgan@tbo.example`, open Insights, review weekly themes and product bug references; open Data & Export and download CSV/JSON.

## Troubleshooting

- If the app says Supabase is not configured, verify `.env` variable names and restart Vite.
- If a seeded profile cannot sign in, provision the exact matching email in Supabase Auth. Check `public.users.auth_user_id` was populated by the trigger.
- If API calls return 403, inspect the signed-in profile and RLS policies. Do not fix access errors by exposing a service-role key.
