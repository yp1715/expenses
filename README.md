# Property Ledger — self-hosted (Supabase + GitHub Pages)

A shared, password-protected rental-expense ledger for your family. One web page,
one shared password, live sync between everyone. There are no individual accounts —
just the single family password to get in.

This is the **hardened** build: the password both (a) signs you into the database and
(b) decrypts the data. Without it, the ledger can't be read *or* written — and even
the database only ever stores AES-encrypted ciphertext.

---

## What you need
- A free Supabase account (supabase.com)
- A GitHub account (free hosting via GitHub Pages)

Setup is about 10 minutes.

---

## 1. Create the Supabase project
1. supabase.com → **New project**. Pick a name and a region near the UK
   (e.g. *London / eu-west*). Wait for it to finish.
2. Sidebar → **SQL Editor → New query**, paste all of `schema.sql`, click **Run**.
   This creates the tables, locks them to authenticated access, and enables live sync.

## 2. Create the one shared login
1. Sidebar → **Authentication → Users → Add user → Create new user**.
2. Enter an email and a password. This password **is** your family password.
   - Use the same email you'll put in `LEDGER_EMAIL` below (default: `ledger@yourfamily.app` — a real inbox isn't required).
   - Turn **Auto Confirm User ON** so no email verification is needed.
3. That's the only account. Everyone shares it.

## 3. Get your keys
**Project Settings → API**: copy the **Project URL** and the **anon public** key.

## 4. Put your values into config.js
Open `config.js` and fill in the three values:

```js
window.LEDGER_CONFIG = {
  SUPABASE_URL:      "https://abcd1234.supabase.co",
  SUPABASE_ANON_KEY: "your-anon-public-key",
  LEDGER_EMAIL:      "ledger@yourfamily.app"   // must match the user you created
};
```

Your keys live **only** in `config.js`. When the app is updated later you replace
only `index.html` — `config.js` and your keys stay exactly as they are.

## 5. Host on GitHub Pages
1. New GitHub repo (e.g. `property-ledger`), upload **both** `index.html` and `config.js`.
2. **Settings → Pages → Source: Deploy from a branch**, `main` / `/ (root)`, **Save**.
3. After a minute it's live at `https://<your-username>.github.io/property-ledger/`.

## 6. First run
- Open the site, enter the family password (the one from step 2).
- Share the **site link** with your siblings, and tell them the **password
  separately** (in person / call / shared password-manager note — not in the same
  message as the link).
- Everyone who signs in sees the same live ledger.

---

## Moving your existing data over
1. In the Claude version: **Expenses → Export / print → Download backup** (a `.json`).
2. On the new site, sign in, then **Expenses → Export / print → Restore from backup…**
   and pick that file. Everything is re-encrypted and uploaded.

---

## How the security works
- **One password, two locks.** What you type is used to sign in to the shared Supabase
  account *and* to derive the AES-256 key that decrypts the data. The password lives
  only in memory during your visit — it's never written into the page or stored.
- **No access without it.** Row Level Security requires an authenticated session, so
  someone who finds your site URL and the public anon key still can't read or write
  anything — they don't have the password to sign in.
- **Encrypted at rest too.** Even with database access, every row is ciphertext. The
  password can't be recovered; see below if it's lost.
- **Lock = sign out.** "Lock now" ends the Supabase session, not just the screen.

## Changing the password
Use **Security (padlock icon) → Change password** while signed in. It rotates the
shared login *and* re-encrypts all data in one step — do it this way, not from the
dashboard, so both layers stay in sync.

## If the password is forgotten
There's no recovery (that's the point of the encryption). To start over: in Supabase,
set a new password on the shared user under **Authentication → Users**, then delete all
rows in the **properties, expenses, receipts** and **meta** tables, and sign in fresh.

## Backups
**Export / print → Download backup** gives an offline copy you can restore. A backup
file is **not** encrypted — store it somewhere safe.

## If live sync isn't working
Re-run `schema.sql` (it adds the tables to the `supabase_realtime` publication), or
check **Database → Replication**. The **sync button** in the top bar is a manual
refresh fallback.

## Note on receipts
Receipts are stored as encrypted text rows. Images are auto-compressed; PDFs are
capped around 3 MB. If you need large PDF receipts later, the upgrade is a Supabase
Storage bucket — ask and I'll wire it up.
