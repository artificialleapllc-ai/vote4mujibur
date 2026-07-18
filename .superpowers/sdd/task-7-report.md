# Task 7 Report: UPDATE_GUIDE.md Creation

## Deliverable

Created `/Users/suhail/vote4mujibur/UPDATE_GUIDE.md` containing the Bangla maintainer guide for adding new activities to the website. The guide was extracted verbatim from the fenced markdown block in task-7-brief.md (lines 13-63), preserving all Bangla text, formatting, and the 4-space-indented JavaScript code sample.

## Verification Results

### Check a: Line count
```
$ wc -l UPDATE_GUIDE.md
      51 UPDATE_GUIDE.md
```
Result: PASS — 51 lines matches the fenced block (lines 13-63 inclusive)

### Check b: Content diff against source
```
$ diff <(sed -n '13,63p' .superpowers/sdd/task-7-brief.md) UPDATE_GUIDE.md
(no output)
```
Result: PASS — Files match exactly, no differences

### Check c: Activities array length
```
$ node -e "$(cat activities.js); console.log(activities.length)"
6
```
Result: PASS — activities array contains 6 entries as expected

## Commit

```
[mp-redesign 0022579] docs: Bangla update guide for the media cell
 1 file changed, 51 insertions(+)
 create mode 100644 UPDATE_GUIDE.md
```

Commit SHA: `0022579`

## Fix: form label language visibility (final QA)

### The Defect

The opinion form's language-pair labels (`.bn-content` / `.en-content` nested in `.form-group`) were not properly hidden in their inactive language mode. The cascade rule `.form-group label { display: block; }` (line 554, specificity 0,1,1) was beating `.en-content { display: none; }` (line 52, specificity 0,1,0), causing 11 visible `.en-content` elements in the `#opinion` form when in Bangla mode (measured by headless-browser QA). This defect predated the redesign and was present in the pre-redesign baseline (commit 5098e7f).

### The Fix

Added three compound-class CSS rules immediately after line 560 (the closing brace of `.form-group label { ... }`), following the established cascade-scoping idiom used elsewhere in the stylesheet (e.g., `.cta-button.en-content`):

**Lines 561-563:**
```css
.form-group label.en-content { display: none; }
body.lang-en .form-group label.bn-content { display: none; }
body.lang-en .form-group label.en-content { display: block; }
```

These rules have the correct specificity:
- `.form-group label.en-content`: specificity (0,2,1), beats the original (0,1,1)
- `body.lang-en` variants: beat both for English mode; `block` matches the labels' intended display

### Verification Results

**Check 1: Grep for the three new rules**
```
$ grep -n 'form-group label.en-content\|form-group label.bn-content' index.html
561:        .form-group label.en-content { display: none; }
562:        body.lang-en .form-group label.bn-content { display: none; }
563:        body.lang-en .form-group label.en-content { display: block; }
```
Result: PASS — all three rules present exactly once, at lines 561-563 in sequence.

**Check 2: Context (8 surrounding lines)**
```
557:            margin-bottom: 0.5rem;
558:            color: var(--dark-green);
559:            font-size: 1.1rem;
560:        }
561:        .form-group label.en-content { display: none; }
562:        body.lang-en .form-group label.bn-content { display: none; }
563:        body.lang-en .form-group label.en-content { display: block; }
564:        .required {
565:            color: var(--red);
```
Result: PASS — rules are inside the main `<style>` block (no `@media` wrapper).

**Check 3: Other label display rules**
```
$ grep -n 'label {' index.html
97:        .lang-label { 
161:        .nav-toggle-label {
178:        .nav-toggle:checked + .nav-toggle-label {
257:        .stat-label { color: var(--ink-soft); font-size: 0.9rem; }
554:        .form-group label {
619:        .upload-label {
877:            .lang-label { font-size: 0.8rem; }
881:            .nav-toggle-label { display: flex; }
926:            .lang-label { font-size: 0.8rem; }
```
Result: PASS — only `.nav-toggle-label` (line 161, sets `display: none`) could theoretically interfere, but it is for a different element class and cannot re-override `.form-group label`. All other label rules do not set `display`. No conflicts detected.

## Controller final QA (headless Chrome, tmp-dir copies, repo tree untouched)

All 9 checklist items pass at HEAD cefe7d2:
1. Filters: সব=6, জনসংযোগ=2, বক্তব্য ও সমাবেশ=1, উন্নয়ন ও ত্রাণ=2, সংসদ=1 — every card tag matches its chip.
2. Load-more: 11 temp entries → 9 initial + visible button; click → 11 cards, button hides.
3. Lightbox: temp 2nd photo → counter "১ / ২"→"২ / ২", both arrows visible, FB link shown, backdrop click closes (Escape verified in Task 5).
4. Live Hero: shows 2026-07-10 entry; temp 2026-07-14 entry → hero swaps to it; removed after.
5. Edge cases: unknown category → অন্যান্য chip + tag; missing photo file (nai.jpg) → no JS error, placeholder path; broken activities.js → 0 cards + "কার্যক্রম লোড করা যাচ্ছে না…" rendered, opinion form + rest of site intact.
6. BN/EN toggle: instrumented survey across header/hero-identity/stats/about/commitments/videos/opinion/contact/footer — clean in both modes AFTER fix cefe7d2 (pre-fix: 11 visible .en-content in #opinion during BN mode; root cause .form-group label specificity (0,1,1) > .en-content (0,1,0); defect present in pre-redesign baseline 5098e7f).
7. Mobile: at 500px (headless new-mode minimum viewport; 390px windows render at 500 and crop screenshots) scrollWidth == clientWidth (no horizontal scroll); lightbox modal fits (left 16, right 484) with close button in view. True ≤480px check needs a real device at handoff.
8. Console: zero errors/console.error across load + all scripted interactions in both QA runs.
9. node -e activities.length → 6.

Known cosmetic carry-overs (final review triage): intra-element text wrapping in about/qualifications on small screens (pre-existing), duplicate 768px media block, ledger items (a)-(f).
