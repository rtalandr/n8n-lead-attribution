# Airtable schema

Three tables in one base.

## `Leads`

One row per lead. Written on arrival, updated after the reply attempt.

| Field | Type | Notes |
|---|---|---|
| Name | Single line | `Unnamed lead` if absent — never used in a greeting |
| Received | Date (with time) | Set by the flow, not Airtable |
| Phone | Single line | Digits and `+` only |
| Email | Email | |
| Service | Single select | Moving, Roofing, Plumbing, HVAC, Solar, Other |
| Source | Single select | Set by the attribution rules. Includes `Unknown` |
| Campaign | Single line | `utm_campaign` |
| Landing page | URL | |
| Tracked number | Single line | The number dialled, if a call |
| Click ID | Single line | `gclid` or `fbclid`. Cannot be recovered later |
| Lead cost | Number | Copied from `Sources`. Empty is meaningful — see below |
| Stage | Single select | New, Contacted, Quoted, Negotiating, Won, Lost |
| Job value | Currency | Filled when won |
| First contact | Date (with time) | When the auto-reply went out |
| Response seconds | Number | The real figure. Source of truth |
| Response minutes | Number | Derived. A sub-minute reply is `0` here — never show it alone |
| Reply channel | Single line | Email, or why nothing was sent |

**Empty `Lead cost` is not zero.** It means nobody has said what this source costs, and the
summary reports it as unknown rather than free.

## `Sources`

One row per channel. This is the only place a cost is typed.

| Field | Type | Notes |
|---|---|---|
| Source | Single line | Must match the `Source` values exactly |
| Cost per lead | Number | Leave empty if genuinely unknown |
| Cost basis | Single select | `Per lead`, `Free by design`, `Cost sits elsewhere`, `Not yet priced` |

`Cost basis` is what lets the report tell free apart from unpriced. Set organic and referral to
`Free by design` or they will be reported as missing data forever.

Costs are held per source rather than typed per lead because nobody maintains a cost field on
every row, and a field nobody maintains produces reports nobody trusts.

## `Summary`

Written by the monthly flow. Never edited by hand.

| Field | Type |
|---|---|
| Source | Single line |
| Period | Single line |
| Leads | Number |
| Won | Number |
| Close rate % | Number |
| Spend | Currency |
| Revenue | Currency |
| Cost per booked job | Currency — empty unless spend is known |
| Median response secs | Number |
| Median response mins | Number — one decimal, derived from seconds |
| Data quality | Single select |
| Note | Long text |
| Generated | Date (with time) |
