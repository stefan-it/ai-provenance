# AI Disclosure 1.0.0

## Summary

Every source file that an AI model substantially contributed to carries an
AI Disclosure block at the top of the file. The block states which models
contributed, how much of the code is AI-generated and how closely a human
reviewed it.

## Motivation

AI-assisted code is now normal, but the degree of assistance varies wildly
between "the model wrote everything" and "the model fixed a typo". Commit
trailers such as `Co-Authored-By` describe a single change and get lost once
code is copied, squashed or vendored. A disclosure block travels with the
file itself: it is visible exactly where the code is read and it can be
audited with a plain `grep`.

## Rules

The key words MUST, MUST NOT, SHOULD and MAY are to be interpreted as
described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

1. A source file to which an AI model substantially contributed MUST start
   with an `AI Disclosure:` block, placed inside the file's top-level
   docstring or leading comment, after an optional one-line description of
   the file.

2. The block MUST contain exactly three fields: `Models:`, `AI-Generated:`
   and `Human-Reviewed:`.

3. `Models:` MUST list every AI model that substantially contributed to the
   file, comma-separated, each entry as `<display name> (<model id>)`.
   Example: `Claude Fable 5 (claude-fable-5)`.

4. `AI-Generated:` MUST carry exactly one of the following grades:

   | Grade       | Meaning                                                              |
   | ----------- | -------------------------------------------------------------------- |
   | `fully`     | AI wrote everything. The human only prompted.                        |
   | `mostly`    | AI wrote the core. The human directed, edited or made small changes. |
   | `partially` | The human wrote the core. AI contributed snippets, fixes or refactors. |
   | `none`      | Pure human work.                                                     |

5. `Human-Reviewed:` MUST carry exactly one of the following grades:

   | Grade       | Meaning                                                       |
   | ----------- | ------------------------------------------------------------- |
   | `fully`     | Every line read and the behavior verified.                    |
   | `partially` | Key parts reviewed or outputs tested, but not line-by-line.   |
   | `minimally` | Skimmed the diff or glanced at the output.                    |
   | `none`      | Merged blind.                                                 |

6. Only a human MAY set or raise `Human-Reviewed:`. An AI tool that creates a
   file MUST set the field to `none` and MUST NOT upgrade it later. The
   review grade is a human attestation, not the model's self-assessment.

7. Grades MUST be written as a single lowercase word, so that repositories
   using this convention stay greppable, e.g.
   `grep -r "AI-Generated: fully" src/`.

8. The block MUST NOT contain timestamps or version numbers. Version control
   history already answers "when" and "in which revision".

9. When a file is substantially edited, the disclosure MUST be updated: add
   the contributing model to `Models:` if it is not listed and re-grade
   `AI-Generated:` if the balance between human and AI authorship shifted.
   After a renewed review, the human SHOULD re-grade `Human-Reviewed:`.

10. Files without any AI contribution MAY omit the block entirely or MAY
    carry it with `AI-Generated: none`, whichever the project prefers. A
    project SHOULD pick one of the two options and use it consistently.

## Examples

Python, as a module docstring:

```python
"""<One-line description of what the file does.>

AI Disclosure:
    Models:         Claude Fable 5 (claude-fable-5)
    AI-Generated:   mostly        # fully | mostly | partially | none
    Human-Reviewed: partially     # fully | partially | minimally | none
"""
```

Any language with line comments, here Go:

```go
// Package example does <one-line description of what the package does>.
//
// AI Disclosure:
//     Models:         Claude Fable 5 (claude-fable-5), Claude Opus 4.8 (claude-opus-4-8)
//     AI-Generated:   partially     # fully | mostly | partially | none
//     Human-Reviewed: fully         # fully | partially | minimally | none
package example
```

## FAQ

**Why per file and not per commit?**

Commits describe changes, files hold the code people actually read. Headers
survive squashes, rebases, vendoring and copy-paste, and they are visible in
every code review and editor.

**Why coarse grades instead of percentages?**

Nobody can honestly measure "63% AI-generated". Four coarse grades can be
judged at a glance and are still meaningful in an audit.

**Why may only humans grade `Human-Reviewed:`?**

Otherwise the model grades its own homework. The field is only worth
something if it reflects what a human actually did.

**How is this enforced?**

Socially, or mechanically with a simple CI check that greps for the block in
every source file. Because the grades are single words, no parser is needed.

## License

[Apache License 2.0](LICENSE).
