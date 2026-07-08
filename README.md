# AI Disclosure

A tiny convention for honestly labeling AI involvement in source code.

Every file that an AI model substantially contributed to starts with a short
disclosure block in its top-of-file docstring or comment:

```python
"""<One-line description of what the file does.>

AI Disclosure:
    Models:         Claude Fable 5 (claude-fable-5)
    AI-Generated:   mostly        # fully | mostly | partially | none
    Human-Reviewed: partially     # fully | partially | minimally | none
"""
```

Three lines answer three questions: which models wrote this, how much of it
did they write and how closely did a human check the result. The grades are
single words, so any repo using the convention stays greppable:

```bash
grep -r "Human-Reviewed: none" src/
```

The full ruleset is defined in [ai-disclosure.md](ai-disclosure.md).

## Make it your own

This repository defines *my* ruleset. If you like the idea but not the exact
rules, fork this repo, adjust `ai-disclosure.md` to your own workflow and point
your projects at your fork.

## License

[Apache License 2.0](LICENSE).
