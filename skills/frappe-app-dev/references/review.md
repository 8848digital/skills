# REVIEW.md — App-Specific PR Review Checklist

Every app ships a `REVIEW.md` at the repo root. Its job: capture the
domain-specific invariants and critical paths a reviewer (human or Claude)
has no way to know without reading the app's business logic — not the
universal checks. The universal checklist (docstrings, security,
performance, licensing, structural conventions, etc.) already runs on every
review automatically via the `quality-code-review` skill's §0–§8 — see
[SKILL.md](../../quality-code-review/SKILL.md). `REVIEW.md` never restates
that; it only holds what's unique to *this* app.

**When to skip it:** never — every app ships one, even a small app with
just one or two bullets. A trivial app is still worth one line saying so
explicitly, rather than leaving the file missing.

## Required sections

```markdown
# REVIEW.md

## Critical Invariants
Bullet list: business rules that must never break, and what breaks if they
do. e.g. "- Ledger entries must always balance (debit == credit) before
submit — see `<module>/<doctype>/<doctype>.py`'s `validate()`. Breaking
this silently corrupts financial reports."

## High-Risk Areas
Bullet list: files/DocTypes/flows where a subtle bug has outsized
consequences (money, compliance, data loss, irreversible external calls) —
point reviewers there first.

## Domain-Specific Checks
Table: PR touches X → reviewer must verify Y.

| If the PR touches... | Verify... |
| --------------------- | --------- |
| Prize Agreement value | Matches the signed contract on file, not just what the user typed |

## Known Footguns
Bullet list: past incidents or near-misses specific to this app, and the
pattern that caused them, so the same mistake doesn't repeat.
```

Omit a section with nothing in it yet (e.g. no `Known Footguns` until
there's been one) rather than leaving a placeholder — add it the first time
there's a real entry.

## Rules

- **Scaffold at app-creation time** ([new-app.md](./new-app.md) Step 6)
  with at least one `Critical Invariants` bullet — don't leave it a stub
  file with only headings.
- **Update in the same PR** that introduces a new critical invariant,
  high-risk area, or footgun — not as a follow-up.
- This **supplements, never replaces** the generic `quality-code-review`
  checklist — don't copy universal items (docstrings, license headers,
  structural conventions) into here; they're already covered.
- Keep entries short — one or two lines each, pointing to the file/function
  for detail rather than re-explaining the logic inline.

## Anti-patterns

- **Don't turn this into a restatement of `quality-code-review`'s §0–§8.**
  That checklist already runs on every review automatically; `REVIEW.md` is
  only for what's unique to this app.
- **Don't let it go stale after an incident.** A known footgun without a
  `REVIEW.md` line is a repeat waiting to happen — add it the same PR the
  incident is fixed, not as a follow-up.
- **Don't bury it in prose.** Keep entries scannable (bullets/tables) — a
  reviewer reading this under time pressure needs to skim it, not parse
  paragraphs.
- **Don't skip it because the app "has no business logic yet."** A new app
  still gets the file, even if its only content is "No critical invariants
  yet — update this as business logic is added."
