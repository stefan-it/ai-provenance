# AI Provenance 2.0.0

## Summary

Every source file that an AI model substantially contributed to carries an
AI Provenance block at the top of the file. The block states which models
contributed, how much of the code they wrote, whether a review agent checked
the result and how closely a human reviewed it.

The block is a label, not a confession. AI-written code is the normal case.
The block exists so that a reader can calibrate trust in a file at a glance
and so that an audit stays a plain `grep`.

## Motivation

AI-assisted code is now normal, but the degree of assistance varies wildly
between "the model wrote everything" and "the model fixed a typo". The same
is true for review: a file may have been read line by line by a human, or
checked by review agents and merged on green CI with nobody reading it. Both
are legitimate workflows for the right kind of change, but a reader deserves
to know which one produced the file in front of them.

Commit trailers such as `Co-Authored-By` describe a single change and get
lost once code is copied, squashed, vendored or stored in a source control
system that keeps metadata server-side. A provenance block travels with the
file itself: it is visible exactly where the code is read and it can be
audited without any tooling.

## Rules

The key words MUST, MUST NOT, SHOULD and MAY are to be interpreted as
described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

1. A source file to which an AI model substantially contributed MUST start
   with an `AI Provenance:` block, placed inside the file's top-level
   docstring or leading comment, after an optional one-line description of
   the file.

2. The block MUST contain exactly four fields, in this order: `Models:`,
   `Authored:`, `AI-Reviewed:` and `Human-Reviewed:`.

3. `Models:` MUST list every AI model that substantially contributed to the
   file, comma-separated, each entry as `<display name> (<model id>)`.
   Example: `Claude Fable 5.1 (claude-fable-5-1)`. Models that only
   reviewed the file are not listed here.

4. `Authored:` MUST carry exactly one of the following grades:

   | Grade          | Meaning                                                               |
   | -------------- | --------------------------------------------------------------------- |
   | `ai`           | AI wrote everything. The human only prompted.                         |
   | `mostly-ai`    | AI wrote the core. The human directed, edited or made small changes.  |
   | `mostly-human` | The human wrote the core. AI contributed snippets, fixes or refactors. |
   | `human`        | Pure human work.                                                      |

5. `AI-Reviewed:` MUST carry exactly one of the following grades:

   | Grade  | Meaning                                                                                                                                 |
   | ------ | --------------------------------------------------------------------------------------------------------------------------------------- |
   | `yes`  | A dedicated review run (a review agent or subagent in a fresh context, not the authoring session) examined the file and every finding was fixed or resolved with a reason. |
   | `none` | No separate review run, or findings were left unresolved.                                                                               |

   A review by the same session that wrote the code does not count. The
   point of the field is a second pair of eyes, even if those eyes are a
   model's.

6. `Human-Reviewed:` MUST carry exactly one of the following grades:

   | Grade     | Meaning                                                                                                                    |
   | --------- | -------------------------------------------------------------------------------------------------------------------------- |
   | `lines`   | A human read every line and understood the behavior.                                                                       |
   | `intent`  | A human understood what the file does and why, checked the evidence (tests, review findings, outputs), but did not read every line. |
   | `skimmed` | A human glanced at the diff or the output.                                                                                 |
   | `none`    | Merged without a human looking at the file.                                                                                |

   `intent` is the expected grade for most AI-written files in a project
   that runs review agents and automated checks. `lines` is for code where
   the blast radius justifies the time: authentication, billing, release
   tooling, data migrations, architecture that others will copy.

7. Only a human MAY set or raise `Human-Reviewed:`. An AI tool that
   creates a file MUST set the field to `none` and MUST NOT upgrade it
   later. An AI tool MAY set `AI-Reviewed:` to `yes` after a
   dedicated review run has completed and all findings are resolved. An
   agent's review never counts as a human's: `AI-Reviewed: yes` does not
   change `Human-Reviewed:`.

8. Grades MUST be written as a single lowercase token (letters and hyphens
   only), so that repositories using this convention stay greppable, e.g.
   `grep -r "Human-Reviewed: none" src/`.

9. The block MUST NOT contain timestamps, version numbers or the names of
    the humans involved. Version control history already answers "when",
    "in which revision" and "who committed".

10. When a file is substantially edited, the block MUST be updated: add the
    contributing model to `Models:` if it is not listed, re-grade
    `Authored:` if the balance between human and AI authorship shifted, and
    reset `AI-Reviewed:` and `Human-Reviewed:` to the grades that are true
    for the edited file. A review of an earlier version does
    not carry over. After a renewed review, the human SHOULD re-grade.

11. Files without any AI contribution MAY omit the block entirely or MAY
    carry it with `Authored: human`, whichever the project prefers. A
    project SHOULD pick one of the two options and use it consistently.

