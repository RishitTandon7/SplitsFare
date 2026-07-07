# NOTES.md — SplitFair Assumptions & Decisions

## Authentication
- **Phone number auth not implemented.** The Stitch login screen shows a phone number input, but Firebase Phone Auth requires reCAPTCHA setup and a paid plan for most volumes. This implementation uses **Google Sign-In** instead, which is simpler and more reliable. The receipt card login UI is preserved; the input is replaced with a Google CTA button.
- If phone auth is desired, replace the sign-in function in `AuthContext.jsx` with Firebase's `signInWithPhoneNumber` flow.
- Google Sign-In is configured using a **hybrid strategy** (`signInWithPopup` with auto-fallback to `signInWithRedirect`) to resolve Cross-Origin-Opener-Policy (COOP) blocks when testing locally or running inside sandboxed frames (e.g. IDE browser preview tabs, embedding frames).
- The Firestore user profile document upsert inside the `onAuthStateChanged` hook is wrapped in a robust `try-catch` block. This prevents any Firestore configuration or rules issues (e.g. if Cloud Firestore is not yet created or rules deny reading/writing in the Firebase Console) from crashing the auth flow and causing infinite redirect loops back to `/login`.

## Light Theme
- The Stitch export only ships a **dark theme**. The light palette in `index.css` is **extrapolated** from the dark palette by inverting the luminance while preserving the brand colors (warm-paper `#F7F3EA`, ink-navy `#14181C`, gold `#e8c17b`). All components use CSS custom properties (`var(--color-*)`) so both themes work without duplicated styles.

## UPI Deep Link Behavior
- On mobile, `window.location.href = upiLink` opens the native UPI app selector.
- On desktop browsers, the UPI scheme is unregistered, so nothing may open. The confirm/dispute prompt still appears after 2.5s either way.
- After the payer clicks "Yes, I Paid", a `pending_confirmation` settlement doc is written to Firestore. The recipient can confirm or dispute from their GroupDetail view (settlement docs are visible via real-time listener).

## Stitch Screens Extrapolated (not directly exported)
- **Account / Settings page** — not in the Stitch export. The nav item links to `/account` which redirects to Dashboard. A placeholder page can be added later.
- **Friends page** — not in the Stitch export. Nav item present; page is a placeholder redirect.
- **Loading states** — extrapolated from the dark palette: spinning `sync` icon in gold on dark background.
- **Empty states** — extrapolated: ghosted icon + label-mono text centered.
- **Error states** — use the `error` token (`#ffb4ab` dark / `#ba1a1a` light).

## Recurring Expenses
- The `recurring.frequency` field is saved to Firestore (`weekly`, `monthly`, or `null`).
- Automatic re-creation of recurring expenses is **not implemented** (would require Cloud Functions or a cron job). The field is stored so it can be acted on server-side later.

## Debt Simplification Algorithm
- Uses a greedy min-cash-flow approach: sort creditors and debtors by absolute amount, greedily settle the smaller balance, repeat. This is `O(n²)` which is fine for groups up to ~100 members.

## Firestore Indexes Required
- `expenses` collection group query with `orderBy('createdAt', 'desc')` on the Activity page requires a composite index. Firebase will show a link in the console to create it automatically on first run.

## Image Assets
- The Stitch export references Google-hosted AIDA images that require specific referer headers. These are not copied into `src/assets/` as they are external. User profile photos come from Firebase Auth (Google profile picture URL).
