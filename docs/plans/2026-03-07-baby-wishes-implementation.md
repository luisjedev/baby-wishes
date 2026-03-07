# Baby Wishes Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build Baby Wishes v1 with authenticated wishlists, invitation-based access, item reservation and manual purchase confirmation flows.

**Architecture:** Start from a TanStack Start project scaffold created manually with the official CLI, then layer Convex as the single source of truth for domain rules, authorization and realtime projections. Organize frontend code by feature so each user flow owns its routes, forms, queries and UI states without scattering business logic across the app.

**Tech Stack:** TanStack Start, React, TypeScript, Tailwind CSS, shadcn/ui, Convex, Netlify, GitHub, Vitest or Jest-compatible test runner

---

## Prerequisite

Before executing this plan, create the base TanStack Start project manually with the official CLI. This plan assumes the scaffold already exists and focuses on adapting it to Baby Wishes instead of generating the shell from scratch.

### Task 1: Align the generated TanStack Start scaffold with project conventions

**Files:**
- Modify: `package.json`
- Modify: `tsconfig.json`
- Modify: `vite.config.ts`
- Create: `postcss.config.js`
- Create: `tailwind.config.ts`
- Create: `components.json`
- Modify: `src/routes/__root.tsx`
- Modify: `src/routes/index.tsx`
- Create: `src/styles/app.css`
- Create: `src/lib/providers/app-provider.tsx`
- Test: `test/units/src/routes/root.test.tsx`

**Step 1: Write the failing test**

```tsx
import { render, screen } from '@testing-library/react'
import { RootRouteShell } from '@/routes/__root'

describe('RootRouteShell', () => {
  it('renders the public landing shell', () => {
    render(<RootRouteShell />)
    expect(screen.getByText('Baby Wishes')).toBeInTheDocument()
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/routes/root.test.tsx`
Expected: FAIL because the generated scaffold does not yet expose the Baby Wishes root shell.

**Step 3: Write minimal implementation**

Adapt the generated TanStack Start app, add global providers, connect Tailwind entrypoints and replace the default landing shell with the Baby Wishes root shell.

```tsx
export function RootRouteShell() {
  return <div className="min-h-screen bg-stone-50">Baby Wishes</div>
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/routes/root.test.tsx`
Expected: PASS

**Step 5: Commit**

```bash
git add package.json tsconfig.json vite.config.ts postcss.config.js tailwind.config.ts components.json src/styles/app.css src/routes/__root.tsx src/routes/index.tsx src/lib/providers/app-provider.tsx test/units/src/routes/root.test.tsx
git commit -m "chore: align tanstack scaffold with app conventions"
```

### Task 2: Configure Convex schema and user profile domain

**Files:**
- Create: `convex/schema.ts`
- Create: `convex/auth.ts`
- Create: `convex/users.ts`
- Create: `src/features/profile/profile.types.ts`
- Create: `test/units/convex/users.test.ts`

**Step 1: Write the failing test**

```ts
import { createUserProfile } from '@/../convex/users'

describe('createUserProfile', () => {
  it('requires a visible name for Google sign-ins', async () => {
    await expect(
      createUserProfile({ provider: 'google', email: 'ana@example.com', visibleName: '' })
    ).rejects.toThrow('VISIBLE_NAME_REQUIRED')
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/users.test.ts`
Expected: FAIL because the user domain functions are missing.

**Step 3: Write minimal implementation**

Define Convex tables for users, wishlists, invitations, memberships, items, reservations and purchases. Add user helpers that normalize profile creation and enforce `visibleName` for accounts that do not have one yet.

```ts
if (!args.visibleName.trim()) {
  throw new Error('VISIBLE_NAME_REQUIRED')
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/users.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/schema.ts convex/auth.ts convex/users.ts src/features/profile/profile.types.ts test/units/convex/users.test.ts
git commit -m "feat: add user profile domain"
```

### Task 3: Add authentication routes and onboarding flow

**Files:**
- Create: `src/routes/auth/login.tsx`
- Create: `src/routes/auth/register.tsx`
- Create: `src/routes/onboarding.tsx`
- Create: `src/features/auth/components/login-form.tsx`
- Create: `src/features/auth/components/register-form.tsx`
- Create: `src/features/profile/components/onboarding-form.tsx`
- Create: `src/features/auth/auth.client.ts`
- Create: `src/features/auth/auth.guards.ts`
- Test: `test/units/src/features/auth/onboarding-redirect.test.tsx`

