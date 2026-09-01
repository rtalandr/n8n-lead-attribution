# Lead attribution and first reply

Two n8n workflows and an Airtable schema that answer one question a lot of small
service businesses cannot answer about themselves:

> **Which lead sources actually produce booked jobs, and what does each booked job cost?**

A lead arrives by webhook. Within a few seconds it has been attributed to a source, priced
against what that source costs, checked for junk, replied to by email, and recorded with the
response time. On the 1st of every month a second workflow turns all of that into a
cost-per-booked-job table.

This is the working system, exported and sanitised. It is not a tutorial and not a toy.

---

## Why this exists

Most lead-tracking setups fail in the same three ways. All three are design problems, not
bugs, and this repo is mostly about how they are avoided.

**1. They guess.** When the source is unclear, they attribute it to something plausible.
A wrong source is worse than an honest gap — it moves real money toward the wrong channel.
Here, anything unmatched is recorded as `Unknown` and stays that way.

**2. They confuse "free" with "we never entered a cost".** Both come out as $0 spend, which
makes the unpriced source look like the best performer on the page. This is the most expensive
reporting mistake in the category and almost nobody guards against it. Every row here states
which of the two it is, and cost per booked job is only ever divided when the spend behind it
is real.

**3. They reply slowly, or reply to junk fast.** Speed is the single biggest lever on lead
conversion, so the reply must go out in seconds — but replying instantly to every bot
submission burns sender reputation. The fix is validation, not verification: check the data
itself in milliseconds instead of asking the person to prove anything.

---

## What's here

```
workflows/lead-intake.json      12 nodes — webhook to email reply
workflows/source-summary.json    6 nodes — scheduled cost-per-booked-job report
docs/attribution-rules.md        the decision order, in full
docs/airtable-schema.md          the three tables and their fields
docs/gotchas.md                  things that cost real time to work out
```

Credentials are stored by reference in n8n, so no secrets ever existed in these exports.
Base and table IDs are replaced with placeholders.

---

## How a source is decided

First match wins, and the order matters. Cheap, certain signals are checked before
inferred ones.

| # | Signal | Result |
|---|---|---|
| 1 | Tracked phone number was dialled | `Phone` |
| 2 | `gclid` present | `Google Ads` — click ID stored |
| 3 | `fbclid` present | `Facebook Ads` — click ID stored |
| 4 | `utm_source` present | mapped, but see below |
| 5 | Referrer header | Angi, Thumbtack, Google LSA, Facebook, own domain, else `Referral` |
| 6 | Contact details but nothing else | `Website form` |
| 7 | none of the above | `Unknown` |

Two details in there are easy to get wrong:

**`utm_source=google` is not Google Ads.** It is only credited as paid when `utm_medium`
contains `cpc`, `paid` or `ppc`. Otherwise it is organic traffic that happened to carry a UTM,
and counting it as paid inflates the cost of a channel that cost nothing.

**Click IDs must be captured at the moment of the click.** `gclid` and `fbclid` are what make
offline conversion import possible later — telling Google or Meta that a specific click became
a real booked job. They cannot be reconstructed afterwards. If the form is not capturing them
today, that data is gone for every lead until it does.

---

## Honest reporting

Every row of the summary carries a `Data quality` value. This is the part worth stealing.

| Value | Meaning |
|---|---|
| `Complete` | Every lead priced. The number is real. |
| `Partly priced` | Some leads priced. Spend is a floor, not a total. |
| `Spend unknown` | No cost entered. **Spend is unknown, not zero.** |
| `Free source` | Genuinely free by nature — organic, referral. |
| `Cost counted elsewhere` | Real cost, but it belongs to the originating channel. |

Cost per booked job is computed only when spend is known, spend is above zero, and at least
one job was won. Otherwise the cell stays empty rather than showing a confident number derived
from a gap.

A report that admits what it does not know is worth more than one that doesn't, because the
first one can be acted on.

---

## Validation, not verification

Six checks run on arrival. All are instant, none ask anything of the person, and together they
decide whether the auto-reply is sent. The lead is always saved either way — a rejected lead
still gets recorded, along with the reason.

| Check | Catches |
|---|---|
| Honeypot field filled | Bots, which fill every field they find |
| Submitted under 2 seconds after page load | Automated submission |
| No usable email or phone | Junk, and anything we couldn't answer anyway |
| Email fails format check | Typos and fakes |
| Disposable email domain | Throwaway inboxes |
| Same person within 30 minutes | Double submissions — don't message twice |

Asking someone to confirm their email before replying would be more certain and would destroy
the thing being sold. Sub-minute response only exists if nothing blocks on a human.

---

## Setting it up

1. Create the three Airtable tables in `docs/airtable-schema.md`.
2. Import both workflows into n8n. Reconnect the Airtable and HTTP credentials — the IDs are
   blanked, the names are kept so you can see what each one needs.
3. Replace the placeholder base and table IDs (`appXXXX…`, `tblLEADS…`) with your own.
4. Fill the `Sources` table with your channels and what each costs per lead. Set `Cost basis`
   to `Free by design` for organic and referral, so they aren't reported as unpriced.
5. Point your form at the `lead` webhook. Send `gclid`, `fbclid`, the UTM parameters, the
   referrer, and a hidden `form_loaded_at` timestamp along with the usual fields.
6. Add a hidden honeypot field named `website` to the form. Real people never fill it in.

Sending uses Resend over plain HTTP, so any provider with a REST API drops in.

---

## Known limitations

Stated rather than hidden, because a repo that claims to be about honest reporting should be.

- **Last-click only.** The source that produced the form fill gets the credit. No multi-touch
  modelling — deliberately, because it needs traffic volume most small businesses don't have.
- **Offline conversion import is not wired up.** The click IDs are captured, which is the part
  that cannot be added later. Sending them back to Google or Meta is not in these workflows.
- **90-day window is hard-coded** in the summary.
- **Job value is entered by hand**, or comes from whatever CRM writes to the table.

## Licence

MIT. Use it, change it, ship it to your clients.
