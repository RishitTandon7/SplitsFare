# SplitsFare — Architecture Notes

## Stack

- **Frontend**: Vite + React 18 SPA, client-only (no custom backend server), deployed on Vercel.
- **Auth**: Firebase Auth (Google sign-in only). `src/firebase.js` initializes Firebase Auth;
  `src/context/AuthContext.jsx` exposes `useAuth()` and syncs a `users` row in Supabase on login.
- **Database**: Supabase Postgres. **Not Firestore** — the app was migrated off Firestore;
  any older docs/comments mentioning Firestore collections, `onSnapshot`, or Firestore security
  rules are stale and describe the pre-migration architecture.

This is a two-identity-provider setup: Firebase issues the identity (ID token), Supabase only
stores and authorizes data access to app data. Supabase's own auth system (`auth.uid()`,
Supabase Auth users) is **not used** — there are no Supabase Auth users, no Supabase sessions.

## How Firebase auth reaches Supabase RLS

Supabase Postgres has no native concept of a Firebase user. This is bridged via **Supabase
Third-Party Auth**, configured once in the Supabase dashboard (Authentication → Third-Party
Auth → Firebase, entering the Firebase Project ID). That tells Supabase to trust and verify
Firebase-issued ID tokens.

Client wiring (`src/supabase.js`):

```js
createClient(url, anonKey, {
  accessToken: async () => {
    await auth.authStateReady(); // avoid a race right after page load/redirect
    return auth.currentUser?.getIdToken();
  },
})
```

Every Supabase request (REST + Realtime) forwards the current Firebase ID token as the bearer
token instead of a Supabase session token.

On the database side, `auth.jwt()->>'sub'` resolves to the Firebase UID once the token verifies.
**Do not use Supabase's built-in `auth.uid()`** — it casts the JWT `sub` claim to `uuid`, but
Firebase UIDs are arbitrary text strings (e.g. `OAylu6lThEYaDq8PjhDkeX0Eoho1`), not UUIDs, so
`auth.uid()` will error or silently return null. Instead every policy uses:

```sql
create or replace function firebase_uid() returns text
language sql stable as $$
  select auth.jwt()->>'sub'
$$;
```

**Every RLS policy in this project must use `firebase_uid()`, never `auth.uid()`.** Mixing the
two is the exact bug class that broke the friend-invite flow during migration (see below).

## Source of truth for schema + RLS

`supabase/schema.sql` is the authoritative, idempotent-ish definition of every table and policy.
There is no ORM/migration tool — changes are made by editing that file and re-running the
relevant `drop policy` / `create policy` (or table) statements directly in the Supabase SQL
editor. When changing a policy, prefer `drop policy if exists X; create policy X ...;` so re-runs
are safe, since Postgres has no `create or replace policy`.

### Tables

| Table | Purpose | Key columns |
|---|---|---|
| `users` | Mirror of the Firebase-authenticated user, keyed by Firebase UID | `id` (text, PK = Firebase UID), `name`, `email`, `email_normalized`, `upi_id`, `photo_url` |
| `friendships` | Bidirectional friend edges | `user_id`, `friend_id` (PK pair) |
| `groups` | Expense-splitting groups | `id` (uuid), `created_by`, `name` |
| `group_members` | Group membership (replaces a Firestore `members[]` array) | `group_id`, `user_id` (PK pair) |
| `expenses` | Expenses within a group | `id`, `group_id`, `paid_by`, `amount`, `split_type` |
| `expense_splits` | Per-member share of an expense (replaces a Firestore `splits{}` map) | `expense_id`, `user_id`, `amount` |
| `settlements` | Settle-up records between two members of a group | `id`, `group_id`, `from_user`, `to_user`, `status` |

### `friendships` RLS (bidirectional — read this before touching it again)

A friend-add writes **two rows in one request** — one from each user's perspective:

```sql
insert into friendships (user_id, friend_id) values
  (currentUid, otherUid),
  (otherUid, currentUid);
```

The second row has `user_id = otherUid`, i.e. **not** the currently-authenticated user. Any
policy that only checks `user_id = firebase_uid()` will reject that second row and fail the
whole insert. Every `friendships` policy must check **both** columns:

```sql
create policy friendships_select on friendships for select
  using (user_id = firebase_uid() or friend_id = firebase_uid());
create policy friendships_insert on friendships for insert
  with check (user_id = firebase_uid() or friend_id = firebase_uid());
create policy friendships_delete on friendships for delete
  using (user_id = firebase_uid() or friend_id = firebase_uid());
```

