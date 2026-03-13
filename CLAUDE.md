# Baby Wishes — Project Context

## What this project is

Baby Wishes is a private pregnancy gift wishlist app. A pregnant user (owner) creates and manages a single active wishlist and invites friends/family (members) via a shareable link. Members can reserve items, cancel their own reservations and confirm purchases. All flows require authentication; the landing page is the only public route.

## Tech stack

| Layer | Technology |
|---|---|
| Framework | TanStack Start (full-stack React) |
| Language | TypeScript (strict) |
| Styling | Tailwind CSS 4 + shadcn/ui (Radix UI primitives) |
| Forms | @tanstack/react-form + Zod |
| Server state | TanStack Query + @convex-dev/react-query |
| Backend & DB | Convex (queries, mutations, crons, auth) |
| Auth | @convex-dev/auth + Google OAuth |
| Deployment | Netlify + GitHub |
| Linting | Biome via Ultracite preset — run `pnpm dlx ultracite fix` before committing |

## Key commands

```bash
pnpm dev                    # Start dev server (port 3000)
pnpm build                  # Production build
pnpm test                   # Run Vitest
pnpm dlx ultracite fix      # Auto-fix lint/format issues
pnpm dlx ultracite check    # Check without fixing
npx convex dev              # Start Convex dev server
```

## Directory structure

```
convex/                     # All backend logic (source of truth)
  schema.ts                 # Table definitions
  auth.ts                   # Auth provider config (Google)
  users.ts                  # User profile domain
  wishlists.ts              # Wishlist domain
  items.ts                  # Item CRUD
  invitations.ts            # Invitation tokens
  memberships.ts            # Membership + expulsion rollback
  reservations.ts           # Reserve, cancel, expire
  purchases.ts              # Purchase confirmation
  crons.ts                  # Scheduled reservation expiry

src/
  routes/                   # TanStack file-based routes
    __root.tsx              # Root layout (providers, theme)
    index.tsx               # Public landing page
    auth/
      login.tsx
      register.tsx
    onboarding.tsx          # Required when visibleName is missing
    app/
      index.tsx             # Hub: Mi lista / Listas a las que me uní
      my-list.tsx           # Owner private view
      joined-lists/
        index.tsx           # Member list index
        $listId.tsx         # Member list detail
      my-reservations.tsx   # Aggregated member reservations
    invite/
      $token.tsx            # Token validation → auth → membership

  features/                 # Feature-organized frontend code
    auth/
    profile/
    lists/
    items/
    invitations/
    memberships/
    reservations/
    purchases/

  components/
    ui/                     # shadcn/ui primitives only
    app/                    # Shared app-level components (error/empty states)

  integrations/
    convex/provider.tsx     # ConvexQueryClient + ConvexProvider
    tanstack-query/         # QueryClient + devtools

  lib/
    utils.ts                # cn() — clsx + tailwind-merge

test/
  units/
    convex/                 # Convex domain logic tests
    src/                    # Frontend unit tests (mirrors src/)
```

## Domain model

### Entities

- **User** — authenticated account, requires `visibleName`
- **Wishlist** — one active list per owner; holds `reservationWindowDays` setting
- **WishlistMember** — persists membership (not the invite link); source of ongoing authorization
- **WishlistItem** — name, image, price, optional link/site; derived state
- **ItemReservation** — who reserved, when, expiry timestamp
- **ItemPurchase** — manual purchase confirmation by the reserver
- **WishlistInvitation** — shareable token, expires in 30 days

### Item states and valid transitions

```
Libre → Reservado        (member reserves)
Reservado → Libre        (member cancels / reservation expires / member expelled)
Reservado → Comprado     (member confirms purchase)
Comprado → Libre         (member who purchased gets expelled)
```

### Roles (contextual, not global)

- **Owner** — sees everything including who reserved/purchased; full CRUD; manages members
- **Member** — sees Libre + Reservado (anonymous); can reserve, cancel own, confirm own purchase; cannot see Comprado
- **Outsider** — no access to list content

## Business rules (enforce in Convex, never only in UI)

- One active wishlist per owner — `ensureActiveWishlistForOwner` is idempotent
- Invitation token expires in 30 days; joining with an expired token returns `INVITATION_EXPIRED`
- Joining with a valid token is idempotent (no duplicate memberships)
- Owner joining via own invite link resolves as owner with no side effects
- Reservation expiry: 7 days default; owner can set 7 / 14 / 30 days
- Expired reservations revert to `Libre` via Convex cron
- Expelling a member reverts all their reservations and purchases to `Libre` atomically
- Reserving a non-free item throws `ITEM_NOT_FREE`; UI must re-fetch after rejection
- `visibleName` is mandatory — throw `VISIBLE_NAME_REQUIRED` on profile creation if empty

## Authentication and onboarding

- Google OAuth via `@convex-dev/auth`
- After login/register: if `visibleName` is missing → redirect to `/onboarding`
- Pending invite token is preserved through auth in query string (`?invite=<token>`)
- Helper: `resolvePostAuthDestination({ visibleName, pendingInviteToken })` → string

## Routes summary

| Route | Auth | Notes |
|---|---|---|
| `/` | Public | Landing/marketing |
| `/auth/login` | Public | Email+password or Google |
| `/auth/register` | Public | |
| `/onboarding` | Required | Blocks until visibleName set |
| `/app` | Required | Hub |
| `/app/my-list` | Required | Owner view |
| `/app/joined-lists` | Required | Member list index |
| `/app/joined-lists/$listId` | Required + membership | Member detail |
| `/app/my-reservations` | Required | Aggregated member reservations |
| `/invite/$token` | Validates token, forces auth | Creates membership on success |

## Convex conventions

- Queries return already-authorized projections — the backend decides what each viewer sees
- Mutations enforce all business rules and permissions; UI is never the last line of defense
- Use item state labels consistently: `Libre`, `Reservado`, `Comprado` (in Spanish, in code too)
- Scheduled jobs live in `convex/crons.ts`

## Frontend conventions

- Organize by feature under `src/features/`, not by file type
- Each feature owns its routes, components, types and client-side logic
- Add shadcn/ui components only when a real screen needs them
- Use `cn()` from `src/lib/utils.ts` for all className composition
- Path aliases: `@/*` and `#/*` both resolve to `src/`

## Testing approach

Tests live in `test/units/` mirroring the source tree. Run domain tests first:

```
test/units/convex/users.test.ts
test/units/convex/wishlists.test.ts
test/units/convex/items.test.ts
test/units/convex/invitations.test.ts
test/units/convex/reservations.test.ts
test/units/convex/purchases.test.ts
test/units/convex/memberships.test.ts
test/units/convex/reservation-expiry.test.ts
```

Each task in the implementation plan follows TDD: write failing test → implement → confirm passing → commit.

## Implementation status

- **Phase 0 (Bootstrap)** ✅ — TanStack Start, Convex, Tailwind, shadcn/ui, theme, routing, Google auth configured
- **Phase 1–4** ⏳ — Auth flows, wishlist CRUD, items, invitations, memberships, reservations, purchases, expiry automation not yet implemented

See `docs/plans/2026-03-07-baby-wishes-implementation.md` for the full 12-task TDD plan.

## Environment variables

```
VITE_CONVEX_URL=           # Convex deployment URL
CONVEX_DEPLOY_KEY=         # For CI deployment
AUTH_GOOGLE_ID=            # Google OAuth client ID
AUTH_GOOGLE_SECRET=        # Google OAuth client secret
```
