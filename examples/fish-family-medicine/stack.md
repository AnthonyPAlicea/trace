---
stack: Fish-Family-Medicine-V1
---

BACKEND:
- Runtime: Node.js (v20)
- Framework: Fastify
- Language: TypeScript

DATA:
- Database: Supabase (Postgres)
- ORM: Prisma
- Realtime: Supabase Channels

FRONTEND:
- Framework: TanStack Start
- State: Zustand
- Styling: Tailwind CSS

SYSTEM DEFAULTS:
- All entities have id (UUID), created_at, and updated_at unless stated otherwise.
- All writes are audited with actor_id.
- Money is stored in cents as integers. Displayed as USD.
- Auth users are managed by Supabase Auth. Patient.user_id, Provider.user_id, and FrontDesk.user_id link to auth.users.
- Domain events are persisted in an outbox table and processed by a worker. Handlers must be idempotent.
