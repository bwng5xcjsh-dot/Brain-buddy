# Tally — Tasks & Rewards

A household task app for **all ages — kids and adults**. Create task lists, snap a photo
when a task is finished, and **AI (Claude vision) checks the photo** against what "done"
should look like. Verified tasks automatically credit rewards: **money, points, and
screen-time minutes**. The design is a modern, neutral product UI (with automatic
dark mode) that works equally well for an adult's errand list and a kid's chore chart.

Works on **desktop, iPhone, and Android** — it's a Progressive Web App (PWA), so it runs
in any browser and can be installed to the home screen like a native app.

---

## ✨ What it does today

| Feature | Status |
|---|---|
| Household + family member profiles (parent / adult / kid) | ✅ Working |
| Chore lists with icons, recurrence (daily / weekly / one-time), per-chore rewards | ✅ Working |
| Photo proof: take a photo right from the app | ✅ Working |
| **AI verification**: Claude looks at the photo, compares it to the chore's "what done looks like" criteria, and passes/fails it with friendly feedback | ✅ Working (needs an Anthropic API key) |
| Auto-credit rewards on verified completion | ✅ Working |
| Parent review queue (fallback when AI is unsure, the photo is skipped, or no API key is set) | ✅ Working |
| Rewards ledger per member: money, points, screen minutes; redeem/cash-out flow | ✅ Working |
| Parent PIN lock on the parent tab | ✅ Working |
| Offline support + installable on iOS/Android/desktop | ✅ Working |
| Automatic screen-time granting on iPad / Android | 🔜 Roadmap (see below) |
| Automatic money transfer to kids' accounts | 🔜 Roadmap (see below) |
| Multi-device sync (one household across everyone's phones) | 🔜 Roadmap |

---

## 🚀 Deploy it (free, ~2 minutes)

The app is 100% static — no server needed. Host it with **GitHub Pages**:

1. In this repo on GitHub, go to **Settings → Pages**.
2. Under **Build and deployment**, set **Source** to "Deploy from a branch",
   pick your branch (e.g. `main`) and folder `/ (root)`, then **Save**.
3. After a minute your app is live at
   `https://<your-username>.github.io/Brain-buddy/`.

Open that URL on any device:

- **iPhone/iPad**: Safari → Share → **Add to Home Screen**
- **Android**: Chrome → menu → **Add to Home screen** (or the "Install app" prompt)
- **Desktop**: Chrome/Edge show an install icon in the address bar

> HTTPS (which GitHub Pages provides) is required for the camera and the
> service worker to work.

### Turn on AI verification

1. Get an API key at [console.anthropic.com](https://console.anthropic.com) → API keys.
2. In the app: **Parent tab → Settings → Anthropic API key** → Save.

The key is stored **only in that device's browser storage** and is sent only to
`api.anthropic.com`. Each photo check is one small vision request — with the default
Claude Opus 5 model a check costs a few cents; the settings let you switch to
Sonnet 5 or Haiku 4.5 to make checks cheaper. Without a key the app still works —
completions just go to the parent review queue instead.

---

## 🏗️ How it works

```
┌────────────────────────────── Any browser (PWA) ─────────────────────────────┐
│                                                                              │
│  Chore lists ── member picks chore ──► 📸 camera ──► resize in-browser       │
│                                                        │                     │
│                                                        ▼                     │
│                                          Claude API (vision request)         │
│                                          "Does this photo show the chore     │
│                                           done, per these criteria?"        │
│                                                        │                     │
│                              ┌─────────────────────────┼──────────────────┐  │
│                              ▼                         ▼                  ▼  │
│                     ✅ verified (confident)     🤔 unsure / no key   ❌ not  │
│                     auto-credit rewards        parent review queue   done — │
│                     (money/points/minutes)                          retry   │
│                                                                              │
│  Local storage: household, chores, ledger, photo thumbnails (per device)     │
└──────────────────────────────────────────────────────────────────────────────┘
```

Key design choices:

- **No backend (v1).** All data lives in the browser's local storage; the only network
  call is the direct Claude API request. This makes hosting free and private —
  but data is per-device (sync is the top roadmap item).
- **AI is advisory, parents are authoritative.** The AI auto-approves only when it's
  confident (≥60%). Anything uncertain lands in the parent review queue, and parents
  can always approve/reject manually.
- **"What done looks like" per chore.** Each chore stores verification criteria written
  by the parent (e.g. "blanket flat, pillows at top, floor clear"), which becomes part
  of the AI prompt — this is what makes verification meaningfully accurate.

---

## 🔌 Roadmap: real-world integrations

You asked for the app to *connect to accounts and actually grant screen time / money*.
Here's the honest engineering picture and the plan:

### 1. Screen time (iPad / iPhone)
Apple does **not** expose Screen Time to websites or third-party servers. The only
sanctioned path is the **Family Controls / ManagedSettings framework**, which requires:
- a **native iOS companion app** (Swift), and
- Apple's Family Controls **entitlement** (you apply; approval required), and
- the app installed on the child's device with parent authorization.

**Plan:** Phase 3 ships a small iOS companion app whose only job is to read the child's
earned minutes from Tally and extend the daily limit accordingly. On Android, the
equivalent is a companion app using device-admin / Digital Wellbeing-style APIs, or
integrating with parental-control platforms that offer partner APIs.

**Today:** the app tracks earned/redeemed screen minutes, and the parent applies them
in Screen Time settings (takes ~10 seconds) — the ledger is the source of truth.

### 2. Money to kids' accounts
Consumer banks and kids' debit cards (Greenlight, GoHenry, Step) don't offer open
public APIs for third-party deposits. Realistic options, in order of effort:
- **Ledger + manual payout** (today): the app tracks balances; parents cash out.
- **Payment links** (Phase 2): generate a Venmo/PayPal/Apple Cash deep link pre-filled
  with the earned amount, so payout is one tap for the parent.
- **Fintech partner** (Phase 4, if this becomes a product): partner with a
  banking-as-a-service provider (e.g. Unit, Treasury Prime) to hold real FDIC-insured
  kid accounts — this involves KYC and compliance and is a business decision, not
  just code.

### 3. Multi-device household sync (Phase 2 — next up)
Add a small backend (Firebase or Supabase both have generous free tiers):
- one household document shared by all family members' devices,
- parent/kid sign-in,
- the Claude API call moves server-side so the API key is never in the browser,
- push notifications ("Sam finished 'Feed the dog' — review it!").

### Suggested phases

| Phase | Deliverable |
|---|---|
| **1 (this repo, done)** | Installable PWA: chores, AI photo verification, rewards ledger, parent review |
| **2** | Supabase/Firebase sync + accounts + server-side AI proxy + payout deep links |
| **3** | Native iOS companion (Family Controls) for automatic screen time; Android equivalent |
| **4** | Real money accounts via a banking-as-a-service partner; app-store distribution |

---

## 🗂️ Files

- `index.html` — the entire app (vanilla JS, no build step)
- `manifest.webmanifest`, `sw.js`, `icon.svg` — PWA install + offline support
- `Index.html` — the original "Brain Buddy" behavioral-training prototype (kept as-is)

## 🔒 Privacy notes

- Chore data, balances, and photo thumbnails stay in the device's browser storage.
- Photos are sent only to the Claude API for verification, only when a key is configured.
- The API key is entered by a parent, stored locally, and used only for `api.anthropic.com`.
