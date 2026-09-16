# AV Room Desk

A lending desk for the college AV room. It tracks every unit of gear, answers "is a DSLR free this weekend?" in one line, enforces the borrowing limit, and works out the late fee and deposit refund when kit comes back.

Built for Round 2 (Builder Round), problem statement `equipment_rental`.

---

## What it does

| Area | What's covered |
| --- | --- |
| Availability | Ask about any item over any date range and get a plain-language answer, including which units are free and, when nothing is, the earliest day one frees up and who is holding it. |
| Borrowing | Hand out a specific numbered unit to a named borrower, with pick-up date, loan length, purpose and phone number. |
| Returns | One click computes days late, the late fee, and exactly how much deposit goes back. Condition and a note are recorded. |
| Deposits | Held per item type. Refund = deposit − late fee, floored at zero. |
| Late fees | Per-day rate per item type, with an optional grace period. |
| Limits | A person may hold at most *N* items (default 2) counting reservations, and never two of the same item. |
| Reservations | Booking a unit for future dates takes it off the shelf for those dates, so two clubs can't be promised one projector. |
| Nudges | Generates a ready-to-send reminder or overdue chase message with the running fee, and copies it to the clipboard. |
| Register | Closed loans with fee and refund, a running activity log, and a CSV export. |

---

## Running it

There is no build step, no package install and no backend. The whole app is one file.

### Fastest way

Open `index.html` in any modern browser. Done.

### In GitHub Codespaces

```bash
python3 -m http.server 8000
```

Then open the forwarded port 8000 in the Ports tab and visit `/index.html`.

Or install the *Live Server* extension and click **Go Live**.

### Requirements

A modern browser (Chrome, Edge, Firefox or Safari). Nothing else — no Node, no database, no API key.

---

## Where the data lives

State is kept in the browser's `localStorage` under the key `avroom.v1`. It survives refreshes and browser restarts on the same machine and browser profile.

On first load the app seeds itself with realistic demo data: 8 item types, 26 units, three live loans (one already overdue) and one weekend reservation, so nothing is empty when you open it.

**Reset to demo data** in the left rail wipes the register and reseeds it.

---

## How to use it

1. **Today at the desk** — pick an item and a date range (or hit the *This weekend* chip). The answer bar tells you yes or no and why. Below it, everything overdue, with a *Write a nudge* button.
2. **Gear shelf** — every unit and its current state. Hover a tag to see who has it and when it's due.
3. **Hand out gear** — choose item, dates and unit, fill in the borrower, and the desk runs its checks before enabling the button. If anything blocks the loan it says so in plain words.
4. **Take it back** — pick the loan, confirm condition, and the receipt shows the deposit arithmetic.
5. **Reservations** — future bookings, ready to convert into a loan on the pick-up day.
6. **Register** — closed loans, fees collected, activity log, CSV export.
7. **Desk rules** — change the per-person limit, grace days, default loan length, and each item's deposit, late fee and maximum loan length. Every change takes effect immediately on the hand-out screen.

---

## Debugging

| Symptom | What to check |
| --- | --- |
| Data doesn't persist | `localStorage` is blocked — private/incognito windows and some strict privacy settings disable it. The app still runs; it just forgets on reload. |
| Nothing shows on load | Open DevTools → Console. If `avroom.v1` holds malformed JSON, the app falls back to demo data automatically; if it doesn't, run `localStorage.removeItem('avroom.v1')` and refresh. |
| Want to inspect state | `JSON.parse(localStorage.getItem('avroom.v1'))` in the console. |
| Dates look off by one | All dates are local `YYYY-MM-DD` strings, never UTC timestamps, so they don't drift across timezones. Check your system date first. |
| Fonts look plain | The Google Fonts link needs network access. The app falls back to a system sans stack offline and stays fully usable. |
| An overdue item won't free up | By design — an overdue loan keeps blocking its unit until it is actually returned, not until its due date. |

---

## Project layout

```
.
├── index.html      the entire application: markup, styles, logic
├── README.md       this file
├── REASONING.md    why it is built this way
└── AI_LOGS.md      the unedited AI conversation
```

---

## Tech stack

Plain HTML, CSS and JavaScript. No framework, no bundler, no dependencies.

That was a deliberate choice: in a 2.5-hour window, a zero-install single file means the evaluator opens one file and the thing works, with no version mismatch, no `npm install`, and no chance of a broken build eating the demo. The logic that matters here — overlap detection, limit checks, fee arithmetic — is the same in any stack.