**Step 1: Write the failing test**

```tsx
import { resolvePostAuthDestination } from '@/features/auth/auth.guards'

describe('resolvePostAuthDestination', () => {
  it('sends the user to onboarding when visible name is missing', () => {
    expect(
      resolvePostAuthDestination({ visibleName: '', pendingInviteToken: 'abc123' })
    ).toBe('/onboarding?invite=abc123')
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/features/auth/onboarding-redirect.test.tsx`
Expected: FAIL because the redirect helper is missing.

**Step 3: Write minimal implementation**

Implement auth routes, login and register forms, and an onboarding page that blocks access until `visibleName` is present. Keep pending invitation token in the redirect state or query string.

```ts
export function resolvePostAuthDestination(input: { visibleName: string; pendingInviteToken?: string }) {
  if (!input.visibleName.trim()) {
    return input.pendingInviteToken ? `/onboarding?invite=${input.pendingInviteToken}` : '/onboarding'
  }

  return input.pendingInviteToken ? `/invite/${input.pendingInviteToken}` : '/app'
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/features/auth/onboarding-redirect.test.tsx`
Expected: PASS

**Step 5: Commit**

```bash
git add src/routes/auth/login.tsx src/routes/auth/register.tsx src/routes/onboarding.tsx src/features/auth/components/login-form.tsx src/features/auth/components/register-form.tsx src/features/profile/components/onboarding-form.tsx src/features/auth/auth.client.ts src/features/auth/auth.guards.ts test/units/src/features/auth/onboarding-redirect.test.tsx
git commit -m "feat: add auth and onboarding flow"
```

### Task 4: Implement wishlist domain and owner list creation

**Files:**
- Create: `convex/wishlists.ts`
- Create: `src/routes/app/index.tsx`
- Create: `src/routes/app/my-list.tsx`
- Create: `src/features/lists/list.types.ts`
- Create: `src/features/lists/components/my-list-page.tsx`
- Create: `test/units/convex/wishlists.test.ts`

**Step 1: Write the failing test**

```ts
import { ensureActiveWishlistForOwner } from '@/../convex/wishlists'

describe('ensureActiveWishlistForOwner', () => {
  it('returns the same wishlist when the owner already has one active list', async () => {
    const wishlistId = await ensureActiveWishlistForOwner({ userId: 'user_1' })
    const secondCallId = await ensureActiveWishlistForOwner({ userId: 'user_1' })
    expect(secondCallId).toBe(wishlistId)
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/wishlists.test.ts`
Expected: FAIL because the wishlist domain does not exist.

**Step 3: Write minimal implementation**

Create the wishlist domain with one active list per owner and wire the authenticated home route so it sends the owner to a hub with `Mi lista` and `Listas a las que me uní`.

```ts
if (existingWishlist) {
  return existingWishlist._id
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/wishlists.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/wishlists.ts src/routes/app/index.tsx src/routes/app/my-list.tsx src/features/lists/list.types.ts src/features/lists/components/my-list-page.tsx test/units/convex/wishlists.test.ts
git commit -m "feat: add owner wishlist domain"
```

### Task 5: Implement owner item CRUD and owner-only filters

**Files:**
- Create: `convex/items.ts`
- Create: `src/features/items/item.types.ts`
- Create: `src/features/items/components/item-form-dialog.tsx`
- Create: `src/features/items/components/owner-item-list.tsx`
- Create: `src/features/items/components/owner-item-filters.tsx`
- Create: `src/features/items/item.filters.ts`
- Test: `test/units/src/features/items/item-filters.test.ts`
- Test: `test/units/convex/items.test.ts`

**Step 1: Write the failing test**

```ts
import { filterOwnerItems } from '@/features/items/item.filters'

describe('filterOwnerItems', () => {
  it('returns only free items for the Libres tab', () => {
    const items = [
      { id: '1', state: 'Libre' },
      { id: '2', state: 'Reservado' },
      { id: '3', state: 'Comprado' },
    ]

    expect(filterOwnerItems(items, 'Libres')).toEqual([{ id: '1', state: 'Libre' }])
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/features/items/item-filters.test.ts`
Expected: FAIL because filters and item list logic are missing.

**Step 3: Write minimal implementation**

