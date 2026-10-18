# agent-false-negative-guard

**Stop AI agents from reporting "absent" or "clean" on empty tool output: positive controls and a three-state result (present / absent / could not measure).**

[![test](https://github.com/Kein-chronicle/agent-false-negative-guard/actions/workflows/test.yml/badge.svg)](https://github.com/Kein-chronicle/agent-false-negative-guard/actions/workflows/test.yml) ![license](https://img.shields.io/badge/license-MIT-blue.svg)

## Why

A command that exits 0 with no output does not mean nothing is there; a sandbox may be hiding it. An agent that reports "no device connected" from an empty result is confidently wrong. The fix is a control that is known to exist and a third answer: could not measure.

> **Status: doc-only.** A discipline to follow, not a tool that catches it for you.

## Install

**As a Claude Code skill** (Claude uses it when the task matches the skill description):

```sh
git clone https://github.com/Kein-chronicle/agent-false-negative-guard.git ~/.claude/skills/measure-or-abstain
```

For a single project, clone into `<project>/.claude/skills/measure-or-abstain` instead, then start a new Claude Code session.

**Just read it:** the whole skill is [SKILL.md](SKILL.md). It is plain Markdown; copy any part into your own `CLAUDE.md` or prompts.

**Requires:** nothing to read it; Python 3.10+ only to run `validate.py`.

## What it covers

- The incident that motivated it.
- Rules: absence needs a positive control, empty output is a result to explain, never collapse "could not measure" into "absent".
- A shell snippet showing the pattern.

## Limits

- A discipline to follow, not a tool that catches it for you.

## More from the same collection

Small, tested tools that grew out of running AI agents and game/app pipelines for months:

- [**llm-verdict-gate**](https://github.com/Kein-chronicle/llm-verdict-gate): Make AI and human review gates impossible to pass by silence: machine-parsed PASS/FAIL, missing verdict = FAIL.
- [**ci-gate-forgery-test**](https://github.com/Kein-chronicle/ci-gate-forgery-test): Prove your CI gates can actually fail: feed each gate forged-bad, empty and missing input and flag the ones that stay green.
- [**stale-verdict-guard**](https://github.com/Kein-chronicle/stale-verdict-guard): Bind a pass/fail verdict to a fingerprint of the source it judged, so an edit after review makes the verdict stale.

## Development

```sh
python3 validate.py
```

This only validates the skill file (frontmatter and referenced paths). See [CONTRIBUTING.md](CONTRIBUTING.md).

## License

MIT. See [LICENSE](LICENSE).

If this saved you a debugging session, a star helps other people find it. Bug reports and failing test cases are even better.

<sub>Keywords: claude-code, claude-skills, agent-skills, ai-agents, debugging, verification, false-negatives, reliability</sub>