12. A project SHOULD state its review policy once, in a `PROVENANCE.md` at
    the repository root or in a section of its README: which review agents
    run, which paths always require `Human-Reviewed: lines`, which
    combinations of grades are acceptable for which kind of change, and
    what verification (tests, CI, manual checks) MUST pass before a file
    may carry `Human-Reviewed: none`. Verification is a property of the
    process and is recorded there, not in the per-file block. The per-file
    block answers "what happened to this file"; the policy answers "what is
    expected here".

## Examples

Python, as a module docstring:

```python
"""<One-line description of what the file does.>

AI Provenance:
    Models:         Claude Fable 5.1 (claude-fable-5-1)
    Authored:       mostly-ai     # ai | mostly-ai | mostly-human | human
    AI-Reviewed:    yes           # yes | none
    Human-Reviewed: intent        # lines | intent | skimmed | none
"""
```

Any language with line comments, here Go:

```go
// Package example does <one-line description of what the package does>.
//
// AI Provenance:
//     Models:         Claude Fable 5.1 (claude-fable-5-1), Claude Opus 5.5 (claude-opus-5-5)
//     Authored:       mostly-human  # ai | mostly-ai | mostly-human | human
//     AI-Reviewed:    yes           # yes | none
//     Human-Reviewed: lines         # lines | intent | skimmed | none
package example
```

A low-risk file that was merged on automated checks alone. This is a valid
state, not a failure:

```python
"""Render the changelog page from CHANGELOG.md.

AI Provenance:
    Models:         Claude Fable 5.1 (claude-fable-5-1)
    Authored:       ai            # ai | mostly-ai | mostly-human | human
    AI-Reviewed:    yes           # yes | none
    Human-Reviewed: none          # lines | intent | skimmed | none
"""
```

## Auditing

Because grades are single tokens, the interesting questions are one `grep`
each. Files nobody and nothing has checked:

```bash
grep -rl "AI-Reviewed: none" src/ | xargs grep -l "Human-Reviewed: none"
```

AI-written files that no review agent has looked at:

```bash
grep -rl "Authored: ai" src/ | xargs grep -l "AI-Reviewed: none"
```

Sensitive paths that should have been read line by line but were not:

```bash
grep -rL "Human-Reviewed: lines" src/auth/ src/billing/
```

## FAQ

**Why per file and not per commit?**

Commits describe changes, files hold the code people actually read. Headers
survive squashes, rebases, vendoring and copy-paste, and they are visible in
every code review and editor. They also survive a move to source control
systems that keep commit metadata on a server.

**Why coarse grades instead of percentages?**

Nobody can honestly measure "63% AI-generated". Four coarse grades can be
judged at a glance and are still meaningful in an audit.

**Why is there no field for tests or verification?**

Because verification is a property of the project's process, not of a file.
Whether tests exist and pass is answered by CI and coverage tooling, and a
hand-written label drifts the moment a test is deleted. The policy in
`PROVENANCE.md` states what must pass before a file may be merged without a
human reviewer.

**Why is `intent` a separate grade and not just `partially`?**

Reading every line of agent-written code does not scale and is not what
careful teams do anymore. Understanding what a file does, checking that the
tests and the review findings back that up, and resolving every finding is a
real review with a real outcome. It deserves its own name, so that it is
neither inflated to "read every line" nor deflated to "skimmed".

**Why track AI review separately from human review?**

Because they are different things and both matter. A review agent catches
most routine mistakes. A human decides whether the change should exist at
all, and owns the result. Collapsing the two into one field would either
overstate what the human did or hide what the agent did.

**Why may only humans grade `Human-Reviewed:`?**

Otherwise the model grades its own homework. The field is only worth
something if it reflects what a human actually did.

**Isn't `Human-Reviewed: none` embarrassing?**

No. For a documentation page, a copy change or an additive feature behind a
flag, a passing review agent and green CI are enough, and pretending a
human read it would be the dishonest option. The block makes the trade-off
visible so that the next reader can decide whether it was the right one.
What has to pass before `none` is allowed belongs in the project's
`PROVENANCE.md` (rule 12).

**How is this enforced?**

Socially, or mechanically with a simple CI check that greps for the block in
every source file and rejects disallowed combinations for protected paths.
Because the grades are single tokens, no parser is needed.

## Changes from AI Disclosure 1.0.0

- Renamed from "AI Disclosure" to "AI Provenance". Disclosure implies
  something to confess; provenance describes where code came from and who
  checked it.
- `AI-Generated:` is now `Authored:` with grades `ai`, `mostly-ai`,
  `mostly-human`, `human` (was `fully`, `mostly`, `partially`, `none`).
- `Human-Reviewed:` grades are now `lines`, `intent`, `skimmed`, `none`
  (was `fully`, `partially`, `minimally`, `none`). `intent` is new and
  describes review of what the code does and the evidence for it, without
  reading every line.
- New field `AI-Reviewed:` for dedicated review runs by agents.
- Editing a file now resets the review grades (rule 10).
- Projects should state their review policy, including required
  verification, once in `PROVENANCE.md` (rule 12).

## License

[Apache License 2.0](LICENSE).
