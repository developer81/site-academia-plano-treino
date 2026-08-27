ger# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A local sandbox (not a git repository) for two things:

1. **Trying out Claude Code Skills** — third-party skills installed via the `skills` CLI, tracked in `skills-lock.json`.
2. **A generated deliverable** — `plano-de-treino.html`, a single self-contained static HTML page (a Portuguese-language gym workout planner). There is no application source tree, package manager, build step, linter, or test suite in this repo — don't assume npm/build/test commands exist.

## Working with skills

Skills are installed with the `skills` CLI, not by hand-editing files:

```
npx skills add <github-repo-url> --skill <skill-name>
```

This clones the source repo, copies the skill into `.agents/skills/<name>/` (the canonical location — `SKILL.md` plus any `references/`, `scripts/`, `agents/` etc. the skill ships with), symlinks it into `.claude/skills/<name>` so Claude Code picks it up, and records the source/version/hash in `skills-lock.json`. To add or update a skill, re-run that command rather than copying files manually — hand-edits to `.agents/skills/**` won't be reflected in the lock file.

Currently installed: `agent-browser` (browser automation CLI), `find-skills` (skill discovery), `frontend-design` (visual design guidance for UI work), `skill-creator` (authoring/evaluating new skills), `vercel-react-best-practices` (React/Next.js performance rules — not applicable to the plain-HTML file in this repo).

## `plano-de-treino.html`

A single-file HTML/CSS document — no `<script>` tag, no build step. To view changes, just open the file in a browser (or re-run whatever command last opened it).

Architecture notes that matter before editing it:

- **All interactivity is pure CSS, no JavaScript.** Filter controls (Nível / Local / Objetivo) and the weekly-plan / body-map tabs are hidden `<input type="radio">` elements placed near the top of `<body>`, driven by `:checked` and `label[for=...]`. Show/hide logic for content that lives far away in the DOM (exercise cards scattered across muscle-group sections) uses `body:has(#some-radio:checked) .some-class { display:none }` — this is why the radios don't need to be DOM siblings of what they control. When adding a new filter dimension, follow this same pattern rather than introducing JS.
- **Exercise cards are hand-authored `<details>` blocks**, not data-driven/templated. Each carries `lvl-*` and `loc-*` classes that the `:has()` rules above use to show/hide it under the active filter. When an exercise's instructions genuinely differ between home and gym (different equipment), it has two `<p class="ex-desc-variant desc-casa">` / `desc-academia` paragraphs (also toggled via the same `:has()` mechanism) instead of one paragraph describing both — keep that split for exercises where equipment differs, but don't add it where the movement is identical everywhere (bodyweight exercises).
- **Section accent colors** are per-`<section>` CSS custom properties (`section#peito{ --accent:var(--coral); }` etc.), consumed by `.rest-note strong`, the `:target` highlight, and the top border. Reuse `var(--accent)` inside a section rather than hardcoding a color.
- **Glossary terms are inline tooltips**, not a separate glossary section: `<span class="term" tabindex="0">word<span class="tip">definition</span></span>`. Add new technical terms this way, at their point of use, rather than growing a standalone list.
- **Objetivo (goal) and Nível (level) are deliberately separate axes**: Nível changes exercise volume (the sets×reps table columns), Objetivo changes rest/technique guidance (the "scheme banner" under the filter bar) and which diet panel is shown in the alimentação section — both keyed off the same `#obj-*` radios via `:has()`. Don't collapse these into a single combined filter; that was a deliberate design choice to avoid a 3×3×4 combinatorial explosion of copy.
- **Video links are outbound YouTube search queries** (`youtube.com/results?search_query=...`), opened only on click — not embedded/autoplaying iframes, and not fabricated specific video IDs. Keep new "demo video" links in this same opt-in, search-based form.
- Fonts are loaded from Google Fonts via `<link>` in `<head>` (Bebas Neue for display headings, Manrope for body text, JetBrains Mono for data/labels) — no local font files.
