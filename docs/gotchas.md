# Things that cost real time

Each of these was a live bug, not a hypothetical.

## A node runs once per input item

n8n executes a node once for every item it receives. A lookup node placed after a step that
returned 93 leads ran 930 times against 10 sources and took 97 seconds.

Set `executeOnce: true` on any node that fetches reference data — a rates table, a config
list. The run dropped to 12 seconds. **Do not remove it** from `Get the sources`.

## Auto-mapping writes every top-level key as a column

The Airtable node's `autoMapInputData` maps every top-level key of the incoming item to a
field of the same name. Two consequences:

- Returning `{ fields: { ... } }` from a Code node fails with `Unknown field name: fields`.
  Return flat column keys instead.
- Any working value you carry between nodes becomes an attempted column write.
  `Unknown field name: _seconds` came from exactly this.

The convention here: working values are prefixed with `_`, and a `Tidy up` node strips every
`_`-prefixed key immediately before the write. One place to fix it, not scattered guards.

Empty strings are also deleted before writing, so an unset field stays unset rather than
being overwritten with blank.

## Measure elapsed time against arrival, not against now

The "submitted too fast" check originally compared the form's load timestamp against
`Date.now()` at the moment the check ran. By then the flow had already spent three or four
seconds saving the lead and searching for duplicates, so every submission looked slow and the
check never fired once.

Compare against when the lead **arrived** — the timestamp written at the top of the flow.

## Don't greet someone with your own placeholder

An unnamed lead is stored as `Unnamed lead` so the row is readable. That string then went
straight into the greeting: *"Hi Unnamed"*.

Any internal placeholder that can reach a customer needs an explicit guard. Here the name is
tested against `/^unnamed/i` and falls back to "there".

## DNS lookups do not tell you whether a domain is available

A parked or newly-registered domain frequently has no nameservers and no A record, so it looks
free to `dig` and to most availability checkers. Ask the registry:

```sh
whois -h whois.verisign-grs.com DOMAIN.com | grep -i "^No match for"
```

Always run a control query against a string nobody could own — plain `whois` on macOS falls
back to IANA for unregistered names and never prints that line, so a missing "No match" can
mean either "taken" or "your command isn't working".

And check the history before buying, not after. A re-registered domain inherits its previous
owner's reputation, including Google Safe Browsing flags:

```sh
curl -s "http://archive.org/wayback/available?url=DOMAIN.com"   # empty = never used
```