Implement item CRUD in Convex and owner UI components for creation, edition, deletion and filtering by `Todos`, `Libres`, `Reservados` and `Comprados`.

```ts
export function filterOwnerItems(items: Array<{ state: string }>, tab: string) {
  if (tab === 'Todos') return items
  if (tab === 'Libres') return items.filter((item) => item.state === 'Libre')
  if (tab === 'Reservados') return items.filter((item) => item.state === 'Reservado')
  return items.filter((item) => item.state === 'Comprado')
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/features/items/item-filters.test.ts`
Run: `npm run test test/units/convex/items.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/items.ts src/features/items/item.types.ts src/features/items/components/item-form-dialog.tsx src/features/items/components/owner-item-list.tsx src/features/items/components/owner-item-filters.tsx src/features/items/item.filters.ts test/units/src/features/items/item-filters.test.ts test/units/convex/items.test.ts
git commit -m "feat: add owner item management"
```

### Task 6: Implement invitation tokens and membership creation

**Files:**
- Create: `convex/invitations.ts`
- Create: `convex/memberships.ts`
- Create: `src/routes/invite/$token.tsx`
- Create: `src/features/invitations/invitation.types.ts`
- Create: `src/features/invitations/components/invitation-entry.tsx`
- Create: `src/features/memberships/membership.types.ts`
- Test: `test/units/convex/invitations.test.ts`
- Test: `test/units/src/features/invitations/invite-entry.test.tsx`

**Step 1: Write the failing test**

```ts
import { joinWishlistFromInvite } from '@/../convex/invitations'

describe('joinWishlistFromInvite', () => {
  it('creates a membership when the token is valid and not expired', async () => {
    const result = await joinWishlistFromInvite({ token: 'valid-token', userId: 'user_2' })
    expect(result.status).toBe('joined')
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/invitations.test.ts`
Expected: FAIL because the invitation domain is missing.

**Step 3: Write minimal implementation**

Implement one active invitation token per wishlist, token validation, 30-day expiry and idempotent membership creation. The invite route must redirect to auth when needed and continue the flow afterwards.

```ts
if (invitation.expiresAt < Date.now()) {
  throw new Error('INVITATION_EXPIRED')
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/invitations.test.ts`
Run: `npm run test test/units/src/features/invitations/invite-entry.test.tsx`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/invitations.ts convex/memberships.ts src/routes/invite/$token.tsx src/features/invitations/invitation.types.ts src/features/invitations/components/invitation-entry.tsx src/features/memberships/membership.types.ts test/units/convex/invitations.test.ts test/units/src/features/invitations/invite-entry.test.tsx
git commit -m "feat: add invitation and membership flow"
```

### Task 7: Build joined lists and contextual list projection

**Files:**
- Create: `src/routes/app/joined-lists/index.tsx`
- Create: `src/routes/app/joined-lists/$listId.tsx`
- Create: `src/features/lists/components/joined-lists-page.tsx`
- Create: `src/features/lists/components/list-context-page.tsx`
- Create: `src/features/lists/list.projections.ts`
- Test: `test/units/src/features/lists/list-projections.test.ts`

**Step 1: Write the failing test**

```ts
import { projectWishlistForViewer } from '@/features/lists/list.projections'

