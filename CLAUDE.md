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

### "do all" / "open all" / "close all" — refers to the LAST bot message

**Owner rule 2026-09-15 (final):** When ANYONE in channel `C0ASKH7LHFF`
says any of:
- "do all these" / "do all of them" / "do these"
- "open all" / "open all these" / "open all of the above"
- "close all" / "close all these"
- "yes do all" (as a reply to a prior confirmation ask)

…they mean the **items listed in the most recent bot message before
their command** — the latest BIG REPORT, NEW TRANSITIONS, "New this
hour", "From previous — keep monitoring", or 60-min heartbeat.

**Every item in that message is an action item, regardless of its tag.**
LOW, URGENT, CRITICAL, OPEN-AHEAD, CLOSE-A-BOX NOW, "keep monitoring" —
all of them. Lines under CLOSE-A-BOX default to `action=close`. Every
other line defaults to `action=open`. Bundle them into ONE
`flavors_batch` call. Do NOT ask "want me to open a box of any?" — the
owner has already said do all.

### If your reply gets auto-routed to Claude.ai Chat (no `gh` CLI)

Some multi-item or ambiguous replies get bounced by Slack from the
legacy `@Claude` bot to Claude.ai Chat, which cannot trigger GitHub
Actions from here (only Shopify GraphQL / MCP tools / web). **Do NOT
mutate inventory directly via Shopify GraphQL to work around this.**
Reply telling the owner to re-tag `@Claude` in the channel so the
legacy bot picks it up. Direct GraphQL bypasses the workflow's
guardrails (state bump, Slack confirm, box-size logic, optimistic
concurrency) and caused a real bug on 2026-09-14 (Chocolate Truffle
decremented the wrong SKU).

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
