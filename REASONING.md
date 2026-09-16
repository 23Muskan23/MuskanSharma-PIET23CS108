# Reasoning

## Reading the problem

The statement is not really a request for an inventory table. It describes a paper register failing at four specific jobs, and each failure points straight at a feature:

| What goes wrong on paper | What that means the software must do |
| --- | --- |
| "Two clubs show up for the same projector" | Units must be individually identified and exclusively held. Counting "3 DSLRs" is not enough — you have to know *which* one is out. |
| "Is a DSLR free this weekend?" and nobody can say | Availability must be answerable over a **future date range**, not just "right now". This is the thing a paper register genuinely cannot do. |
| "Kit goes missing" | Every unit needs a state, a named holder and an audit trail. |
| "Borrowers hang on to things far too long" | Due dates, visible overdue standing, a per-day fee, and an outbound nudge. |

So I treated **availability over a date range** as the centre of the product, not a side feature. It is the first thing on the first screen, phrased as the question people actually ask, and answered in a full sentence rather than a number in a cell.

The brief's own ordering — "get borrowing, availability and returns solid first, then deposits and limits" — set the build order, and I stuck to it.

## The one modelling decision everything else rests on

**A unit, not an item, is the thing that gets lent.**

An item type (`Canon EOS 200D kit`) owns numbered units (`DSLR-01`, `DSLR-02`, `DSLR-03`). Loans and reservations point at a unit.

This costs a little more code than a quantity counter, and it buys:

- genuine double-booking prevention rather than an inventory count that can go negative;
- per-unit history, so a body that keeps coming back damaged is visible;
- a tag you can physically write on the case, which is how a real AV room already works.

Availability then reduces to one honest question: *for this date range, which units have no overlapping loan and no overlapping reservation?*

```
overlap(a1,a2,b1,b2)  ⇔  a1 ≤ b2 ∧ b1 ≤ a2
```

Everything else — the free count, the "all out" verdict, the earliest free day, the unit dropdown on the hand-out form — is that one predicate applied differently.

## Decisions worth defending

**An overdue loan blocks its unit until it actually comes back, not until its due date.** A due date is a promise, not an event. If DSLR-01 was due Tuesday and it's Friday, the shelf does not have a DSLR-01 on it, and the availability answer must not pretend otherwise. This is exactly the lie the paper register tells.

**Reservations count towards the per-person limit.** Otherwise the limit is trivially defeated: book three things for tomorrow, collect them tomorrow, hold four. The rule is about one person monopolising the room, so it has to count claims, not just physical possession.

**One unit of a given item per person.** Implied by "one person shouldn't be able to book out half the room at once" — a limit of 2 that lets you take two of the only two projectors doesn't solve the stated problem.

**The refund is shown before the return is confirmed.** Deposit, days late, fee and the final number are laid out as a receipt on the confirm dialog. The desk volunteer handing cash back should never have to do arithmetic under pressure, and the borrower should see the same figure the volunteer sees.

**When the answer is "no", say why and say when.** A bare "unavailable" sends the student back to the queue tomorrow. So an unavailable answer carries the earliest free date and who is holding the units. That is the difference between an inventory screen and a service desk.

**The nudge is drafted, not sent.** The app has no mail or SMS backend and I would rather not fake one. It writes a message with the correct name, unit, due date, running fee and deposit at stake, and copies it to the clipboard for WhatsApp or the club group. The human decides the tone and presses send; the app removes the excuse of not knowing the numbers.

**Charges are editable, not hard-coded.** Deposit, late fee and maximum loan length sit per item type on the Desk rules screen, along with the per-person limit and a grace period. A different lending desk — a sports store, a lab, a library of things — reconfigures rather than forks. The brief asked for this explicitly: "for any lending desk and not just this one".

## Interface choices

Design follows the counter it's meant to sit on. The left rail is named for tasks a volunteer actually does (*Hand out gear*, *Take it back*) rather than CRUD nouns. Unit tags are set in monospace because they are codes written on hardware. Green / amber / red for free, out and overdue is carried by a state word as well as colour, so it survives colour blindness and a bad projector.

The dashboard leads with the question, not with statistics. Counts of items out and fees running are useful, but they are not what the person standing at the counter is asking.

## What I deliberately left out

- **Accounts and roles.** A shared desk terminal is the real deployment. Login would eat build time and add nothing to the graded problem.
- **A server.** State in `localStorage` keeps the whole thing to one file with zero install. The domain logic is isolated in pure functions over plain data, so swapping the persistence layer for an API is a contained change.
- **Email or SMS delivery.** See above — drafted, not faked.
- **Damage charges beyond the deposit.** Condition is recorded on return and flagged in the register, but I did not invent a repair-cost workflow the brief never mentioned.

## If I had another day

Per-unit maintenance status so a damaged body leaves the availability pool; a waiting list that offers a unit to the next person the moment it returns; a repeat-offender signal on the hand-out screen (the data is already there — history is shown, it just doesn't gate anything yet); and a printable QR code per unit so a return is a scan rather than a search.
