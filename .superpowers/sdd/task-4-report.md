# Task 4 Report: Live Hero + constituency stats band + shared activity helpers

## Status: DONE

## What was found / done

On starting, `index.html` already had an **uncommitted working-tree diff** implementing this entire task (hero markup, CSS, JS helpers, script tag) — apparently from a prior session that didn't commit. I verified every piece against `.superpowers/sdd/task-4-brief.md` line-by-line rather than trusting it blindly, found it matched, and committed it.

- Step 1 (script tag): `<script src="activities.js"></script>` at index.html:1362, immediately before the main inline `<script>` at line 1363. Confirmed via `grep -n '<script'` — correct order.
- Step 2 (hero markup): `<section class="hero">` (line 899) replaced with the hero-grid + hero-identity + `#heroLatest` markup, followed by `<section class="stats-band">` with the three stat tiles (২টি / ১৬টি / ৪,৬৪,৬৩১). Byte-diffed against the brief — identical except one canonically-equivalent Unicode variant (see Concerns).
- Step 3 (CSS): old `.hero` green-gradient/centered rules, `.hero::before`, `@keyframes heroFloat`, `.hero h1`, `.hero .subtitle`, `.symbol-logo`, and the old `.cta-button` (which used `var(--red)`) were all removed. New rules (`.hero`, `.hero-grid`, `.hero-photo`, `.hero-identity h1`, `.hero-role`, `.hero-quote`, `.cta-button` (green), `.hero-latest .latest-card`, `.latest-badge`, `.latest-body`, `.stats-band`, `.stats-grid`, `.stat-tile`, `.stat-value`, `.stat-label`, and the mobile media query) inserted verbatim, matching the brief.
- Step 4 (JS helpers): `ACTIVITY_CATEGORIES`, `OTHER_CATEGORY`, `PLACEHOLDER_IMG`, `toBnDigits`, `formatBanglaDate`, `normalizeActivity`, `getActivities`, and the `renderHeroLatest` IIFE inserted at the top of the main inline `<script>`, before the existing "Language Toggle Functionality" handler. Names/signatures match the brief exactly (verified byte-for-byte for the JS block).
- Step 6 (commit): `git add index.html && git commit -m "feat: live hero with latest activity and constituency stats band"` → commit `50a0399`.

## Verification performed

1. **Syntax check**: extracted the inline script body (index.html lines 1364–1614, between the `<script src="activities.js">` tag and the closing `</script>`/`</body>`) to `/Users/suhail/.claude/jobs/ff7b756f/tmp/main.js`. `node --check` → passed clean.

2. **Unit checks** (combined `activities.js` + the extracted helper block, run in Node with `assert`):
   ```
   formatBanglaDate('2026-07-12') === '১২ জুলাই ২০২৬'   → PASS
   formatBanglaDate('12-07-2026') === '12-07-2026'      → PASS
   toBnDigits(464631) === '৪৬৪৬৩১'                       → PASS
   getActivities()[0].title === (2026-07-10 entry title) → PASS
     both = "গোদাগাড়ীতে গণসংযোগ ও মতবিনিময়"
   normalizeActivity({category:'ভুল'}).category === 'অন্যান্য' → PASS
   normalizeActivity({}).dateKey === '0000-00-00'             → PASS
   → "ALL ASSERTIONS PASSED"
   ```
   Also ran `getActivities()` with the `activities` global entirely absent (simulating the "rename activities.js" browser test from Step 5): returned `null` with no exception — matches the brief's fallback design (`typeof activities === 'undefined'` guard doesn't throw on an undeclared identifier), and `renderHeroLatest` bails out early in that case, leaving the hero identity-only.

3. **HTTP + curl**: served the directory with `python3 -m http.server 8000`, fetched `index.html`, confirmed presence of `hero-grid` (3×), `heroLatest` (2×), `stats-band` (2×), `২টি`, `১৬টি`, `৪,৬৪,৬৩১` (1× each), and that `<script src="activities.js">` (line 1362) precedes the main inline `<script>` (line 1363).

4. **Grep sweeps**: no leftover old hero rules (`symbol-logo`, `hero-symbol`, `heroFloat`, `margin-top: 60px`, `.hero .subtitle`, `.hero h1` → zero matches). Exactly one `.cta-button {` definition in the whole file (the new green one).

## Concerns

- **Non-blocking Unicode nuance**: one Bangla phrase in the hero (`রাজশাহী-১ (গোদাগাড়ী-তানোর)`) uses a precomposed `ড়` (U+09DC, RRA) in the actual file where the brief's markdown source has the decomposed form `ড` + nukta (U+09A1 + U+09BC). Verified with Python `unicodedata.normalize`: both NFC-normalize and NFD-normalize to the same string — this is a genuine canonical-equivalence pair, not a typo or content divergence, and renders identically in any Unicode-correct browser/font. No action taken since it's cosmetic and (per the diff) this exact phrase already existed pre-task in other parts of the file.
- **Rendered layout not visually inspected** (no browser available in this environment) — the brief's Step 5 asks for a manual reload/toggle/rename-activities.js check. Structural/data correctness is verified per above, but actual visual appearance (grid alignment, mobile breakpoint, hero card layout, EN/BN toggle visual behavior) should get a human/browser look before merging.
- Untracked files in the repo (`.DS_Store` files, a screenshot PNG with a Bangla filename) were left alone and not included in the commit — unrelated to this task.

## Commit
`50a0399` — "feat: live hero with latest activity and constituency stats band" (index.html only)

## Fix: CSS cascade specificity (post-review)

**Defect**: The `.container` rule (line 318, `max-width: 1200px`) appears later in the stylesheet than `.hero-grid` (line 186, `max-width: 1100px`) and `.stats-grid` (line 238, `max-width: 900px`). Equal specificity + later position means `.container`'s 1200px wins, causing the hero and stats band to render wider than designed.

**Changes made**:
1. Line 186: `.hero-grid` → `.hero .hero-grid` (specificity: 0,1,1 → 0,2,1)
2. Line 238: `.stats-grid` → `.stats-band .stats-grid` (specificity: 0,1,1 → 0,2,1)

**Verification**:

1. New selectors exist exactly once each:
   ```
   $ grep -n 'hero .hero-grid\|stats-band .stats-grid' index.html
   186:        .hero .hero-grid {
   238:        .stats-band .stats-grid {
   ```

2. No remaining bare top-level rules (only @media rules remain, which don't define max-width/margin):
   ```
   $ grep -nE '^\s*\.hero-grid\s*\{|^\s*\.stats-grid\s*\{' index.html
   257:            .hero-grid { grid-template-columns: 1fr; gap: 1.5rem; }
   259:            .stats-grid { grid-template-columns: repeat(3, 1fr); gap: 0.5rem; }
   ```
   (Lines 257 & 259 are indented inside @media block at line 256, so they are media queries, not top-level.)

3. Parent elements confirmed in markup:
   ```
   $ grep -n '<section class="hero">\|<section class="stats-band">' index.html
   899:<section class="hero">
   917:<section class="stats-band">
   ```
   - Line 900: `<div class="container hero-grid">` is a direct child of line 899's `<section class="hero">`
   - Line 918: `<div class="container stats-grid">` is a direct child of line 917's `<section class="stats-band">`
