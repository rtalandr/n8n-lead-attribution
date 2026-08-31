# Attribution rules

Last-click. First match wins. Nothing is guessed.

The order below is deliberate: signals that are *certain* are checked before signals that are
*inferred*, and inferred signals before signals that are merely *consistent with* a source.

## The order

### 1. Tracked phone number — `Phone`

If the request carries `tracked_number`, `called_number` or `to`, a specific number was
dialled. A tracked number exists for exactly one channel, so this is the most certain signal
available and it is checked first.

### 2. `gclid` — `Google Ads`

Google appends this to every ad click. Its presence is proof, not inference. Stored in
`Click ID`.

### 3. `fbclid` — `Facebook Ads`

Same for Meta. Stored in `Click ID`.

**Why click IDs are stored at all.** They are the key that lets you tell Google or Meta later
that click #X became a booked job worth $Y — offline conversion import, which is what teaches
the ad platform to find more people like your actual customers rather than more people who
fill in forms. The ID must be captured at the moment of the click. It cannot be reconstructed
afterwards, from anything. A form not capturing them today is losing that option permanently,
one lead at a time.

### 4. `utm_source` — mapped

Recognised values map to a source. Everything unrecognised becomes `Unknown` rather than being
passed through as a source name, because a UTM is typed by a human and humans typo.

**`google` is special.** It maps to `Google Ads` only when `utm_medium` contains `cpc`, `paid`
or `ppc`. Otherwise it is `Website form`.

This matters more than it looks. UTMs get attached to organic links, email signatures and
social posts all the time. Crediting all of them to Google Ads inflates the spend attributed
to a paid channel with traffic that cost nothing, which makes the paid channel look worse than
it is — and the usual response to that is to cut budget on a channel that was working.

### 5. Referrer — inferred

Checked as a substring, lowercased:

| Contains | Source |
|---|---|
| `angi`, `homeadvisor` | Angi |
| `thumbtack` | Thumbtack |
| `localservices`, `/lsa` | Google LSA |
| `facebook`, `instagram` | Facebook Ads |
| your own domain | Website form |
| anything else | Referral |

Weaker than a click ID — referrers are stripped by browsers, proxies and privacy settings —
but a present referrer is still real evidence.

### 6. Contact details and nothing else — `Website form`

Someone arrived with no tracking of any kind and filled the form in. Direct traffic.

### 7. Nothing matched — `Unknown`

Recorded, reported, never guessed.

## Why `Unknown` is kept visible

The temptation is to assign unmatched leads to the most likely source, so the report has no
gaps. Don't.

A wrong source doesn't just add noise — it moves budget. Attributing 40 unknown leads to
Facebook makes Facebook look twice as efficient as it is, and the business spends more there.
The gap is information: a large `Unknown` count means tracking is broken somewhere, and that
is a fixable problem worth surfacing. Hiding it hides the fix.

## What this does not do

**Multi-touch.** The last click before the form fill takes the credit. A lead who found you
through organic search in March, saw a Facebook ad in April, then clicked a Google ad and
converted is recorded as Google Ads.

This is a deliberate choice, not a shortcut. Multi-touch modelling needs conversion volume
that most single-location service businesses do not have; below a few hundred conversions a
month the models produce confident-looking numbers built on too little data. Last-click is
cruder and honest about being crude.
