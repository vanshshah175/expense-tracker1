# PennyWise – Multi-Account Expense Tracker

A single-page expense tracker with multiple accounts, income/expense tracking, and a Firebase backend (Authentication + Cloud Firestore) so your data syncs across devices.

## Features

- **Sign in** with email/password or Google
- **Multiple accounts** (wallet, bank, credit card, savings, cash), each with its own balance
- **Income and expense tracking** with categories
  - Expenses: Food, Drink, Travel, Utilities, Milk, Recharge, Petrol, Medicine, Medical, Daily Needs
  - Income: Salary, Investment
- **Custom dates** – back-date entries to record previous months
- **Month filter** – view totals and history for any month or all time
- **Realtime sync** across browsers and devices
- **Offline support** – changes made offline sync when you reconnect
- **Per-user privacy** – Firestore rules restrict each user to their own data

## Project files

| File | Purpose |
|------|---------|
| `index.html` | The whole app (HTML, CSS, JavaScript) |
| `firestore.rules` | Firestore security rules |
| `README.md` | This file |

## Setup

### 1. Firebase project

1. Go to the [Firebase Console](https://console.firebase.google.com) and create a project.
2. Add a **Web app** (the `</>` icon) and copy its `firebaseConfig`.
3. Paste the config into the `firebaseConfig` object near the top of the `<script type="module">` block in `index.html`.

### 2. Authentication

In **Build → Authentication → Sign-in method**, enable:

- Email/Password
- Google

### 3. Firestore database

1. Go to **Build → Firestore Database → Create database**.
2. Keep the database ID as `(default)`.
3. Choose a nearby location (e.g. `asia-south1` Mumbai). This cannot be changed later.
4. Start in **production mode**.
5. Open the **Rules** tab, paste the contents of `firestore.rules`, and click **Publish**.

No collections need to be created by hand; the app creates them on first use.

## Running locally

Google sign-in does not work when the file is opened directly (`file://`), so serve it over HTTP:

```bash
npx serve
```

Then open the address it prints (usually `http://localhost:3000`).

Make sure `localhost` is listed under **Authentication → Settings → Authorized domains** (it is by default).

## Deploying (optional)

Firebase Hosting is free for small apps:

```bash
npm install -g firebase-tools
firebase login
firebase init hosting     # use "." as the public directory, single-page app: No
firebase deploy
```

After deploying, your hosting domain is added to Authorized domains automatically.

## Data model

All data lives under the signed-in user's ID:

```
users/{uid}/accounts/{accountId}
    name, icon, balance, createdAt

users/{uid}/transactions/{transactionId}
    accountId, description, amount, category, type ("income" | "expense"), date (ISO string)
```

Adding or deleting a transaction updates the account balance in the same atomic batch write, so balances stay correct even when used from several devices.

## Security rules

```
match /users/{userId}/{document=**} {
  allow read, write: if request.auth != null && request.auth.uid == userId;
}
```

The Firebase web config in `index.html` is not secret – it only identifies your project. Your data is protected by the security rules above, so always keep them published.

## Notes

- The **Current Balance** of an account changes with every transaction, including back-dated ones. If you enter old entries, set the account's starting balance to what it was at the start of that period.
- Deleting an account also deletes all of its transactions.
- Currency is displayed in USD. To change it, edit the `formatCurrency` function in `index.html` (e.g. `'en-IN'` with `currency: 'INR'` for rupees).
- Data from the older localStorage-only version is not migrated automatically.

## Tech stack

- Vanilla HTML, CSS, JavaScript (no build step)
- Firebase JS SDK v10 (Authentication, Cloud Firestore) loaded from the CDN
- Font Awesome icons, Inter font
