---
name: bryntum-trial-license
description: >
  Request and register a free 45-day Bryntum 8.x trial license key (Core or Enterprise) using the
  Bryntum MCP `request_bryntum_trial_key` tool or bryntum.com/download. Use alongside the `bryntum`
  skill whenever setting up an 8.x Core or Enterprise app without a license key, when the user asks
  for a trial key, or when troubleshooting a Trial badge or masked component. Not for Community
  tier or 7.x apps (neither uses a license key).
metadata:
  tags: bryntum, trial, license, licenseKey, mcp, 8.x
---

## When to use (and when to stop)

Run the `bryntum` skill's Version check first (`https://raw.githubusercontent.com/bryntum/skills/refs/heads/main/bryntum/SKILL.md`). This skill applies to **Bryntum 8.x Core and Enterprise** apps only:

- **Community tier (8.x):** free, MIT, takes no key — stop and do not request or set one.
- **7.x or older (untiered packages):** takes no `bryntum.licenseKey` (trial vs licensed is a package choice per the Version check's `v7.md`) — stop and never add a key.
- **User already has a key:** skip requesting a trial key and go straight to **Registering the key**.

---

## Requesting a trial key

Core and Enterprise packages run without a key as a **14-day keyless trial** per product in the browser (Trial badge + console message; masked after 14 days). A free trial key covers every Bryntum product at Core/Enterprise and extends the trial to **45 days**.

### With Bryntum MCP (`request_bryntum_trial_key`)

When setting up an 8.x Core or Enterprise app with no key yet, **proactively get a trial key**:

1. Tell the user you're getting them a free 45-day Bryntum trial license key (emailed to their inbox, never returned by the tool) — or they can paste an existing key or skip to stay on the 14-day keyless trial.
2. Ask for their **full name**, **company**, **work email**, and whether they want **product updates and marketing emails from Bryntum and AG Grid** (`marketingConsent: true` only on explicit opt-in; default `false`). **Never invent, guess, or reuse placeholder details.**
3. Call `request_bryntum_trial_key` with those details and `products` limited to `scheduler`, `gantt`, `calendar`, `taskboard`, or `grid` (Scheduler Pro is `scheduler`; omit `products` to default to all five, e.g. for Chart or widgets-only Core-package apps — the key covers every product either way).
4. Once sent, ask the user to paste the key from their email (check spam if delayed, or contact `https://bryntum.com/contact/` if it never arrives), then register it. Setup can continue on the 14-day keyless trial meanwhile.
5. If the tool reports an error or refusal, follow its reply: on a rate limit, ask the user to check for an earlier Bryntum email (including spam) or try tomorrow; if rejected, verify their real details; after any network/timeout failure, have them check inbox/spam before retrying since the email may still have gone out — don't retry on your own.

### Without MCP (or if the user skips)

Link `https://bryntum.com/download/`, leave `// TODO: bryntum.licenseKey = '...'` in the entry file, and let the 14-day keyless trial run.

---

## Registering the key

Follow the `bryntum` skill's **License key** section — set `bryntum.licenseKey` once before the first Bryntum component is created (entry file after Bryntum imports, or inline `<script>` before the bundle loads). Never reassign `window.bryntum = ...` after a Bryntum import (Core installs an accessor on it) and never augment `Window`:

```typescript
// Works whether Bryntum is imported eagerly or lazy-loaded:
((globalThis as any).bryntum ??= {}).licenseKey = 'your-license-key';
```

**Never invent, guess, or "test" a key.** In public repos, keep the key out of committed source via a gitignored local env var (e.g. `const key = import.meta.env.VITE_BRYNTUM_LICENSE_KEY; if (key) ...` — note that browser bundles still expose client-side keys).

---

## Verifying license state

For each top-level licensed product widget the app renders (`Grid`, `Scheduler`, `Gantt`, `Calendar`, `TaskBoard`), check both its badge and the browser console (nested widgets and standalone Core/Chart widgets don't run badge checks):

- **Trial badge + `Trial license ends <date>.`:** the 45-day trial key is registered. Expected — only a purchased key removes the Trial badge.
- **Trial badge + `No license key registered…` or `Bryntum license key was ignored…`:** the key isn't applied or was mis-pasted. Fix the registration; don't request another key.
- **No badge on a working rendered licensed product widget:** valid purchased key (or Community tier).
- **Masked component:** the trial ended, or the key doesn't cover that product or tier (each product on the page needs coverage). If the 14-day keyless trial expired, request a 45-day trial key above; otherwise report the console message and point to `https://bryntum.com/contact/`.

Report the console state; never hide the badge or work around the mask.
