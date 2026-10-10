# skills-orquesta

Two multilingual skills for Claude (Claude Code / Cowork) for technical projects: Flutter/Android apps, Docker, and home security (cameras, Frigate, Home Assistant, NAS, UPS).

| Skill | What it does |
|---|---|
| `verify` | Forces real verification before anything is called "done": real commands, evidence, and a fixed report (OK / FAIL / NOT VERIFIED). |
| `director` | A **director** that classifies the task, blocks drift and requires every action to be justified, plus several **researchers** that build a Markdown notebook (`knowledge/`) with real sources. The director queries the notebook with precise searches instead of reading it all. Researchers look in every place that may help (official docs, code and issues, forums, reviews, papers, shops, other languages) and label each source by type and trust. |

> **Status: early testing.** Español: [README.es.md](README.es.md)

## Languages

Each language is a complete, independent folder. **Install one language only**: the skills share the same names, so installing two languages overwrites one with the other.

| Folder | Language | Folder | Language |
|---|---|---|---|
| `en` | English (reference) | `ko` | 한국어 |
| `es` | Español (original) | `ru` | Русский |
| `pt` | Português | `ar` | العربية |
| `fr` | Français | `hi` | हिन्दी |
| `de` | Deutsch | `tr` | Türkçe |
| `it` | Italiano | `nl` | Nederlands |
| `zh` | 简体中文 | `pl` | Polski |
| `ja` | 日本語 | `id` | Bahasa Indonesia |

**Translation quality:** the original is in Spanish. The other languages were translated by an AI and have **not been reviewed by native speakers**. They are structurally identical to the original (same sections, same line count), but wording may be improved. Corrections are welcome as pull requests, and so are new languages (copy the `en` folder and translate it).

## Install

Copy the two skill folders of your language into your skills folder. Example for English:

```bash
# For all your projects
mkdir -p ~/.claude/skills
cp -r en/verify en/director ~/.claude/skills/

# Or for one project only
mkdir -p .claude/skills
cp -r en/verify en/director .claude/skills/
```

Spanish: `cp -r es/verificar es/director ~/.claude/skills/`

Restart your Claude session and check that `verify` (or `verificar`) and `director` appear.

## Install as a plugin (Spanish, with automatic reminders)

The `plugin/` folder is a Claude Code plugin called `orquesta`. It bundles the Spanish `director` and `verificar` skills plus a hook that reminds Claude to use them at the start of every session. You no longer have to type `/director` or `/verificar`. The plugin is **Spanish only**; for other languages use the manual install above.

```bash
claude plugin marketplace add Sks1785/skills-orquesta
claude plugin install orquesta@skills-orquesta
```

Or inside a Claude Code session: `/plugin marketplace add Sks1785/skills-orquesta`, then `/plugin install orquesta@skills-orquesta`. Then run `/reload-plugins` or open a new session.

Things to know:
- The skills are namespaced by the plugin: `/orquesta:director` and `/orquesta:verificar`.
- If you copied `director` / `verificar` into `~/.claude/skills` before, move them out first, or you will have duplicates.
- The hook only runs an `echo`. On Windows, Claude Code uses Git Bash when it is installed (tested: one clean line). With the PowerShell fallback the same text prints one word per line, which is still readable.
- The hook adds one short reminder at session start. To remove it, edit or delete `plugin/hooks/hooks.json`. To uninstall: `claude plugin uninstall orquesta@skills-orquesta`.
- `plugin/skills/` is a copy of `es/` (same text; only the line endings differ). If you change one, change the other.
- Tested on Windows 11 with Claude Code 2.1.278: `claude plugin validate` passes, the session-start hook fires and both skills load (version 0.1.0 also had a per-prompt hook; it was removed in 0.1.1). It has only been tried on one real task.

## Use
- Start a task with `/director`, or let Claude activate it from its description.
- After each important change, run `/verify`.
- To correct the director, tell it "correct:", "that's not right:" or "note this:" (or the equivalent in your language) and it logs the lesson in `knowledge/_lessons.md`.

## Honest limits
- Director and researchers are the same AI with separate contexts, not independent experts. Mandatory sources and validation reduce errors; they do not remove them.
- More agents mean more time and more tokens.
- Text search in the notebook depends on tags and synonyms, not on meaning.
- Community sources (forums, Reddit, videos) are treated as user experience, not as documentation, and need two independent sources before counting as confirmed.

## License
MIT (see `LICENSE`).
