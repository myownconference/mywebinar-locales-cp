# Session start

Before any work in a new session, read these context sources in order.
Sources 1-3 are loaded *by reference* (this file points at them, it does not contain them);
the codebase map at the bottom lives here because it is versioned with the code.

1. **GLOBAL RULES** — `~/.claude/CLAUDE.md`. Partnership mindset, Plan First, workflow,
   the `архив` / `коммит` / `поехали` commands.
2. **GLOBAL MEMORY** — every file in `~/.claude/memory/**` (if present). Cross-project:
   user profile, a digest of the global rules (`reference_global_rules.md`), and the
   session-data layout (`reference_session_data_layout.md`).
3. **PROJECT MEMORY** — `~/.claude/projects/<this-project>/memory/`. The single source of
   truth for project knowledge: decisions, conventions detail, session history. Read
   `MEMORY.md` (the index) first, then open the topic files it links on demand — do **not**
   bulk-read the whole dir. Resolve the path from cwd:
   ```bash
   ls ~/.claude/projects/$(echo "$PWD" | sed 's|/|-|g')/memory/
   ```

Only after reading sources 1-3, start the session's tasks.

## Memory protocol (read AND maintain)

- **One operational memory, one location.** All project memory lives in source #3. Do **not**
  create a `./memory/` in the repo — a second store causes a split brain.
- **Absolute paths only.** Never scan from `/`; projects live under `~/Projects/**`
  (global rules → *Filesystem & Paths*).
- **Update memory after every task** (translation batch, audit, investigation) automatically,
  without being asked — a mandatory final step like lint/audit (global rules → *After Completing
  a Task*). New finding → a topic file in #3 plus one index line in its `MEMORY.md`.
- **`MEMORY.md` hard cap: ≤ 24 KB AND ≤ 190 lines.** Claude Code loads only the first 200 lines
  / 25 KB; the rest is silently dropped. Check `wc -c` and `wc -l` on every update; details go
  in topic files, never inline in the index.

## Audit discipline (parity checks, re-audits)

> How to make audits of this repo accurate and reproducible. Generic principles; a project may
> extend with concrete locale/key specifics. Full detail: PROJECT MEMORY → `reference_repo_structure.md`.

**Key population**
- **Key census = the canonical `en` file, never a partial locale.** Any "how many keys / which
  keys" question must count `en/admin.php` (the reference, 1203 keys) — a translated locale can
  drift (e.g. `ru` has 1205). Count keys with `grep -cE "^\s*'[a-z0-9_-]+' =>" <file>`.
- **Parity check = diff the key SET, not the values.** "All 36 locales have key X" needs a
  presence check across every locale dir, not eyeballing one. A locale missing a key is a real
  defect, not cosmetic.

**Aggregates & numbers**
- **State the exact scope of every aggregate, inline:** which file (admin/countries/etc.),
  which locale, raw key count vs translated. A bare "translated" is unverifiable.
- **"Every / all 36 locales" needs the full population, not a sample.** `*/admin.php` glob over
  all 36 dirs; report the count of matches vs 36 before calling a key universal.

**Cross-source reconciliation**
- **Locale-code → language must be verified by content, not guessed from the 2-letter code.**
  Several codes here are internal aliases, not ISO country codes (`gr`=Greek, `cn`=zh-Hans,
  `cz`=cs, `nb`=no). Check PROJECT MEMORY's locale map before treating a code as its apparent
  language.
- **Lower line-count ≠ "English fallback".** `be` (Belarusian) and `br` (Brazilian Portuguese)
  are **partial translations** (be ≈ 255, br ≈ 142 translated lines) — they contain real
  translations in their own language, just fewer of them. Untranslated keys keep the `en` value.
  Measure translation *coverage*, not whether translation exists at all.

**Framing & conclusions**
- **Translation coverage is a spectrum, not a binary.** Don't label a locale "fallback" from a
  single-sample line-diff. Measure per-locale coverage (script-detection line counts) before
  claiming a locale is untranslated. See the audit antipattern note in PROJECT MEMORY.

**Output**
- Per material claim: expected / actual / verdict (e.g. VERIFIED / PARTIAL / REFUTED) + the
  command — so a re-auditor reproduces without guessing your filter.

## Why this file exists

ZCode (opencode convention) loads only `<project>/AGENTS.md` and ignores `CLAUDE.md` and the
global files — so this file is the single source for both ZCode and Codex. Claude Code instead
auto-loads `<project>/CLAUDE.md`; pair this file with a thin `CLAUDE.md` (from
`CLAUDE.template.md`) whose only job is `@AGENTS.md`, so Claude Code reaches this same file. So
it does double duty: a **bootloader** pointing at global rules + memory (sources 1-3), and the
**in-repo codebase map** below. Keep the bootloader thin; load heavy / operational context by
reference, keep only the map inline.

**Size limits.** The official agents.md spec has none. Codex caps the combined instruction chain
at **32 KiB** (`project_doc_max_bytes`, silent truncation beyond; raise in
`~/.codex/config.toml`). Best practice keeps the bootloader short; the map grows with the
project but stays well under 32 KiB.

