# SplitFair — Build Spec for Claude Code

Build a complete, working expense-splitting web app called **SplitFair** — a Splitwise alternative with direct UPI payments. Follow this spec end-to-end and produce a runnable project. Ask no clarifying questions — where something is ambiguous, make the most sensible choice and note it in a `NOTES.md` file at the end.

---

## 1. Tech stack

- **Frontend:** React 18 + Vite
- **Routing:** react-router-dom v6
- **Styling:** Tailwind CSS (custom theme, see design system below)
- **Backend/Auth/DB:** Firebase (Auth + Firestore + Storage), client SDK only — no custom server
- **State:** React Context + Firestore real-time listeners (no Redux)
- **Deployment target:** static build, Firebase Hosting compatible

---

## 2. Design system — USE THE STITCH EXPORT, DO NOT INVENT A NEW UI

The UI has already been designed in Google Stitch and exported. The exported files live in `splitfair/stitch-export/` (HTML/CSS, possibly with image assets). **Claude Code must treat this export as the source of truth for all visual design** — colors, spacing, fonts, component layout, border-radius, shadows, iconography — and reproduce it as React components. Do NOT redesign, restyle, "improve," or substitute a generic Tailwind default theme instead of it.

Process:
1. Read every HTML/CSS file in `splitfair/stitch-export/` before writing any component.
2. Extract the actual color values, font-family declarations, spacing scale, and border-radius values used in the export, and put them into `tailwind.config.js` as theme extensions (do not guess — use the literal values found in the exported CSS).
3. Convert each exported screen into a corresponding React page/component (see routes in Section 7), preserving exact markup structure, visual styling, and asset references (copy images into `src/assets/`).
4. **Two themes exist in the Stitch export: dark and light.** Implement theme switching via a `ThemeContext` that toggles a `data-theme="dark" | "light"` attribute on `<html>`, with both palettes defined as CSS custom properties (extracted from the export) so every component works in both modes without duplicated styles. Persist the user's choice in `localStorage`. Default to the user's OS preference (`prefers-color-scheme`) on first visit.
5. Any screen not covered by the Stitch export (e.g. edge-case empty states, error states, loading states) should closely extrapolate the established visual language (same fonts, colors, spacing, component shapes) rather than falling back to Tailwind defaults.

### Motion (keep snappy, 150–250ms, respect `prefers-reduced-motion`)
Layer these interactions on top of the Stitch visuals — exports are static, so motion still needs to be built:
- Balance numbers count up/down (odometer effect) when they change
- New expense cards "print in" from the top with a slide + fade, staggered if multiple
- "Pay with UPI" button: press scale-down, then transitions into confirmation screen
- On full group settle-up: brief celebration burst using the export's accent colors
- Theme toggle (dark/light) animates as a smooth cross-fade, not an instant snap

---

## 3. Core features (and the Splitwise flaws each one fixes)

| Feature | Fixes |
|---|---|
| UPI deep link on "Pay Now" (`upi://pay?pa=...&pn=...&am=...&cu=INR&tn=...`) | Splitwise "settle up" doesn't move real money — this opens the user's actual UPI app with amount + note pre-filled |
| Two-sided payment confirmation (payer marks paid → recipient confirms or disputes) | Splitwise has no proof a payment happened |
| Free debt-simplification algorithm (min-cash-flow graph reduction) | Splitwise paywalls this; show it prominently as "Simplified Settlements" |
| Split types: equal / exact amount / percentage / shares | Splitwise's flow for uneven splits is clunky |
| Real-time Firestore listeners on balances/activity feed | Splitwise sync feels laggy |
| Optional recurring expenses | Splitwise requires manual re-entry every time |
| Live group activity feed | Splitwise buries activity |

---

## 4. Data model (Firestore)

```
users/{uid}
  - name: string
  - email: string
  - upiId: string          // e.g. "rishit@okhdfcbank"
  - photoURL: string

groups/{groupId}
  - name: string
  - members: string[]      // array of uids
  - createdBy: uid
  - createdAt: timestamp

groups/{groupId}/expenses/{expenseId}
  - description: string
  - amount: number
  - paidBy: uid
  - splitType: "equal" | "exact" | "percentage" | "shares"
  - splits: { [uid]: number }   // computed share per member
  - category: string
  - recurring: { frequency: "weekly"|"monthly"|null }
  - createdAt: timestamp

groups/{groupId}/settlements/{settlementId}
  - from: uid
  - to: uid
  - amount: number
  - status: "pending_confirmation" | "confirmed" | "disputed"
  - initiatedAt: timestamp
  - confirmedAt: timestamp | null
```

---

## 5. Debt simplification algorithm

