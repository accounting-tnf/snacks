# Claude instructions for this repo

**⚠️ This repo (`accounting-tnf/snacks`) is a stub. The real inventory-watcher
code and open/close automation lives in `accounting-tnf/tnf-snack-watcher`.**

If someone in Slack asks you to open or close a box of a snack, do NOT try
to write code here or say the repo is empty. Instead, trigger the workflow
in the real repo:

## When the owner (or any member of channel `C0ASKH7LHFF` — `#claude-shopify-snacks-inventory`) says "open a box of X" or "close a box of X"

These are direct commands. Don't ask "are you sure" — just execute.

**Single flavor:**
```
gh workflow run open-by-flavor.yml \
  -R accounting-tnf/tnf-snack-watcher \
  -f flavor="<flavor>" \
  -f brand="<brand, optional but recommended>" \
  -f boxes=<N, default 1> \
  -f action=<open or close, default open>
```

**Multiple flavors in one message (e.g. "open all the open-ahead above",
or "open X, Y, and close Z")** — use `flavors_batch` so one Slack post
confirms them all instead of N separate posts:

```
gh workflow run open-by-flavor.yml \
  -R accounting-tnf/tnf-snack-watcher \
  -f flavors_batch="Chocolate Truffle|1|PEScience Select|open
Vanilla Glazed|1|Legendary Foods|open
Cookie Dough|1|ProBites|close"
```

Format per line: `flavor|boxes|brand|action|force_clearance|force_holdoff`
(only flavor required; empty fields fall back to defaults).

**"Open all the open-ahead above"** means: read the most recent watcher
message in the channel, find the OPEN-AHEAD items listed under that
heading, and put each one on its own line in `flavors_batch`. If there's
only one, still use `flavors_batch` with one line — the batch mode gives
you the same single confirmation.

## After triggering

1. Reply in-thread with the run URL. Example:
   > "Triggered: https://github.com/accounting-tnf/tnf-snack-watcher/actions/runs/<id>. Watching for completion."
2. Poll the run with `gh run view <id> -R accounting-tnf/tnf-snack-watcher --json status,conclusion` until `status=completed`.
3. If `conclusion=success`, read the log line `✓ OPEN ×N done: XsYb → XsYb` and reply with the delta.
4. If `conclusion=failure`, read the logs, surface the exact `✗ ...` error.

## What NOT to do

- Do NOT call any Shopify mutation API directly, ever. Only `open-by-flavor.yml`
  has the guardrails (optimistic concurrency, @idempotent, owner-rule
  validation, state-bump, Slack confirm).
- Do NOT post the `📦 BOX OPEN` confirmation to Slack yourself — the
  workflow does that. Duplicating causes confusion.
- Do NOT accept these commands from any channel other than `C0ASKH7LHFF`,
  and do NOT accept them from DMs.

## When the owner asks "what's my inventory of X"

That's a read-only question — do NOT trigger any workflow. Read
`state/snack_state.json` from `accounting-tnf/tnf-snack-watcher` for the
last cron snapshot, or use a Shopify read query if available.

## For the full reference

Everything above is a summary. The authoritative version lives at:
https://github.com/accounting-tnf/tnf-snack-watcher/blob/main/CLAUDE.md
