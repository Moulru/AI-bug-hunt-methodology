# AI Bug Hunt Methodology

[한국어](README.ko.md)

A step-by-step methodology that tells an AI coding agent how to hunt for **real, reproducible bugs** across a whole project, not just review code.

Hand it to Claude Code, Codex, Cursor, Gemini CLI, or any other agent, and it will:

1. Build a **coverage table** first (every screen × every entry path × every exit path × every state), so "done" means every cell was checked, not just that every method was run.
2. Apply **20 methods**: pyramid and inverted-pyramid exploration, user scenarios, state transitions, boundary values, implicit assumptions, data flow, contracts, duplication, async/race conditions, reverse permissions, regressions, external behavior, timeouts, server-side enforcement, navigation lifecycle, writes after failure and side effects, and UI consistency and wording.
3. Run an **independent second hunt** with fresh eyes that never see the first hunt's findings.
4. Report each bug as **Confirmed** or **Suspected**, with severity, reproduction steps, cause, files, and how far it was actually verified.
5. Keep a **missed-bug retrospective**, so the methodology improves every time a bug slips through.

The methodology itself is [`skills/bug-hunt/SKILL.md`](skills/bug-hunt/SKILL.md).

## Install

### Claude Code

```text
/plugin marketplace add Moulru/ai-bug-hunt-methodology
/plugin install bug-hunt@ai-bug-hunt-methodology
```

Then ask: **"Run a full bug hunt on this project."**

### Any other agent (Codex, Cursor, Gemini CLI, ChatGPT, ...)

Give the agent this file and ask it to follow it:

```text
https://raw.githubusercontent.com/Moulru/ai-bug-hunt-methodology/main/skills/bug-hunt/SKILL.md
```

Or copy [`AGENTS.md`](AGENTS.md) into your project root.

## Report format

Each bug includes: severity (Critical / High / Medium / Low), title, trigger conditions, steps to reproduce, actual result, expected result, cause, related files and functions, impact, whether it was reproduced, **verification level** (code only / unit test / screen render / live server or DB call / real device), and **which method found it**.

The report ends with the filled coverage table, per-method coverage and untested areas, and the missed-bug retrospective.

## Repository layout

```text
skills/bug-hunt/SKILL.md      The methodology (English), as an agent skill
METHODOLOGY.ko.md             The methodology (Korean)
AGENTS.md                     Pointer for agents that read AGENTS.md
.claude-plugin/               Claude Code plugin and marketplace manifest
```

## Contributing

Found a bug the methodology should have caught? Open an issue with one line: the bug, which method should have caught it, why it was missed, and what to change. That is exactly the retrospective in method 20.

## License

[MIT](LICENSE)
