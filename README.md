# SplitsFare

A **Splitwise alternative** with direct UPI payments — built with React 18, Vite, Tailwind CSS, and Firebase.

## Features

- 🟡 **UPI Deep Links** — "Pay with UPI" button opens your UPI app (GPay/PhonePe/Paytm) with amount and note pre-filled
- ✅ **Two-sided payment confirmation** — Payer marks paid → recipient confirms or disputes
- 🧮 **Free debt simplification** — Min-cash-flow algorithm reduces group debts to minimal transactions ("Simplified Settlements")
- ⚡ **Real-time Firestore listeners** — Balances and activity feed update instantly
- 📱 **4 split types** — Equal / Exact amounts / Percentage / Shares
- 🔄 **Recurring expenses** — Weekly or monthly
- 🎨 **Dark + Light themes** — Smooth cross-fade, persisted to localStorage, defaults to OS preference

---

## Setup

### 1. Firebase project

1. Go to [Firebase Console](https://console.firebase.google.com/) → **Add project**
2. Enable **Authentication** → Sign-in providers → **Google**
3. Enable **Firestore Database** (start in test mode for development)
4. Go to **Project settings** → **Your apps** → Add a web app → Copy the config

### 2. Firestore Security Rules (recommended for production)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth.uid == uid;
    }
    match /groups/{groupId} {
      allow read, write: if request.auth.uid in resource.data.members
        || request.auth.uid == request.resource.data.createdBy;
      match /expenses/{expenseId} {
        allow read, write: if request.auth.uid in get(/databases/$(database)/documents/groups/$(groupId)).data.members;
      }
      match /settlements/{settlementId} {
        allow read, write: if request.auth.uid in get(/databases/$(database)/documents/groups/$(groupId)).data.members;
      }
    }
  }
}
```

### 3. Environment variables

Copy `.env.example` to `.env` and fill in your Firebase project values:

```bash
cp .env.example .env
```

```env
VITE_FIREBASE_API_KEY=your_api_key
VITE_FIREBASE_AUTH_DOMAIN=your_project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your_project_id
VITE_FIREBASE_STORAGE_BUCKET=your_project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=123456789
VITE_FIREBASE_APP_ID=1:123:web:abc
```

### 4. Install & run

```bash
npm install
npm run dev
```

Open [http://localhost:5173](http://localhost:5173).

### 5. Build for production

```bash
npm run build
```

Output goes to `dist/` — deploy to Firebase Hosting, Vercel, or any static host.

---

## Project Structure

```
src/
  main.jsx              # React entry
  App.jsx               # Routes
  index.css             # Design system tokens + animations
  firebase.js           # Firebase init
  context/
    AuthContext.jsx     # Google auth + user doc upsert
    ThemeContext.jsx    # Dark/light theme + localStorage
  utils/
    upi.js              # buildUpiLink()
    splitEngine.js      # computeBalances(), simplifyDebts(), split helpers
  components/
    BalanceHero.jsx     # Odometer animated net balance
    ReceiptCard.jsx     # Expense receipt strip
    GroupCard.jsx       # Dashboard group card
    SplitTypeSelector.jsx  # 2×2 split type grid
    SettleUpModal.jsx   # UPI pay + confirm/dispute flow
    BottomNav.jsx       # Tab bar
    TopBar.jsx          # Header
    ThemeToggle.jsx     # Dark/light toggle
  pages/
    Login.jsx           # Google sign-in
    Dashboard.jsx       # Net balance + group list
    GroupDetail.jsx     # Expenses + simplified settlements
    AddExpense.jsx      # 4-step expense creation
    Activity.jsx        # Global activity timeline
```
