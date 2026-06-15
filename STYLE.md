# Essay conventions

Conventions for the essays on this site. Keep this short; it documents the
mechanical rules, not voice.

## Revision notes (standing convention)

**Any time a published essay is edited after its `date:`, append a dated
revision note to the foot of that essay** — the series is about honesty and
verification, so it tracks its own corrections in the open.

Placement: after the closing `Related:` line, separated by a `---`.

Format:

```markdown
---

*Revised YYYY-MM-DD: <one line on what changed and why; name the section if a
claim was softened>.*
{:.essay-revision}
```

- The `{:.essay-revision}` IAL styles it as a muted footnote (`assets/css/site.css`).
- Also bump `last_updated:` in the front matter to the same date.
- Keep it to one line. Substantive changes only (retitle, correction, a new
  section or pattern). Don't log typo-level tweaks.
- If a claim was softened because it was thin/unverified, say so plainly — a
  public "we tightened the sourcing here" is the point, not something to hide.

Example (live):

```markdown
*Revised 2026-06-15: retitled from "Instruction is not a control," with a
matching emperor's-new-clothes line added to "A law by any other name." In
"Secrets and identity," a single-sourced, uncorroborated incident attribution
was softened; the structural argument is unchanged.*
{:.essay-revision}
```

## URLs

Essay slugs (the directory name under `essays/`) are permanent once published.
Retitle freely in the front matter and the index card, but **do not rename the
slug** — it would break the published URL, inbound links, and analytics. A
title/slug mismatch is fine (e.g. "The Emperor's New Controls" lives at
`/essays/instruction-is-not-a-control/`).
