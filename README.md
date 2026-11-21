# AI Provenance

A tiny convention for labeling where source code came from and who checked
it.

AI-written code is the normal case now. What varies is how much of a file a
model wrote, whether a review agent looked at it and how closely a human
looked at it. Every file that an AI model substantially contributed to
answers those questions in a short block in its top-of-file docstring or
comment:

```python
"""<One-line description of what the file does.>

AI Provenance:
    Models:         Claude Fable 5.1 (claude-fable-5-1)
    Authored:       mostly-ai     # ai | mostly-ai | mostly-human | human
    AI-Reviewed:    yes           # yes | none
    Human-Reviewed: intent        # lines | intent | skimmed | none
"""
```

The block is a label, not a confession. `Human-Reviewed: none` on a
low-risk file with green CI and a clean agent review is a legitimate
state. The block exists so the next reader can calibrate trust at a glance.

Only a human may set `Human-Reviewed:`. An agent may set
`AI-Reviewed:` after a dedicated review run, but an agent's review never
counts as a human's.

The grades are single tokens, so any repo using the convention stays
greppable:

```bash
# files nobody and nothing has checked
grep -rl "AI-Reviewed: none" src/ | xargs grep -l "Human-Reviewed: none"

# AI-written files no review agent has looked at
grep -rl "Authored: ai" src/ | xargs grep -l "AI-Reviewed: none"
```

The full ruleset is defined in [ai-provenance.md](ai-provenance.md).

## Coming from AI Disclosure 1.0.0

This convention was previously called "AI Disclosure". Version 2.0.0 renames
it, renames `AI-Generated:` to `Authored:`, replaces the review grades with
`lines`, `intent`, `skimmed` and `none`, and adds the `AI-Reviewed:`
field. The full list of changes is at the end of
[ai-provenance.md](ai-provenance.md).

## Make it your own

This repository defines *my* ruleset. If you like the idea but not the exact
rules, fork this repo, adjust `ai-provenance.md` to your own workflow and
point your projects at your fork.

## License

[Apache License 2.0](LICENSE).