Implement in `src/utils/splitEngine.js`:
1. Compute net balance per member across all expenses in a group (positive = owed money, negative = owes money).
2. Sort creditors (positive balances) and debtors (negative balances).
3. Greedily match the largest debtor to the largest creditor, settle the smaller of the two amounts, repeat until all balances are ~0.
4. Output a minimal list of `{ from, to, amount }` transactions — this is the "Simplified Settlements" feature shown in the group view.

---

## 6. UPI deep link utility

Implement in `src/utils/upi.js`:
```js
export function buildUpiLink({ payeeVpa, payeeName, amount, note }) {
  const params = new URLSearchParams({
    pa: payeeVpa,
    pn: payeeName,
    am: amount.toFixed(2),
    cu: 'INR',
    tn: note || 'SplitFair settlement',
  });
  return `upi://pay?${params.toString()}`;
}
```
On the Settle Up screen, render this as a link/button (`<a href={upiLink}>`). On click, after a short delay, prompt the payer to confirm "Did the payment go through?" and write a `pending_confirmation` settlement doc; the recipient then confirms or disputes from their own view.

---

## 7. Screens / routes

- `/login` — Firebase Auth (Google sign-in), minimal ink background, gold CTA
- `/` — Dashboard: giant Fraunces net-balance number (jade if owed, rust if owing), list of groups as receipt-strip cards
- `/group/:id` — Group detail: activity feed (receipt-strip cards), "Simplified Settlements" panel, add-expense button
- `/group/:id/add-expense` — multi-step: amount → participants → split type (segmented control) → category
- Settle Up modal (opened from group detail) — amount + recipient UPI ID, "Pay with UPI" button, confirm/dispute state
- `/activity` — global timeline across all groups, ledger-margin dashed connector style

---

## 8. File structure to generate

```
splitfair/
  stitch-export/        // raw exported HTML/CSS/assets from Google Stitch — read-only reference, source of visual truth
  index.html
  package.json
  vite.config.js
  tailwind.config.js
  postcss.config.js
  .env.example
  src/
    main.jsx
    App.jsx
    index.css
    firebase.js
    context/AuthContext.jsx
    context/ThemeContext.jsx
    utils/upi.js
    utils/splitEngine.js
    assets/               // images copied over from stitch-export
    pages/Login.jsx
    pages/Dashboard.jsx
    pages/GroupDetail.jsx
    pages/AddExpense.jsx
    pages/Activity.jsx
    components/ReceiptCard.jsx
    components/BalanceHero.jsx
    components/SettleUpModal.jsx
    components/SplitTypeSelector.jsx
    components/GroupCard.jsx
    components/ThemeToggle.jsx
  README.md
  NOTES.md
```

`.env.example` should list:
```
VITE_FIREBASE_API_KEY=
VITE_FIREBASE_AUTH_DOMAIN=
VITE_FIREBASE_PROJECT_ID=
VITE_FIREBASE_STORAGE_BUCKET=
VITE_FIREBASE_MESSAGING_SENDER_ID=
VITE_FIREBASE_APP_ID=
```

---

## 9. Build order

1. Read every file in `splitfair/stitch-export/` first. Extract exact colors, fonts, spacing, and border-radius values used for both the dark and light themes.
2. Scaffold Vite + React + Tailwind, wire up `tailwind.config.js` and `index.html` using ONLY the values extracted from the Stitch export (fonts via Google Fonts if that's what the export uses).
3. Build `ThemeContext` + `ThemeToggle` — CSS custom properties for both palettes, `data-theme` attribute switching, `localStorage` persistence, `prefers-color-scheme` default.
4. Build `firebase.js` reading from `import.meta.env.VITE_FIREBASE_*`.
5. Build `AuthContext` + `Login` page (Google sign-in), matching the Stitch login screen exactly.
6. Build `splitEngine.js` and `upi.js` utilities with unit-testable pure functions.
7. Build `ReceiptCard` (or whatever the export's expense-row component is) and `BalanceHero` (odometer-style animated number), matching the export's visuals.
8. Build Dashboard, GroupDetail, AddExpense, SettleUpModal, Activity — wiring Firestore reads/writes as specified in the data model, using the exported markup/styling for each corresponding screen.
9. Add motion (CSS transitions/keyframes, respecting `prefers-reduced-motion`) on top of the static export.
10. Write `README.md` with setup steps (Firebase project creation, enabling Google Auth, Firestore rules, `npm install`, `npm run dev`).
11. Write `NOTES.md` listing any assumptions made, including any Stitch screens that were extrapolated rather than directly exported.

Do all of this in one pass, creating every file listed above with working code — not placeholders or TODOs, except where actual Firebase project credentials are required (those stay as env vars).