describe('projectWishlistForViewer', () => {
  it('hides purchased items from members', () => {
    const result = projectWishlistForViewer(
      [{ id: '1', state: 'Comprado' }, { id: '2', state: 'Reservado' }],
      'member'
    )

    expect(result).toEqual([{ id: '2', state: 'Reservado' }])
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/features/lists/list-projections.test.ts`
Expected: FAIL because the contextual projection logic is missing.

**Step 3: Write minimal implementation**

Implement the joined lists index and detail routes. The detail page must reuse the same underlying list data but render fields and actions according to `owner` or `member` context.

```ts
if (viewerRole === 'owner') return items
return items.filter((item) => item.state !== 'Comprado')
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/features/lists/list-projections.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add src/routes/app/joined-lists/index.tsx src/routes/app/joined-lists/$listId.tsx src/features/lists/components/joined-lists-page.tsx src/features/lists/components/list-context-page.tsx src/features/lists/list.projections.ts test/units/src/features/lists/list-projections.test.ts
git commit -m "feat: add contextual list views"
```

### Task 8: Implement reservation, cancellation and manual purchase confirmation

**Files:**
- Create: `convex/reservations.ts`
- Create: `convex/purchases.ts`
- Create: `src/routes/app/my-reservations.tsx`
- Create: `src/features/reservations/components/my-reservations-page.tsx`
- Create: `src/features/reservations/reservation.types.ts`
- Create: `src/features/purchases/purchase.types.ts`
- Test: `test/units/convex/reservations.test.ts`
- Test: `test/units/convex/purchases.test.ts`

**Step 1: Write the failing test**

```ts
import { reserveItem } from '@/../convex/reservations'

describe('reserveItem', () => {
  it('rejects reservation when the item is not free', async () => {
    await expect(reserveItem({ itemId: 'item_1', userId: 'user_2' })).rejects.toThrow('ITEM_NOT_FREE')
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/reservations.test.ts`
Expected: FAIL because reservation and purchase mutations are missing.

**Step 3: Write minimal implementation**

Implement atomic Convex mutations for reserve, cancel and confirm purchase. Expose a member-only reservations page that aggregates the user reservations across joined lists.

```ts
if (item.state !== 'Libre') {
  throw new Error('ITEM_NOT_FREE')
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/reservations.test.ts`
Run: `npm run test test/units/convex/purchases.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/reservations.ts convex/purchases.ts src/routes/app/my-reservations.tsx src/features/reservations/components/my-reservations-page.tsx src/features/reservations/reservation.types.ts src/features/purchases/purchase.types.ts test/units/convex/reservations.test.ts test/units/convex/purchases.test.ts
git commit -m "feat: add reservation and purchase flows"
```

### Task 9: Add owner member management and expulsion rollback

**Files:**
- Modify: `convex/memberships.ts`
- Modify: `convex/reservations.ts`
- Modify: `convex/purchases.ts`
- Create: `src/features/memberships/components/member-list.tsx`
- Test: `test/units/convex/memberships.test.ts`

**Step 1: Write the failing test**

```ts
import { expelMemberFromWishlist } from '@/../convex/memberships'

describe('expelMemberFromWishlist', () => {
  it('returns reserved and purchased items to Libre', async () => {
    const result = await expelMemberFromWishlist({ wishlistId: 'list_1', memberUserId: 'user_2', ownerUserId: 'user_1' })
    expect(result.revertedItems).toBe(2)
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/memberships.test.ts`
Expected: FAIL because expulsion rollback logic is not implemented.

**Step 3: Write minimal implementation**

Add owner-only member management and an expulsion mutation that removes membership and rewinds all affected items to `Libre`.

```ts
await revertMemberReservations(args.wishlistId, args.memberUserId)
await revertMemberPurchases(args.wishlistId, args.memberUserId)
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/memberships.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/memberships.ts convex/reservations.ts convex/purchases.ts src/features/memberships/components/member-list.tsx test/units/convex/memberships.test.ts
git commit -m "feat: add member expulsion rollback"
```

### Task 10: Add reservation expiry automation and owner settings

**Files:**
- Create: `convex/crons.ts`
- Modify: `convex/reservations.ts`
- Modify: `convex/wishlists.ts`
- Create: `src/features/lists/components/reservation-window-settings.tsx`
- Test: `test/units/convex/reservation-expiry.test.ts`

**Step 1: Write the failing test**

```ts
import { expireReservationsForWishlist } from '@/../convex/reservations'

describe('expireReservationsForWishlist', () => {
  it('moves expired reservations back to Libre', async () => {
    const result = await expireReservationsForWishlist({ now: Date.now() })
    expect(result.expiredCount).toBe(1)
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/convex/reservation-expiry.test.ts`
Expected: FAIL because expiry automation is missing.

**Step 3: Write minimal implementation**

Implement reservation duration settings on the wishlist, store expiry timestamps at reservation time and add a Convex cron or scheduled job that resets expired reservations to `Libre`.

```ts
const reservationWindowDays = wishlist.reservationWindowDays ?? 7
const expiresAt = Date.now() + reservationWindowDays * 24 * 60 * 60 * 1000
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/convex/reservation-expiry.test.ts`
Expected: PASS

**Step 5: Commit**

```bash
git add convex/crons.ts convex/reservations.ts convex/wishlists.ts src/features/lists/components/reservation-window-settings.tsx test/units/convex/reservation-expiry.test.ts
git commit -m "feat: add reservation expiry automation"
```

### Task 11: Harden route guards, error states and public/private UX

**Files:**
- Modify: `src/routes/__root.tsx`
- Modify: `src/routes/app/index.tsx`
- Modify: `src/routes/invite/$token.tsx`
- Create: `src/components/app/app-error-state.tsx`
- Create: `src/components/app/app-empty-state.tsx`
- Create: `test/units/src/routes/invite-error-state.test.tsx`

**Step 1: Write the failing test**

```tsx
import { render, screen } from '@testing-library/react'
import { InvitationErrorState } from '@/components/app/app-error-state'

describe('InvitationErrorState', () => {
  it('renders the expired invitation message', () => {
    render(<InvitationErrorState code="INVITATION_EXPIRED" />)
    expect(screen.getByText('Esta invitacion ha caducado')).toBeInTheDocument()
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/routes/invite-error-state.test.tsx`
Expected: FAIL because shared app states are missing.

**Step 3: Write minimal implementation**

Add reusable empty and error states for invite failures, unauthorized access and empty collections. Tighten route-level guards so no authenticated route leaks to outsiders.

```tsx
if (code === 'INVITATION_EXPIRED') {
  return <p>Esta invitacion ha caducado</p>
}
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/routes/invite-error-state.test.tsx`
Expected: PASS

**Step 5: Commit**

```bash
git add src/routes/__root.tsx src/routes/app/index.tsx src/routes/invite/$token.tsx src/components/app/app-error-state.tsx src/components/app/app-empty-state.tsx test/units/src/routes/invite-error-state.test.tsx
git commit -m "feat: add route guards and error states"
```

### Task 12: Prepare deployment, CI and contributor documentation

**Files:**
- Create: `.env.example`
- Create: `netlify.toml`
- Create: `.github/workflows/ci.yml`
- Modify: `README.md`
- Test: `test/units/src/routes/root.test.tsx`

**Step 1: Write the failing test**

```tsx
import { render, screen } from '@testing-library/react'
import { RootRouteShell } from '@/routes/__root'

describe('RootRouteShell metadata', () => {
  it('keeps the Baby Wishes brand visible in the shell', () => {
    render(<RootRouteShell />)
    expect(screen.getByText('Baby Wishes')).toBeInTheDocument()
  })
})
```

**Step 2: Run test to verify it fails**

Run: `npm run test test/units/src/routes/root.test.tsx`
Expected: FAIL only if the shell regressed during setup. If it already passes, keep the test and proceed to infrastructure changes.

**Step 3: Write minimal implementation**

Document local setup, required environment variables, Convex and Netlify deployment steps, and add a CI workflow that runs install, typecheck, lint and tests.

```yaml
- run: npm ci
- run: npm run lint
- run: npm run typecheck
- run: npm run test
```

**Step 4: Run test to verify it passes**

Run: `npm run test test/units/src/routes/root.test.tsx`
Expected: PASS

**Step 5: Commit**

```bash
git add .env.example netlify.toml .github/workflows/ci.yml README.md test/units/src/routes/root.test.tsx
git commit -m "chore: add deployment and ci docs"
```

## Implementation Notes

- Keep Convex as the only authority for permissions, invite validity and item state transitions.
- Do not derive security-sensitive access from frontend route state alone.
- Keep owner and member views on top of the same list data model; only projections and actions change.
- Use the `Libre`, `Reservado` and `Comprado` labels consistently across schema, UI and tests.
- Add design-system primitives only when they are used by a real screen.

## Testing Order

1. `npm run test test/units/convex/users.test.ts`
2. `npm run test test/units/convex/wishlists.test.ts`
3. `npm run test test/units/convex/items.test.ts`
4. `npm run test test/units/convex/invitations.test.ts`
5. `npm run test test/units/convex/reservations.test.ts`
6. `npm run test test/units/convex/purchases.test.ts`
7. `npm run test test/units/convex/memberships.test.ts`
8. `npm run test test/units/convex/reservation-expiry.test.ts`
9. `npm run test`

## Suggested Linear Breakdown

1. Epic: Foundation and app shell
2. Epic: Auth and onboarding
3. Epic: Owner wishlist management
4. Epic: Invitations and memberships
5. Epic: Member reservation flow
6. Epic: Reservation expiry and operational hardening
7. Epic: Deployment and CI