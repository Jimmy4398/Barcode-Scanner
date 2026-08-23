# Operation Elf Scanner

A mobile-friendly barcode scanning app built for **Operation Elf**, a holiday charity program that collects and distributes gifts for children in need. Volunteers use it to scan barcoded gift tags and update tracking information in real time, across four role-specific views.

This repo hosts the **frontend only** (`scanner.html`, served via GitHub Pages). The backend — a Google Apps Script project called `Code.gs` — lives in a separate Google Apps Script project attached to the organization's Google Sheet, and is not part of this repo.

## Why two separate pieces?

Google Apps Script's own web pages run inside a sandboxed iframe that browsers block camera access from. So the actual camera-scanning UI has to live somewhere else entirely (this repo, via GitHub Pages), and talks to the Apps Script backend over the network for anything involving the Google Sheet. `scanner.html` never touches the Sheet directly.

```
 Phone browser  ──sign in──▶  Google
 (this repo)    ◀─token────

 Phone browser  ──API call──▶  Apps Script  ──reads/writes──▶  Google Sheet
 (this repo)    ◀───result───  (Code.gs, separate project)     (Wish List + Team tabs)
```

## The four views

| View | Who uses it | What it does |
|---|---|---|
| **Site Host Elf** | Site coordinators | Scan an item, review its details, update Status / Notes / Completeness / Missing items |
| **Packing Day Elf** | Packing day volunteers | Scan an item, see a color-coded destination table, confirm it's headed the right place |
| **Fairy Godmother** | Gift-completion volunteers | Scan an item, review what's needed, mark the gift complete, see which table to send it to |
| **Santa's Workshop** | Admins only | A live read-only dashboard — no scanning — for tracking progress across the whole event |

Access to each view is controlled by role (`admin`, `sitehost`, `volunteer`), managed entirely from a `Team` tab in the Google Sheet — no code changes needed to add or remove someone.

## Access control

Real Google Sign-In, not a shared password. Every request carries a signed identity token that the backend independently verifies against Google's own servers and checks against the roster — enforced server-side on every single action, not just hidden in the UI.

## Notable design decisions (read this before diving into the code)

- **View names in the code don't always match what's on screen.** Internally, the "Fairy Godmother" view is still called `guiding` throughout the code — a leftover from an earlier name (originally "Guiding Spirit") that never got renamed at the code level. This is intentional, not a bug.
- **Packing Day's color check only applies to 4 of its 8 stations** (Yellow/Blue/Pink/Purple Table). The other 4 (Check-in, Fairy Godmother, Packing, Truck) never block the Accept button — and Truck specifically skips the server lookup entirely, not just the color comparison, as a deliberate speed optimization.
- **Every network call has a 12-second timeout and exactly one automatic retry.** If something still fails after that, the user sees an explicit error rather than a silent hang.
- **A "Season" toggle in the Team tab** can pause Packing Day Elf and Fairy Godmother entirely (showing a friendly banner instead), without touching any code — useful for the gap between events.
- **Sensitive columns (Email, Phone, Sponsor) are never sent to the browser** for any role — the backend filters exactly which fields each view is allowed to receive, not just which fields the UI happens to display.

## Deployment

1. This file (`scanner.html`) is hosted via GitHub Pages directly from this repo.
2. It expects a corresponding Apps Script backend deployed separately, with its Web App URL hardcoded near the top of the `<script>` block (`API_URL`).
3. Any change to `scanner.html` just needs a normal commit to this repo — GitHub Pages picks it up automatically, no separate build step.
4. Backend changes (`Code.gs`) are deployed independently, from within the Apps Script editor (Deploy → Manage deployments → New version).

## A note for anyone new to this codebase

There's a large orientation comment block at the very top of the `<script>` section in `scanner.html` covering the trickier parts of how the app actually behaves (the scan/accept/decline flow, the network retry logic, the season-gate re-check logic, and more) in more depth than this README. Start there if you're about to make a change.