---

# Project — MyOwnConference control-panel locales

Localization files (PHP `return [...]` arrays) for the MyOwnConference control panel. This repo
holds **translations only**, not application code — the CP application lives elsewhere and
consumes these files. 36 locales × 5 files = 180 PHP files, ~368 K lines total, since 2021-09-30.
MIT License. Full conventions + locale code map: PROJECT MEMORY → `reference_repo_structure.md`.

## Production

N/A for this repo. It is a data-only localization store; there is no server, runtime, or log
path here. The CP application that consumes these files is a separate project.

## Layout

```
<locale>/               one dir per locale, 2-letter internal code (36 total)
  admin.php             control-panel UI strings (~1203 keys in en; the big file)
  countries.php         country names
  paid-webinars.php     pricing / payment
  referral.php          referral program
  tests.php             webinar checks/tests
README.md               one line ("Locales for MyOwnConference control panel")
LICENSE                 MIT, 2021 MyOwnConference.com
.gitignore              .DS_Store
.vscode/settings.json   local editor config (ansible python path)
```

Locale dirs present (36): `ar be bg br cn cz da de en es et fi gr he hi hu it ja ka kk ko lt lv
nb nl pl pt ro ru sk sl sv tr uk vi`. ⚠️ Codes are internal aliases, not all ISO — see PROJECT
MEMORY's locale map (`gr`=Greek, `br`=PT-EN-fallback, `cn`=zh-Hans, `cz`=cs, `nb`=no).

## Data / control flow

```
(new UI string in CP app)
  → decide a kebab-case key + English value
  → add to en/<file>.php (reference)
  → add to all 36 locales (translated; be/br copy en value)
  → commit "locales: add <key> (all 36 languages)"
```

## Quick start

This is a data-only repo — there is no install, build, lint, typecheck, or test step. Work is
editing PHP locale arrays and committing with the project's message convention.

```bash
git status                  # working tree state
git log -10 --oneline       # recent changes (convention: "locales: add <key> ...")
# key parity check across all locales:
for d in */; do grep -q "'<key-name>'" "$d/admin.php" || echo "missing in $d"; done
```

## Key files

| File | Lines | Purpose |
|------|------:|---------|
| `en/admin.php` | 7430 | reference locale, control-panel strings (1203 keys) |
| `en/countries.php` | 1382 | country names |
| `en/tests.php` | 915 | webinar test/check strings |
| `en/paid-webinars.php` | 324 | pricing/payment strings |
| `en/referral.php` | 181 | referral program strings |
| `ru/admin.php` | — | Russian; 1205 keys (drift of +2 vs en, historical) |

Each of the 36 locale dirs mirrors the `en/` 5-file set. `admin.php` is by far the largest file
in every locale.

## Conventions

- **File format:** `<?php` → MyOwnConference copyright header → `return [` → key blocks
  separated by `////...` lines with the key name echoed in the comment. Tabs for indentation;
  values are single-quoted multi-line strings (`\'` to escape). See PROJECT MEMORY →
  `reference_repo_structure.md` for the exact template.
- **Key naming:** kebab-case, ASCII lowercase + digits + hyphen (e.g. `activate-javascript`,
  `paid-for`, `per-month-info`, `upload-photo`).
- **`en` is the reference locale.** New keys land there first, then propagate to all 36 locales.
  Translation **coverage varies**: locales like `ru uk bg ar cn gr he hi ja ka kk ko` are nearly
  fully translated (~1200 keys with non-Latin scripts), while `be` (Belarusian) and `br`
  (Brazilian Portuguese) are **partial** (be ≈ 255 translated lines, br ≈ 142). ⚠️ A lower line
  count is NOT "English fallback" — be/br contain real translations in their target language;
  untranslated keys there simply retain the `en` value. When adding a key, provide a translation
  where the locale is actively maintained; leave the `en` value only where no translation exists.
- **Commit message convention:** `locales: add <key> (all 36 languages)` for a full rollout;
  `locales: add <key> (en/ru/uk)` for a partial first step followed by
  `locales: translate <key> to all languages`; bare `locales update` for revisions.
- **No application code, no hardcoded strings to hunt** — every string already lives in these
  files by construction. The "move hardcoded text to locales" rule applies to the CP app repo,
  not here.
- **Copyright header year is `2003-2023`** across all 180 files and is **intentionally not
  auto-bumped** — do not mass-edit it without an explicit request.

**Copyright header** (MyOwnConference org default — prepend to any new locale file in this repo):

```
// +-------------------------------------------------------------------+
// | Copyright 2003-2023 MyOwnConference.com. All rights reserved.     |
// |                                                                   |
// | Unauthorized copying of this file, via any medium is strictly     |
// | prohibited. Proprietary and confidential.                         |
// |                                                                   |
// | Written for myownconference.com <contact@myownconference.com>     |
// +-------------------------------------------------------------------+
```