`group_members` has the analogous shape for the group-join-by-link flow (a user inserting a row
for themself into a group they're not yet in): its insert policy is
`user_id = firebase_uid() or is_group_member(group_id, firebase_uid())`.

### Upserts and RLS — a Postgres gotcha

`INSERT ... ON CONFLICT DO UPDATE` (Supabase `.upsert()` with the default `ignoreDuplicates:
false`) requires an `UPDATE` policy to exist on the table **even when no conflict actually
occurs** — Postgres checks it as part of planning the statement. `friendships` and
`group_members` intentionally have no `UPDATE` policy (nothing about a duplicate edge/membership
needs updating), so any upsert onto them **must** pass `ignoreDuplicates: true` (→
`ON CONFLICT DO NOTHING`) or it will fail with an RLS error even though the row values are fine.
See `JoinGroup.jsx` and `AddFriendInvite.jsx` for the pattern.

### `is_group_member`

A `security definer` helper (`supabase/schema.sql`) used inside `group_members`/`groups`/
`expenses`/`settlements` policies. It exists specifically to avoid **RLS self-recursion**: a
policy on `group_members` that queries `group_members` directly (to check "is this uid a member
of this group") would re-trigger its own RLS check infinitely. `security definer` bypasses RLS
for that one lookup.

## Realtime

Live updates (replacing Firestore's `onSnapshot`) use Supabase Realtime `postgres_changes`
subscriptions, centralized in `src/utils/groupsApi.js` (`subscribeMyGroups`, `subscribeGroup`,
`subscribeGroupExpenses`) and used directly in `Account.jsx`/`Friends.jsx` for their own tables.
All tables are added to the `supabase_realtime` publication in `schema.sql`.

## Auth Gotchas

### Symptom: 401 / 42501 on every INSERT despite correct RLS policies

**Exact error seen:**
```
POST /rest/v1/friendships  →  401 Unauthorized
{code: "42501", message: "new row violates row-level security policy for table \"friendships\""}
```

Both RLS policy text and JWT were verified correct. Even a single-row insert
where `user_id = firebase_uid()` was trivially true still failed.

**Root cause: `role: "anon"` custom claim on Firebase tokens**

PostgREST reads the `role` claim from the JWT and uses it as the Postgres role
for the entire session. Firebase ID tokens have no `role` claim by default —
but if anything (a one-off script, Firebase Console, or a prior Cloud Function)
has set `customClaims: { role: "anon" }` on a user, every token that user
receives will carry `role: "anon"`.

PostgREST will then run the session as the `anon` Postgres role instead of
`authenticated`. The `anon` role cannot read `current_setting('request.jwt.claims')`
in the same way, so `auth.jwt()` returns NULL during `WITH CHECK` evaluation of
INSERT/UPDATE/DELETE, making `firebase_uid()` return NULL, making every RLS
write policy fail — even when the USING/WITH CHECK logic is perfectly correct.

**Why SELECT still worked:** Supabase grants `anon` SELECT on public tables by
default, so reads silently succeeded even in the wrong role. Writes didn't.

**Regression test — run in DevTools console to verify role at any time:**
```js
const { data } = await window.supabase.rpc('debug_jwt');
console.log('role in JWT:', data?.role);  // must be "authenticated", NOT "anon"
```
`debug_jwt` is a `SECURITY DEFINER` RPC already in the DB:
```sql
create or replace function debug_jwt() returns jsonb
language sql stable security definer as $$
  select auth.jwt()
$$;
```

**The fixes applied:**

1. **Cloud Function** (`functions/index.js`) — `beforeUserCreated` trigger that
   stamps `{ role: "authenticated" }` on every new user before Firebase writes
   the account. Deployed via `firebase deploy --only functions`.

2. **Migration script** (`scripts/set-auth-claims.mjs`) — loops through all
   existing users via `admin.auth().listUsers()` and sets `role: "authenticated"`
   on any user that doesn't already have it. Run once after deploying the Cloud
   Function. See the script header for prerequisites and usage.

3. **`AuthContext.jsx`** — calls `getIdToken(true)` immediately after
   `signInWithPopup` returns, so even the very first Supabase request the client
   makes after sign-in carries the freshly-claimed token.

4. **`src/supabase.js`** — already calls `getIdToken(true)` on every request
   via the `accessToken` callback, preventing stale/expired tokens from being
   reused for Supabase calls.

**`firebase_uid()` must be SECURITY DEFINER**

After the role fix, `firebase_uid()` also needs `SECURITY DEFINER` so it can
always read `auth.jwt()` regardless of the calling Postgres role:

```sql
create or replace function firebase_uid() returns text
language sql stable security definer set search_path = public as $$
  select auth.jwt()->>'sub'
$$;
```

Run this in the Supabase SQL editor if not already applied. You can verify it
exists and is correct with:
```sql
select prosecdef, prosrc from pg_proc
join pg_namespace n on n.oid = pronamespace
where proname = 'firebase_uid' and n.nspname = 'public';
-- prosecdef should be true
```

**How to re-check after any future auth changes:**
1. Open the app in the browser, open DevTools console.
2. Run `const { data } = await window.supabase.rpc('debug_jwt'); console.log(data?.role);`
3. Confirm `"authenticated"`. If you see `"anon"` or `null`, re-run the
   migration script and re-deploy the Cloud Function.

