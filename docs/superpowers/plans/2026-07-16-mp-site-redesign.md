# MP Site Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Transform the vote4mujibur campaign site into the official site of Prof. Mujibur Rahman, MP for Rajshahi-1, centered on a manually curated, filterable activity timeline.

**Architecture:** Progressive in-place transformation of the existing single-file `index.html` (no rewrite-from-blank), plus a new `activities.js` data file (the only file the non-technical media cell edits) and an `activity-photos/` folder. No build step; plain HTML/CSS/JS served statically.

**Tech Stack:** Vanilla HTML/CSS/JS, Google Fonts (existing), Supabase REST (existing opinion form, untouched), YouTube embeds (existing). Node.js used only as a dev-time syntax checker.

**Spec:** `docs/superpowers/specs/2026-07-16-mp-site-redesign-design.md` — read it before starting.

## Global Constraints

- No build step, no new runtime dependencies, no framework. GitHub-web-UI-editable files only.
- Single accent green `#00703c`; tints `#e6f2ec`; ink `#0b1f16`; secondary ink `#5a6e64`; background `#f7f9f8`; surfaces white; radius `10px`; shadow `0 2px 8px rgba(11,31,22,.08)`.
- Valid activity categories, exact strings: `জনসংযোগ`, `বক্তব্য ও সমাবেশ`, `উন্নয়ন ও ত্রাণ`, `সংসদ`. Unknown → displayed as `অন্যান্য`.
- Activity dates are ISO `YYYY-MM-DD` in ASCII digits in data; always displayed as Bangla (`১২ জুলাই ২০২৬`).
- Activities are Bangla-only (no `en-content` pair). All other sections keep the existing BN/EN toggle pairs (`bn-content`/`en-content` classes).
- A malformed `activities.js` must never blank the site: it loads in its own `<script>` tag and every consumer guards `typeof activities === 'undefined'`.
- Keep working: language toggle, mobile nav toggle, smooth scrolling, Supabase opinion form (URL and anon key unchanged), Vercel insights script, `SolaimanLipi` font, favicon.
- Commit locally after each task. **Do not `git push` until the final task's QA checklist passes** — the remote is the live site.
- Verification is manual browser testing (no JS test framework exists and adding one violates YAGNI for a static page) plus `node -e` syntax checks. Every verify step lists exact expected observations. Serve locally with `python3 -m http.server 8000` from the repo root and open `http://localhost:8000`.

## File Structure

- `index.html` — the entire site. Modified by Tasks 2–6. Owned by developers.
- `activities.js` — **new.** Global `const activities = [...]` + Bangla how-to header comment. The only file the media cell edits. Created in Task 1.
- `activity-photos/` — **new.** Flat folder of activity images, named `YYYY-MM-DD-<slug>-<n>.jpg`. Created in Task 1.
- `UPDATE_GUIDE.md` — **new.** Bangla step-by-step guide for the media cell. Created in Task 7.
- Existing assets (`profile.jpg`, `istehar_*.jpg`, `Manifesto_*.pdf`, `SolaimanLipi.ttf`, `দাঁড়িপাল্লায় ভোট দিন.png`) stay where they are.

Reference facts gathered during planning (verified 2026-07-16):

- Voter totals from the old results feed: গোদাগাড়ী ২৯৪,২৩০ + তানোর ১৭০,৪০১ = **৪,৬৪,৬৩১**
- Constituency: ২টি উপজেলা (গোদাগাড়ী, তানোর), ১৬টি ইউনিয়ন (গোদাগাড়ী ৯ + তানোর ৭) — **confirm the ১৬ with the site owner at handoff**
- YouTube embeds to keep: `wtzGoNASEOM`, `80kLqX-sSwA`, `63W6Zy2LdFw`
- Facebook pages: `https://www.facebook.com/ProfMujib.Official` and `https://www.facebook.com/mrmedia11/` (মিডিয়া সেল)

---

### Task 1: Safety tag + activities data layer

**Files:**
- Create: `activities.js`
- Create: `activity-photos/` (6 seed images copied from existing repo photos)

**Interfaces:**
- Produces: global `const activities` — array of `{date: string, category: string, title: string, description: string, photos: string[], link?: string}`. Tasks 4 and 5 consume it. Seed photos live at `activity-photos/sample-1.jpg` … `sample-6.jpg`.

- [ ] **Step 1: Tag the campaign site for reference and rollback**

```bash
git tag campaign-site
```

Any later need to inspect removed content: `git show campaign-site:index.html`.

- [ ] **Step 2: Create the photo folder with seed images**

Seed photos are stand-ins copied from existing campaign images; the owner will replace them with real Facebook-page photos.

```bash
mkdir activity-photos
cp image_1.jpg activity-photos/sample-1.jpg
cp image_2.jpg activity-photos/sample-2.jpg
cp image_3.jpg activity-photos/sample-3.jpg
cp vote_1.jpg  activity-photos/sample-4.jpg
cp vote_2.jpg  activity-photos/sample-5.jpg
cp vote_3.jpg  activity-photos/sample-6.jpg
```

- [ ] **Step 3: Create `activities.js`**

Create the file with exactly this content:

```js
/* ============================================================
   কার্যক্রম তালিকা — অধ্যাপক মুজিবুর রহমান, এমপি (রাজশাহী-১)
   ============================================================

   নতুন কার্যক্রম যোগ করার নিয়ম:

   ১. ছবি আপলোড করুন activity-photos/ ফোল্ডারে
      (ফাইলের নাম ইংরেজিতে দিন, যেমন: 2026-07-15-sova-1.jpg)

   ২. নিচের নমুনাটি কপি করে "const activities = [" লাইনের
      ঠিক নিচে পেস্ট করুন এবং তথ্য বসান:

    {
        date: "2026-07-15",                  // বছর-মাস-দিন (ইংরেজি সংখ্যায়)
        category: "জনসংযোগ",                  // নিচের তালিকা থেকে হুবহু কপি করুন
        title: "এখানে শিরোনাম লিখুন",
        description: "এখানে বিস্তারিত বিবরণ লিখুন।",
        photos: ["activity-photos/2026-07-15-sova-1.jpg"],
        link: ""                             // ফেসবুক পোস্টের লিংক (ঐচ্ছিক)
    },

   ৩. category হিসেবে শুধু এই চারটির একটি লিখুন:
      "জনসংযোগ"
      "বক্তব্য ও সমাবেশ"
      "উন্নয়ন ও ত্রাণ"
      "সংসদ"

   সতর্কতা: প্রতিটি লাইনের শেষের কমা (,) এবং উদ্ধৃতি চিহ্ন ("...")
   ঠিক রাখুন। তারিখের ক্রম নিয়ে ভাবতে হবে না — ওয়েবসাইট নিজেই
   নতুন থেকে পুরাতন সাজিয়ে নেয়।
   ============================================================ */

const activities = [
    {
        date: "2026-07-10",
        category: "জনসংযোগ",
        title: "গোদাগাড়ীতে গণসংযোগ ও মতবিনিময়",
        description: "গোদাগাড়ী উপজেলার বিভিন্ন এলাকায় সাধারণ মানুষের সঙ্গে মতবিনিময় করেন এবং তাঁদের সমস্যার কথা শোনেন অধ্যাপক মুজিবুর রহমান এমপি।",
        photos: ["activity-photos/sample-1.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    },
    {
        date: "2026-07-05",
        category: "উন্নয়ন ও ত্রাণ",
        title: "তানোরে গ্রামীণ সড়ক সংস্কার কাজ পরিদর্শন",
        description: "তানোর উপজেলায় চলমান গ্রামীণ সড়ক সংস্কার কাজের অগ্রগতি সরেজমিনে পরিদর্শন করেন এবং দ্রুত ও মানসম্মত কাজ সম্পন্ন করার নির্দেশ দেন।",
        photos: ["activity-photos/sample-2.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    },
    {
        date: "2026-06-28",
        category: "সংসদ",
        title: "জাতীয় সংসদে রাজশাহী-১ এর উন্নয়ন নিয়ে বক্তব্য",
        description: "জাতীয় সংসদের অধিবেশনে গোদাগাড়ী-তানোরের রাস্তাঘাট, শিক্ষা ও কৃষি খাতের উন্নয়নে বরাদ্দ বৃদ্ধির দাবি তুলে ধরেন।",
        photos: ["activity-photos/sample-3.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    },
    {
        date: "2026-06-20",
        category: "বক্তব্য ও সমাবেশ",
        title: "কাঁকনহাটে কর্মী সমাবেশে প্রধান অতিথির বক্তব্য",
        description: "কাঁকনহাট পৌরসভায় আয়োজিত কর্মী সমাবেশে প্রধান অতিথি হিসেবে বক্তব্য রাখেন এবং এলাকার উন্নয়নে সকলকে ঐক্যবদ্ধভাবে কাজ করার আহ্বান জানান।",
        photos: ["activity-photos/sample-4.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    },
    {
        date: "2026-06-12",
        category: "উন্নয়ন ও ত্রাণ",
        title: "অসহায় পরিবারের মাঝে ত্রাণ সামগ্রী বিতরণ",
        description: "গোদাগাড়ী ও তানোরের দুস্থ ও অসহায় পরিবারের মাঝে খাদ্য ও ত্রাণ সামগ্রী বিতরণ করা হয়।",
        photos: ["activity-photos/sample-5.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    },
    {
        date: "2026-06-05",
        category: "জনসংযোগ",
        title: "মুন্ডুমালা পৌরসভায় বিভিন্ন প্রতিষ্ঠান পরিদর্শন",
        description: "মুন্ডুমালা পৌরসভার স্কুল, মাদ্রাসা ও স্বাস্থ্যকেন্দ্র পরিদর্শন করেন এবং শিক্ষক, শিক্ষার্থী ও স্থানীয়দের সঙ্গে কথা বলেন।",
        photos: ["activity-photos/sample-6.jpg"],
        link: "https://www.facebook.com/mrmedia11/"
    }
];
```

**Note:** these six entries are drafted placeholders in the style of the মিডিয়া সেল Facebook page. Flag at handoff: the owner replaces titles/descriptions/photos with real posts.

- [ ] **Step 4: Verify the file parses and has 6 entries**

```bash
node -e "$(cat activities.js); console.log(activities.length, 'activities,', new Set(activities.map(a=>a.category)).size, 'categories')"
```

Expected output: `6 activities, 4 categories`

- [ ] **Step 5: Commit**

```bash
git add activities.js activity-photos/
git commit -m "feat: add activities data layer with seed entries"
```

---

### Task 2: Modern Official base — design tokens, page metadata, header/nav

**Files:**
- Modify: `index.html` — the `:root` CSS block (~line 21), `<title>`/`<head>` (~lines 5–10), the `<header>`/`<nav>` block (search for `<!-- Header -->`), and header/nav CSS rules.

**Interfaces:**
- Consumes: nothing from other tasks.
- Produces: CSS custom properties (`--green`, `--green-dark`, `--green-tint`, `--ink`, `--ink-soft`, `--bg`, `--surface`, `--border`, `--radius`, `--shadow`, `--red`) that all later tasks' CSS uses; nav anchor targets `#activities`, `#about`, `#commitments`, `#videos`, `#contact` that later tasks must give to their sections.

- [ ] **Step 1: Replace the design tokens**

Find the current `:root` block:

```css
:root {
    --green: #006633;
    --dark-green: #004422;
    --red: #DC143C;
    --white: #ffffff;
    --light-gray: #f5f5f5;
}
```

Replace with:

```css
:root {
    --green: #00703c;
    --green-dark: #00502b;
    --green-tint: #e6f2ec;
    --ink: #0b1f16;
    --ink-soft: #5a6e64;
    --bg: #f7f9f8;
    --surface: #ffffff;
    --border: #e3eae6;
    --radius: 10px;
    --shadow: 0 2px 8px rgba(11,31,22,.08);
    --red: #b3261e; /* form validation marks only — never decorative */
    /* legacy aliases so untouched rules keep working mid-transformation */
    --dark-green: #00502b;
    --white: #ffffff;
    --light-gray: #f7f9f8;
}
```

Also update `body` to `background: var(--bg); color: var(--ink);` and change the lang-switch checked slider from red to green: find `input:checked + .slider { background-color: var(--red); }` and change to `var(--green-dark)`.

- [ ] **Step 2: Update page metadata**

- `<title>` → `অধ্যাপক মুজিবুর রহমান — সংসদ সদস্য, রাজশাহী-১`
- Add after the viewport meta: `<meta name="description" content="অধ্যাপক মুজিবুর রহমান, সংসদ সদস্য, রাজশাহী-১ (গোদাগাড়ী-তানোর) — কার্যক্রম, অঙ্গীকার ও যোগাযোগ।">`

- [ ] **Step 3: Rewrite the nav links**

In the `<ul class="nav-links">` block, replace the six `<li>` items with:

```html
<li><a href="#activities" class="bn-content">কার্যক্রম</a><a href="#activities" class="en-content">Activities</a></li>
<li><a href="#about" class="bn-content">পরিচিতি</a><a href="#about" class="en-content">About</a></li>
<li><a href="#commitments" class="bn-content">অঙ্গীকার</a><a href="#commitments" class="en-content">Commitments</a></li>
<li><a href="#videos" class="bn-content">ভিডিও</a><a href="#videos" class="en-content">Videos</a></li>
<li><a href="#contact" class="bn-content">যোগাযোগ</a><a href="#contact" class="en-content">Contact</a></li>
```

In the `.logo` span, change `প্রফেসর মুজিবুর রহমান` → `অধ্যাপক মুজিবুর রহমান` (bn) and keep the English span. Keep the base64 logo img, `navToggle` checkbox, and hamburger label untouched.

- [ ] **Step 4: Restyle the header to Modern Official**

Find the `header {` CSS rule (green gradient) and replace with:

```css
header {
    background: var(--surface);
    border-bottom: 3px solid var(--green);
    color: var(--ink);
    padding: 0.75rem 0;
    position: fixed;
    width: 100%;
    top: 0;
    z-index: 1000;
    box-shadow: var(--shadow);
}
```

Then fix contrast in dependent rules (nav link colors were white-on-green): set `.nav-links a { color: var(--ink); }`, hover/active color `var(--green)`, `.logo span { color: var(--ink); }`, and hamburger bars (`.nav-toggle-label span`) `background: var(--ink)`. On mobile, the dropdown `.nav-links` background becomes `var(--surface)` with `border-top: 1px solid var(--border)` if it was green. Check each rule that referenced the old header colors.

- [ ] **Step 5: Verify in browser**

Run: `python3 -m http.server 8000` → open `http://localhost:8000`.
Expected: white sticky header with green underline; five Bangla nav links; tab title shows এমপি framing; language toggle still flips BN/EN everywhere; mobile width (devtools ≤ 480px) hamburger opens a readable menu. Nav links 404 to nothing (targets don't exist yet) — acceptable until Tasks 4–6.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: Modern Official design tokens, metadata, and header"
```

---

### Task 3: Remove campaign-era sections and dead code

**Files:**
- Modify: `index.html` — delete blocks listed below plus their CSS and JS.

**Interfaces:**
- Consumes: nothing.
- Produces: a page with the old hero as the only remaining pre-content section, ready for Task 4 to replace.

- [ ] **Step 1: Delete campaign-era HTML blocks**

Delete each of these (locate by the comment or `id`; each runs to its closing `</section>`/`</div>`):

1. `<!-- Top Banner Slider -->` — `<section class="top-banner">` (ishtehar slider; the images stay in the repo for Task 6)
2. `<section class="result-ticker-section">` — the "এক নজরে রাজশাহী-১" results summary cards **and** the `resultTable` wrapper
3. `<!-- Vote Day Update Section -->` — `<section id="vote-day" class="vote-update">`
4. `<section id="news" class="section">` — campaign news cards (including commented-out ones)
5. The emergency-contact popup markup (search `emergencyPopup`) and the vote-image modal markup (search `voteModal`) if present as standalone elements

- [ ] **Step 2: Delete the matching JavaScript**

In the trailing `<script>` block, delete:

- The top-banner slider logic (search `topBannerSlider`, `topBannerDots`)
- The entire `loadResultTable` IIFE (search `loadResultTable`) including the `DATA_URL` constant and `resultTableCache_v2` cache code
- The vote modal handlers (search `voteModal`)
- The emergency popup handlers (search `emergencyPopup`, `popupClose`)

Keep: language toggle, nav toggle, smooth scrolling `anchor` loop, `setupFileUpload`, and the Supabase `opinionForm` submit handler.

- [ ] **Step 3: Delete the matching CSS**

Remove rule blocks whose selectors mention: `top-banner`, `result-ticker`, `result-summary`, `summary-card`, `result-table`, `vote-update`, `vote-grid`, `vote-card`, `vote-ticker`, `vote-caption`, `news-grid`, `news-card`, `news-image`, `news-content`, `news-title`, `news-tag`, `news-date`, `news-description`, `read-more`, `emergency-popup`, `vote-modal`. Grep to confirm zero remaining references:

```bash
grep -cE 'top-banner|result-ticker|result-summary|resultTable|vote-update|vote-grid|voteModal|emergencyPopup|news-card' index.html
```

Expected: `0`

- [ ] **Step 4: Verify in browser**

Reload `http://localhost:8000`. Expected: page shows header → old hero → manifesto → about → … with no banner slider, no results, no vote-day, no news section; browser console shows **zero errors**; language toggle and mobile nav still work.

- [ ] **Step 5: Commit**

```bash
git add index.html
git commit -m "feat: remove campaign-era sections (banner, results, vote day, news)"
```

---

### Task 4: Live Hero + constituency stats band + shared activity helpers

**Files:**
- Modify: `index.html` — replace `<section class="hero">`; add stats band after it; add `<script src="activities.js"></script>` before the main inline script; add shared JS helpers.

**Interfaces:**
- Consumes: global `activities` array from Task 1.
- Produces (Task 5 relies on these exact names, defined in the main inline `<script>`):
  - `toBnDigits(value) → string` — ASCII digits to Bangla digits
  - `formatBanglaDate(iso) → string` — `"2026-07-12"` → `"১২ জুলাই ২০২৬"`; returns input unchanged if not ISO
  - `normalizeActivity(raw) → {dateKey, dateLabel, category, title, description, photos, link}` — applies category/photo/date fallbacks
  - `getActivities() → array|null` — null when `activities` is missing/invalid; otherwise normalized entries sorted newest-first
  - `ACTIVITY_CATEGORIES` (array of the 4 valid strings), `OTHER_CATEGORY` (`'অন্যান্য'`), `PLACEHOLDER_IMG` (data-URI string)
  - Hero markup contains `<div class="hero-latest" id="heroLatest" hidden></div>`

- [ ] **Step 1: Load activities.js**

Immediately before the main inline `<script>` near the end of `<body>`, add:

```html
<script src="activities.js"></script>
```

(Own tag = a syntax error there cannot kill the main script.)

- [ ] **Step 2: Replace the hero section**

Replace the whole `<section class="hero">…</section>` with:

```html
<!-- Live Hero -->
<section class="hero">
    <div class="container hero-grid">
        <div class="hero-identity">
            <img src="profile.jpg" alt="অধ্যাপক মুজিবুর রহমান" class="hero-photo">
            <h1 class="bn-content">অধ্যাপক মুজিবুর রহমান</h1>
            <h1 class="en-content">Professor Mujibur Rahman</h1>
            <p class="hero-role bn-content">সংসদ সদস্য · রাজশাহী-১ (গোদাগাড়ী-তানোর)</p>
            <p class="hero-role en-content">Member of Parliament · Rajshahi-1 (Godagari–Tanore)</p>
            <p class="hero-quote bn-content">“জনগণের সেবাই আমার অঙ্গীকার”</p>
            <p class="hero-quote en-content">“Serving the people is my commitment”</p>
            <a href="#activities" class="cta-button bn-content">সর্বশেষ কার্যক্রম দেখুন</a>
            <a href="#activities" class="cta-button en-content">See Latest Activities</a>
        </div>
        <div class="hero-latest" id="heroLatest" hidden></div>
    </div>
</section>

<!-- Constituency stats band -->
<section class="stats-band">
    <div class="container stats-grid">
        <div class="stat-tile"><span class="stat-value">২টি</span><span class="stat-label bn-content">উপজেলা</span><span class="stat-label en-content">Upazilas</span></div>
        <div class="stat-tile"><span class="stat-value">১৬টি</span><span class="stat-label bn-content">ইউনিয়ন</span><span class="stat-label en-content">Unions</span></div>
        <div class="stat-tile"><span class="stat-value">৪,৬৪,৬৩১</span><span class="stat-label bn-content">মোট ভোটার</span><span class="stat-label en-content">Total Voters</span></div>
    </div>
</section>
```

- [ ] **Step 3: Add hero/stats CSS**

Replace the old `.hero` rules (green gradient, centered) with:

```css
.hero {
    background: var(--surface);
    padding: 7.5rem 0 3rem;   /* clears the fixed header */
}
.hero-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 2.5rem;
    align-items: center;
    max-width: 1100px;
    margin: 0 auto;
    padding: 0 1.5rem;
}
.hero-photo {
    width: 110px; height: 110px;
    object-fit: cover;
    border-radius: 50%;
    border: 3px solid var(--green);
    margin-bottom: 1rem;
}
.hero-identity h1 { color: var(--ink); font-size: 2.2rem; line-height: 1.3; }
.hero-role { color: var(--ink-soft); margin: 0.25rem 0 0.75rem; }
.hero-quote { color: var(--ink); font-weight: 600; margin-bottom: 1.25rem; }
.cta-button {
    display: inline-block;
    background: var(--green);
    color: #fff;
    padding: 0.7rem 1.8rem;
    border-radius: 30px;
    text-decoration: none;
    font-weight: 600;
    transition: background 0.2s;
}
.cta-button:hover { background: var(--green-dark); }

.hero-latest .latest-card {
    background: var(--surface);
    border-radius: var(--radius);
    box-shadow: 0 4px 16px rgba(11,31,22,.12);
    overflow: hidden;
    cursor: pointer;
}
.latest-badge {
    display: block;
    background: var(--green);
    color: #fff;
    font-size: 0.8rem;
    font-weight: 700;
    padding: 0.3rem 0.9rem;
}
.latest-card img { width: 100%; height: 220px; object-fit: cover; display: block; }
.latest-body { padding: 0.9rem 1.1rem; }
.latest-body h3 { color: var(--ink); font-size: 1.15rem; margin-bottom: 0.3rem; }
.latest-meta { color: var(--ink-soft); font-size: 0.85rem; }

.stats-band { background: var(--green-tint); padding: 1.5rem 0; }
.stats-grid {
    display: grid;
    grid-template-columns: repeat(3, 1fr);
    gap: 1rem;
    max-width: 900px;
    margin: 0 auto;
    padding: 0 1.5rem;
}
.stat-tile {
    background: var(--surface);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    text-align: center;
    padding: 1rem 0.5rem;
}
.stat-value { display: block; color: var(--green); font-size: 1.6rem; font-weight: 700; }
.stat-label { color: var(--ink-soft); font-size: 0.9rem; }

@media (max-width: 768px) {
    .hero-grid { grid-template-columns: 1fr; gap: 1.5rem; }
    .hero-identity h1 { font-size: 1.7rem; }
    .stats-grid { grid-template-columns: repeat(3, 1fr); gap: 0.5rem; }
    .stat-value { font-size: 1.2rem; }
}
```

Note: `.stat-label` spans use `bn-content`/`en-content`, which the base stylesheet sets to `display:block/none` — that already works for the toggle, no extra CSS needed.

- [ ] **Step 4: Add the shared activity helpers + hero renderer**

At the **top** of the main inline `<script>` (before existing handlers), add:

```js
// ---- Activity data helpers (shared by hero + activities section) ----
const ACTIVITY_CATEGORIES = ['জনসংযোগ', 'বক্তব্য ও সমাবেশ', 'উন্নয়ন ও ত্রাণ', 'সংসদ'];
const OTHER_CATEGORY = 'অন্যান্য';
const PLACEHOLDER_IMG = 'data:image/svg+xml;utf8,' + encodeURIComponent(
    '<svg xmlns="http://www.w3.org/2000/svg" width="640" height="400">' +
    '<rect width="100%" height="100%" fill="#e6f2ec"/>' +
    '<text x="50%" y="50%" font-size="36" fill="#5a6e64" text-anchor="middle" dominant-baseline="middle" font-family="sans-serif">ছবি নেই</text></svg>');

function toBnDigits(value) {
    return String(value).replace(/\d/g, (d) => '০১২৩৪৫৬৭৮৯'[d]);
}

function formatBanglaDate(iso) {
    const m = /^(\d{4})-(\d{2})-(\d{2})$/.exec(String(iso).trim());
    if (!m) return String(iso);
    const months = ['জানুয়ারি','ফেব্রুয়ারি','মার্চ','এপ্রিল','মে','জুন',
                    'জুলাই','আগস্ট','সেপ্টেম্বর','অক্টোবর','নভেম্বর','ডিসেম্বর'];
    const mi = parseInt(m[2], 10) - 1;
    if (mi < 0 || mi > 11) return String(iso);
    return toBnDigits(parseInt(m[3], 10)) + ' ' + months[mi] + ' ' + toBnDigits(m[1]);
}

function normalizeActivity(raw) {
    const r = raw || {};
    const dateStr = String(r.date || '').trim();
    const isIso = /^\d{4}-\d{2}-\d{2}$/.test(dateStr);
    return {
        dateKey: isIso ? dateStr : '0000-00-00',      // invalid dates sort last (list is newest-first)
        dateLabel: formatBanglaDate(dateStr),
        category: ACTIVITY_CATEGORIES.includes(String(r.category || '').trim())
            ? String(r.category).trim() : OTHER_CATEGORY,
        title: String(r.title || 'শিরোনামহীন'),
        description: String(r.description || ''),
        photos: Array.isArray(r.photos) ? r.photos.filter((p) => typeof p === 'string' && p) : [],
        link: (typeof r.link === 'string' && /^https?:\/\//.test(r.link)) ? r.link : ''
    };
}

function getActivities() {
    if (typeof activities === 'undefined' || !Array.isArray(activities)) return null;
    return activities.map(normalizeActivity)
        .sort((a, b) => b.dateKey.localeCompare(a.dateKey));
}

// ---- Live hero: newest activity ----
(function renderHeroLatest() {
    const box = document.getElementById('heroLatest');
    if (!box) return;
    const list = getActivities();
    if (!list || list.length === 0) return;   // hero stays identity-only
    const a = list[0];
    const photo = a.photos[0] || PLACEHOLDER_IMG;
    box.innerHTML =
        '<div class="latest-card">' +
        '<span class="latest-badge">● সর্বশেষ কার্যক্রম</span>' +
        '<img src="' + photo + '" alt="" onerror="this.onerror=null;this.src=PLACEHOLDER_IMG;">' +
        '<div class="latest-body"><h3></h3><p class="latest-meta"></p></div></div>';
    box.querySelector('h3').textContent = a.title;
    box.querySelector('.latest-meta').textContent = a.category + ' · ' + a.dateLabel;
    box.hidden = false;
    box.querySelector('.latest-card').addEventListener('click', () => {
        document.getElementById('activities')?.scrollIntoView({ behavior: 'smooth' });
    });
})();
```

- [ ] **Step 5: Verify in browser**

Reload. Expected: hero shows portrait + name + "সংসদ সদস্য" left, and the **2026-07-10 গণসংযোগ** entry (newest seed) as a badged card right; stats band shows ২টি / ১৬টি / ৪,৬৪,৬৩১; console error-free. Toggle EN: identity flips to English, activity card stays Bangla. Temporarily rename `activities.js` → reload → hero shows identity only, no console crash from the main script (a 404 for activities.js is fine) → rename back.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: live hero with latest activity and constituency stats band"
```

---

### Task 5: Activities section — filters, grid, load-more, lightbox

**Files:**
- Modify: `index.html` — insert the activities section directly after the stats band; add CSS; add JS after the Task 4 helpers.

**Interfaces:**
- Consumes: `getActivities()`, `formatBanglaDate` (via `dateLabel`), `ACTIVITY_CATEGORIES`, `OTHER_CATEGORY`, `PLACEHOLDER_IMG` from Task 4.
- Produces: `<section id="activities">` (nav + hero CTA target). No later task depends on its internals.

- [ ] **Step 1: Insert the section markup after the stats band**

```html
<!-- Activities -->
<section id="activities" class="activities-section">
    <div class="container">
        <h2 class="section-title bn-content">কার্যক্রম</h2>
        <h2 class="section-title en-content">Activities</h2>
        <p class="section-lead bn-content">সংসদ সদস্য হিসেবে অধ্যাপক মুজিবুর রহমানের চলমান কার্যক্রম — নতুন থেকে পুরাতন।</p>
        <p class="section-lead en-content">Ongoing activities of Professor Mujibur Rahman as MP — newest first.</p>
        <div class="activity-filters" id="activityFilters"></div>
        <div class="activity-grid" id="activityGrid"></div>
        <p class="activity-message" id="activityMessage" hidden></p>
        <div class="activity-more-wrap">
            <button type="button" class="activity-more" id="activityMore" hidden>আরও দেখুন</button>
        </div>
    </div>
</section>

<!-- Activity lightbox -->
<div class="activity-modal" id="activityModal" hidden>
    <div class="activity-modal-card" role="dialog" aria-modal="true">
        <button type="button" class="modal-close" id="modalClose" aria-label="বন্ধ করুন">✕</button>
        <div class="modal-photo-wrap">
            <button type="button" class="modal-nav" id="modalPrev" aria-label="আগের ছবি">‹</button>
            <img id="modalPhoto" src="" alt="">
            <button type="button" class="modal-nav" id="modalNext" aria-label="পরের ছবি">›</button>
            <span class="modal-photo-count" id="modalCount"></span>
        </div>
        <div class="modal-body">
            <p class="modal-meta"><span class="activity-tag" id="modalCategory"></span> <span id="modalDate"></span></p>
            <h3 id="modalTitle"></h3>
            <p id="modalDescription"></p>
            <a id="modalLink" href="" target="_blank" rel="noopener" hidden>ফেসবুকে দেখুন →</a>
        </div>
    </div>
</div>
```

- [ ] **Step 2: Add the CSS**

```css
.activities-section { background: var(--bg); padding: 3.5rem 0; }
.activity-filters { display: flex; flex-wrap: wrap; gap: 0.5rem; justify-content: center; margin: 1.25rem 0 1.75rem; }
.filter-chip {
    background: var(--surface);
    border: 1px solid var(--green);
    color: var(--green);
    border-radius: 30px;
    padding: 0.35rem 1.1rem;
    font-family: inherit;
    font-size: 0.95rem;
    cursor: pointer;
    transition: all 0.15s;
}
.filter-chip.active, .filter-chip:hover { background: var(--green); color: #fff; }

.activity-grid { display: grid; grid-template-columns: repeat(3, 1fr); gap: 1.25rem; }
.activity-card {
    background: var(--surface);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    overflow: hidden;
    cursor: pointer;
    transition: transform 0.15s, box-shadow 0.15s;
}
.activity-card:hover { transform: translateY(-3px); box-shadow: 0 6px 18px rgba(11,31,22,.14); }
.activity-card img { width: 100%; height: 180px; object-fit: cover; display: block; }
.activity-card-body { padding: 0.9rem 1.1rem; }
.activity-tag {
    display: inline-block;
    background: var(--green-tint);
    color: var(--green);
    font-size: 0.78rem;
    font-weight: 600;
    padding: 0.1rem 0.7rem;
    border-radius: 12px;
}
.activity-date { color: var(--ink-soft); font-size: 0.82rem; margin-left: 0.4rem; }
.activity-card-body h3 { color: var(--ink); font-size: 1.05rem; margin: 0.45rem 0 0.3rem; line-height: 1.45; }
.activity-excerpt {
    color: var(--ink-soft); font-size: 0.9rem;
    display: -webkit-box; -webkit-line-clamp: 2; -webkit-box-orient: vertical; overflow: hidden;
}
.activity-message { text-align: center; color: var(--ink-soft); padding: 2rem 0; }
.activity-more-wrap { text-align: center; margin-top: 1.75rem; }
.activity-more {
    background: var(--surface); border: 1px solid var(--green); color: var(--green);
    border-radius: 30px; padding: 0.55rem 2.2rem; font-family: inherit; font-size: 1rem; cursor: pointer;
}
.activity-more:hover { background: var(--green); color: #fff; }

.activity-modal {
    position: fixed; inset: 0; z-index: 2000;
    background: rgba(11,31,22,.75);
    display: flex; align-items: center; justify-content: center; padding: 1rem;
}
/* author display values override the hidden attribute — restore it explicitly */
.activity-modal[hidden], #modalLink[hidden] { display: none; }
.activity-modal-card {
    background: var(--surface); border-radius: var(--radius);
    max-width: 720px; width: 100%; max-height: 90vh; overflow-y: auto; position: relative;
}
.modal-close {
    position: absolute; top: 0.6rem; right: 0.6rem; z-index: 2;
    background: rgba(255,255,255,.9); border: none; border-radius: 50%;
    width: 34px; height: 34px; font-size: 1rem; cursor: pointer;
}
.modal-photo-wrap { position: relative; background: var(--green-tint); }
.modal-photo-wrap img { width: 100%; max-height: 55vh; object-fit: contain; display: block; }
.modal-nav {
    position: absolute; top: 50%; transform: translateY(-50%);
    background: rgba(255,255,255,.9); border: none; border-radius: 50%;
    width: 38px; height: 38px; font-size: 1.4rem; cursor: pointer;
}
.modal-nav#modalPrev { left: 0.6rem; }
.modal-nav#modalNext { right: 0.6rem; }
.modal-photo-count {
    position: absolute; bottom: 0.6rem; right: 0.8rem;
    background: rgba(11,31,22,.65); color: #fff; font-size: 0.78rem;
    padding: 0.1rem 0.6rem; border-radius: 12px;
}
.modal-body { padding: 1.1rem 1.4rem 1.5rem; }
.modal-body h3 { color: var(--ink); margin: 0.4rem 0 0.5rem; }
.modal-body p { color: var(--ink-soft); }
.modal-meta { font-size: 0.85rem; }
#modalLink { display: inline-block; margin-top: 0.8rem; color: var(--green); font-weight: 600; text-decoration: none; }

@media (max-width: 900px) { .activity-grid { grid-template-columns: repeat(2, 1fr); } }
@media (max-width: 560px) { .activity-grid { grid-template-columns: 1fr; } }
```

- [ ] **Step 3: Add the JS**

Insert directly after the Task 4 `renderHeroLatest` IIFE:

```js
// ---- Activities section ----
(function activitiesSection() {
    const grid = document.getElementById('activityGrid');
    const filtersBox = document.getElementById('activityFilters');
    const moreBtn = document.getElementById('activityMore');
    const message = document.getElementById('activityMessage');
    if (!grid) return;

    const PAGE_SIZE = 9;
    const all = getActivities();

    if (!all) {
        message.textContent = 'কার্যক্রম লোড করা যাচ্ছে না — activities.js ফাইলটি পরীক্ষা করুন';
        message.hidden = false;
        return;
    }
    if (all.length === 0) {
        message.textContent = 'এখনও কোনো কার্যক্রম যোগ করা হয়নি।';
        message.hidden = false;
        return;
    }

    const hasOther = all.some((a) => a.category === OTHER_CATEGORY);
    const chipLabels = ['সব'].concat(ACTIVITY_CATEGORIES, hasOther ? [OTHER_CATEGORY] : []);
    let currentFilter = 'সব';
    let visibleCount = PAGE_SIZE;

    chipLabels.forEach((label) => {
        const b = document.createElement('button');
        b.type = 'button';
        b.className = 'filter-chip' + (label === 'সব' ? ' active' : '');
        b.textContent = label;
        b.addEventListener('click', () => {
            currentFilter = label;
            visibleCount = PAGE_SIZE;
            filtersBox.querySelectorAll('.filter-chip').forEach((c) => c.classList.toggle('active', c === b));
            render();
        });
        filtersBox.appendChild(b);
    });

    function filtered() {
        return currentFilter === 'সব' ? all : all.filter((a) => a.category === currentFilter);
    }

    function render() {
        const list = filtered();
        grid.innerHTML = '';
        list.slice(0, visibleCount).forEach((a) => {
            const card = document.createElement('div');
            card.className = 'activity-card';
            const img = document.createElement('img');
            img.src = a.photos[0] || PLACEHOLDER_IMG;
            img.alt = a.title;
            img.loading = 'lazy';
            img.onerror = function () { this.onerror = null; this.src = PLACEHOLDER_IMG; };
            const body = document.createElement('div');
            body.className = 'activity-card-body';
            const meta = document.createElement('p');
            const tag = document.createElement('span');
            tag.className = 'activity-tag';
            tag.textContent = a.category;
            const date = document.createElement('span');
            date.className = 'activity-date';
            date.textContent = a.dateLabel;
            meta.append(tag, date);
            const h3 = document.createElement('h3');
            h3.textContent = a.title;
            const p = document.createElement('p');
            p.className = 'activity-excerpt';
            p.textContent = a.description;
            body.append(meta, h3, p);
            card.append(img, body);
            card.addEventListener('click', () => openModal(a));
            grid.appendChild(card);
        });
        moreBtn.hidden = list.length <= visibleCount;
    }

    moreBtn.addEventListener('click', () => { visibleCount += PAGE_SIZE; render(); });
    render();

    // ---- Lightbox ----
    const modal = document.getElementById('activityModal');
    const modalPhoto = document.getElementById('modalPhoto');
    const modalCount = document.getElementById('modalCount');
    let modalPhotos = [];
    let photoIndex = 0;

    function showPhoto() {
        modalPhoto.src = modalPhotos[photoIndex] || PLACEHOLDER_IMG;
        modalPhoto.onerror = function () { this.onerror = null; this.src = PLACEHOLDER_IMG; };
        modalCount.textContent = toBnDigits(photoIndex + 1) + ' / ' + toBnDigits(modalPhotos.length);
        modalCount.hidden = modalPhotos.length < 2;
        document.getElementById('modalPrev').hidden = modalPhotos.length < 2;
        document.getElementById('modalNext').hidden = modalPhotos.length < 2;
    }

    function openModal(a) {
        modalPhotos = a.photos.length ? a.photos : [PLACEHOLDER_IMG];
        photoIndex = 0;
        document.getElementById('modalCategory').textContent = a.category;
        document.getElementById('modalDate').textContent = a.dateLabel;
        document.getElementById('modalTitle').textContent = a.title;
        document.getElementById('modalDescription').textContent = a.description;
        const link = document.getElementById('modalLink');
        link.hidden = !a.link;
        if (a.link) link.href = a.link;
        showPhoto();
        modal.hidden = false;
        document.body.style.overflow = 'hidden';
    }

    function closeModal() {
        modal.hidden = true;
        document.body.style.overflow = '';
    }

    document.getElementById('modalClose').addEventListener('click', closeModal);
    document.getElementById('modalPrev').addEventListener('click', () => {
        photoIndex = (photoIndex - 1 + modalPhotos.length) % modalPhotos.length; showPhoto();
    });
    document.getElementById('modalNext').addEventListener('click', () => {
        photoIndex = (photoIndex + 1) % modalPhotos.length; showPhoto();
    });
    modal.addEventListener('click', (e) => { if (e.target === modal) closeModal(); });
    document.addEventListener('keydown', (e) => {
        if (modal.hidden) return;
        if (e.key === 'Escape') closeModal();
        if (e.key === 'ArrowLeft') document.getElementById('modalPrev').click();
        if (e.key === 'ArrowRight') document.getElementById('modalNext').click();
    });
})();
```

Note: the hero's latest-card click already targets `#activities`; no change needed there.

- [ ] **Step 4: Verify happy path in browser**

Reload. Expected: 6 cards newest-first (১০ জুলাই ২০২৬ first); chips সব + 4 categories, **no অন্যান্য chip**; clicking `উন্নয়ন ও ত্রাণ` shows exactly 2 cards; সব restores 6; load-more button hidden (6 ≤ 9); clicking a card opens the lightbox with photo, Bangla date, description, ফেসবুকে দেখুন link; Escape and backdrop close it; single-photo entries show no ‹ › arrows.

- [ ] **Step 5: Verify edge cases with a temporary entry**

Temporarily paste at the top of the `activities` array:

```js
    { date: "12-07-2026", category: "ভুল", title: "টেস্ট", description: "এজ কেস", photos: ["activity-photos/nai.jpg"] },
```

Reload. Expected: entry appears **last** (invalid date), tagged `অন্যান্য`, an `অন্যান্য` chip now exists, card shows the "ছবি নেই" placeholder, raw date string shown as-is. Then break the file (delete a `}`), reload — activities section shows "কার্যক্রম লোড করা যাচ্ছে না…", hero identity still renders, nav/toggle still work. **Restore the file exactly** and re-run the Task 1 parse check (`6 activities, 4 categories`).

- [ ] **Step 6: Add ≥9 pagination check**

Temporarily duplicate the last three seed entries (with dates `2026-05-01`, `2026-05-02`, `2026-05-03`, plus a 10th `2026-04-28`), reload: exactly 9 cards render, আরও দেখুন visible, click → 10th card appears and button hides. Remove the duplicates; parse check again.

- [ ] **Step 7: Commit**

```bash
git add index.html
git commit -m "feat: activities timeline with filters, load-more, and lightbox"
```

---

### Task 6: Reframe kept sections — about, commitments, videos, contact, footer

**Files:**
- Modify: `index.html` — about section (`id="about"`), qualifications section, manifesto + vision sections (merged into `id="commitments"`), videos (`id="videos"`), contact (`id="contact"`), footer, and the opinion section lead.

**Interfaces:**
- Consumes: design tokens from Task 2.
- Produces: sections with ids `about`, `commitments`, `videos`, `contact` matching the Task 2 nav.

- [ ] **Step 1: Update the about section to MP framing**

In `<section id="about">`, replace the Bangla intro `<h3>` + first `<p>` and highlight box with:

```html
<h3>রাজশাহী-১ এর নির্বাচিত সংসদ সদস্য</h3>
<p>অধ্যাপক মুজিবুর রহমান জামায়াতে ইসলামী বাংলাদেশের সিনিয়র নায়েবে আমীর এবং রাজশাহী-১ (গোদাগাড়ী-তানোর) আসনের নির্বাচিত সংসদ সদস্য। শিক্ষা, রাজনীতি ও সমাজসেবায় কয়েক দশকের অভিজ্ঞতা নিয়ে তিনি এলাকার মানুষের সেবায় নিয়োজিত।</p>
<div class="highlight-box">
    <strong>নির্বাচনী অভিজ্ঞতা:</strong> অধ্যাপক মুজিব ১৯৮৬ সালে রাজশাহী-১ আসন থেকে সংসদ সদস্য নির্বাচিত হন এবং ২০২৬ সালের জাতীয় সংসদ নির্বাচনে একই আসন থেকে পুনরায় বিপুল ভোটে নির্বাচিত হয়েছেন।
</div>
```

And the matching English block:

```html
<h3>The Elected Member of Parliament for Rajshahi-1</h3>
<p>Professor Mujibur Rahman is the Senior Nayeb-e-Ameer of Bangladesh Jamaat-e-Islami and the elected Member of Parliament for Rajshahi-1 (Godagari–Tanore), bringing decades of experience in education, politics, and community service.</p>
<div class="highlight-box">
    <strong>Election Experience:</strong> Professor Mujib was elected MP from Rajshahi-1 in 1986, and was re-elected from the same constituency with a large majority in the 2026 national election.
</div>
```

Keep the remaining paragraphs (books, party history) unchanged. In the qualifications card "রাজনৈতিক নেতৃত্ব", change the Bangla `<p>` to `জামায়াতে ইসলামী বাংলাদেশের সিনিয়র নায়েবে আমীর, ১৯৮৬ ও ২০২৬ সালে রাজশাহী-১ থেকে নির্বাচিত এমপি` and English to `Senior Nayeb-e-Ameer of Bangladesh Jamaat-e-Islami, elected MP from Rajshahi-1 in 1986 and 2026`.

- [ ] **Step 2: Merge manifesto + vision into a single commitments section**

Change `<section id="vision" class="vision">` to `<section id="commitments" class="vision">`. Replace its titles/lead:

```html
<h2 class="section-title bn-content">অঙ্গীকার</h2>
<h2 class="section-title en-content">Commitments</h2>
<p class="section-lead bn-content">যে অঙ্গীকার নিয়ে নির্বাচিত হয়েছি — বাস্তবায়নই এখন লক্ষ্য।</p>
<p class="section-lead en-content">The commitments we were elected on — now being delivered.</p>
```

Keep the `vision-grid` items exactly as they are. Then move the two `manifesto-card` divs (PDF দেখুন/ডাউনলোড) from the manifesto section to the end of this section's `.container` (inside their existing `manifesto-grid` wrapper), retitled with a small heading above them:

```html
<h3 class="sub-title bn-content">নির্বাচনী ইশতেহার</h3>
<h3 class="sub-title en-content">Election Manifesto</h3>
```

Below the manifesto grid, add the ishtehar image gallery:

```html
<div class="istehar-gallery">
    <a href="istehar_1.jpg" target="_blank"><img src="istehar_1.jpg" alt="ইশতেহার ১" loading="lazy"></a>
    <a href="istehar_2.jpg" target="_blank"><img src="istehar_2.jpg" alt="ইশতেহার ২" loading="lazy"></a>
    <a href="istehar_3.jpg" target="_blank"><img src="istehar_3.jpg" alt="ইশতেহার ৩" loading="lazy"></a>
    <a href="istehar_4.jpg" target="_blank"><img src="istehar_4.jpg" alt="ইশতেহার ৪" loading="lazy"></a>
    <a href="istehar_5.jpg" target="_blank"><img src="istehar_5.jpg" alt="ইশতেহার ৫" loading="lazy"></a>
    <a href="istehar_6.jpg" target="_blank"><img src="istehar_6.jpg" alt="ইশতেহার ৬" loading="lazy"></a>
    <a href="istehar_7.jpg" target="_blank"><img src="istehar_7.jpg" alt="ইশতেহার ৭" loading="lazy"></a>
    <a href="istehar_8.jpg" target="_blank"><img src="istehar_8.jpg" alt="ইশতেহার ৮" loading="lazy"></a>
    <a href="istehar_9.jpg" target="_blank"><img src="istehar_9.jpg" alt="ইশতেহার ৯" loading="lazy"></a>
</div>
```

With CSS:

```css
.sub-title { color: var(--ink); text-align: center; margin: 2.5rem 0 1rem; font-size: 1.3rem; }
.istehar-gallery { display: grid; grid-template-columns: repeat(auto-fill, minmax(140px, 1fr)); gap: 0.6rem; margin-top: 1.25rem; }
.istehar-gallery img { width: 100%; border-radius: var(--radius); box-shadow: var(--shadow); display: block; }
```

Delete the now-empty `<section class="manifesto-section">` wrapper (title, lead, and container) once its cards have moved.

- [ ] **Step 3: Restyle remaining sections to the token palette**

Sweep the CSS for leftover campaign styling in the kept sections (grep for `#006633`, `#004422`, `#DC143C`, `linear-gradient` outside the header/hero rules you already replaced). Point section backgrounds at `var(--bg)`/`var(--surface)` alternately (about: surface; commitments: bg; videos: surface; opinion: bg; contact: surface), headings at `var(--ink)`, accents at `var(--green)`. The `.section-title` underline/accent and the card classes in these sections (`.qual-card`, `.vision-item`, `.manifesto-card`, `.contact-item`, and the video wrappers — check the actual class names in the file) get `border-radius: var(--radius); box-shadow: var(--shadow); background: var(--surface);`. Do not touch form logic.

- [ ] **Step 4: Update contact + footer content**

In the contact section's দল card, change the Bangla `<p>` to `জামায়াতে ইসলামী বাংলাদেশ<br>সিনিয়র নায়েবে আমীর` (EN: `Bangladesh Jamaat-e-Islami<br>Senior Nayeb-e-Ameer`). In the সোশ্যাল মিডিয়া card, replace the single link `<p>` with:

```html
<p><a href="https://www.facebook.com/ProfMujib.Official" target="_blank">@ProfMujib.Official</a></p>
<p><a href="https://www.facebook.com/mrmedia11/" target="_blank">মিডিয়া সেল</a></p>
```

In the footer: `© 2025` → `© ২০২৬` (bn) / `© 2026` (en); add a fourth social link:

```html
<a href="https://www.facebook.com/mrmedia11/" target="_blank">
    <span class="bn-content">মিডিয়া সেল</span>
    <span class="en-content">Media Cell</span>
</a>
```

- [ ] **Step 5: Verify in browser**

Reload and walk the page top-to-bottom in both languages. Expected: no crimson/old-green remnants anywhere; about reads as sitting MP with 1986 + 2026 elections; অঙ্গীকার contains vision grid + 2 manifesto PDF cards + 9 ishtehar thumbnails that open full-size in a new tab; videos and opinion form render in the new palette; form still submits (fill it with test data once — expect the success message); nav links land on every section; mobile layout single-columns cleanly.

- [ ] **Step 6: Commit**

```bash
git add index.html
git commit -m "feat: reframe about/commitments/videos/contact for MP era"
```

---

### Task 7: UPDATE_GUIDE.md + final QA + handoff

**Files:**
- Create: `UPDATE_GUIDE.md`

**Interfaces:**
- Consumes: the `activities.js` format from Task 1.
- Produces: the maintainer-facing documentation; nothing downstream.

- [ ] **Step 1: Write `UPDATE_GUIDE.md`**

```markdown
# ওয়েবসাইটে নতুন কার্যক্রম যোগ করার নিয়ম

এই গাইডটি তাঁদের জন্য যাঁরা কোডিং জানেন না। সবকিছু GitHub ওয়েবসাইট
থেকেই করা যাবে — কোনো সফটওয়্যার ইনস্টল করতে হবে না।

## ধাপ ১: ছবি আপলোড করুন

১. GitHub-এ এই প্রজেক্টটি খুলুন এবং **activity-photos** ফোল্ডারে ঢুকুন।
২. উপরে ডান দিকে **Add file → Upload files** চাপুন।
৩. ছবিগুলো টেনে এনে ছাড়ুন (ফাইলের নাম ইংরেজিতে রাখুন, যেমন
   `2026-07-15-sova-1.jpg`)।
৪. নিচে সবুজ **Commit changes** বোতাম চাপুন।

## ধাপ ২: activities.js ফাইলে তথ্য যোগ করুন

১. প্রজেক্টের মূল পাতায় ফিরে **activities.js** ফাইলটিতে ক্লিক করুন।
২. ডান দিকে পেন্সিল (✏️ Edit) আইকনে ক্লিক করুন।
৩. ফাইলের উপরের দিকে `const activities = [` লেখা লাইনটি খুঁজুন।
৪. তার ঠিক নিচে এই নমুনাটি পেস্ট করে নিজের তথ্য বসান:

    {
        date: "2026-07-15",
        category: "জনসংযোগ",
        title: "এখানে শিরোনাম লিখুন",
        description: "এখানে বিস্তারিত বিবরণ লিখুন।",
        photos: ["activity-photos/2026-07-15-sova-1.jpg"],
        link: "https://www.facebook.com/mrmedia11/posts/..."
    },

৫. **Commit changes** চাপুন। কয়েক মিনিটের মধ্যে ওয়েবসাইটে দেখা যাবে।

## মনে রাখবেন

- **date**: বছর-মাস-দিন, ইংরেজি সংখ্যায় (যেমন `2026-07-15`)। ওয়েবসাইটে
  নিজে থেকেই বাংলায় দেখাবে।
- **category**: শুধু এই চারটির একটি, হুবহু:
  `জনসংযোগ` · `বক্তব্য ও সমাবেশ` · `উন্নয়ন ও ত্রাণ` · `সংসদ`
- **photos**: একাধিক ছবি দিতে চাইলে কমা দিয়ে লিখুন:
  `photos: ["activity-photos/chobi-1.jpg", "activity-photos/chobi-2.jpg"],`
- **link**: ফেসবুক পোস্টের লিংক। না দিতে চাইলে `link: ""` লিখুন।
- প্রতিটি তথ্যের শেষে কমা (,) এবং দুই পাশের উদ্ধৃতি চিহ্ন ("...") ঠিক আছে
  কি না দেখে নিন — এটিই সবচেয়ে সাধারণ ভুল।
- নতুন কার্যক্রম কোথায় বসালেন তা নিয়ে ভাবতে হবে না — ওয়েবসাইট নিজেই
  তারিখ অনুযায়ী নতুন থেকে পুরাতন সাজিয়ে নেয়।

## কিছু ভুল হয়ে গেলে

ভয়ের কিছু নেই — ভুল হলে ওয়েবসাইটের কার্যক্রম অংশে একটি বার্তা দেখাবে,
বাকি সাইট ঠিক থাকবে। GitHub-এ activities.js ফাইলের **History** বোতাম
থেকে আগের সংস্করণ দেখে ঠিক করে নেওয়া যায়, অথবা যিনি সাইটটি
দেখাশোনা করেন তাঁকে জানান।
```

- [ ] **Step 2: Run the full QA checklist (spec §Verification)**

With `python3 -m http.server 8000` running, confirm each:

1. Filters: each chip shows only its category; সব shows all 6
2. Load-more: re-run the Task 5 Step 6 temporary ≥9 check if not already convincing, then restore
3. Lightbox: multi-photo carousel (temporarily give one seed entry a second photo, verify ‹ › and counter, restore), description, FB link, Escape/backdrop close
4. Live Hero shows the 2026-07-10 entry; add a temporary `2026-07-14` entry → hero swaps to it → remove
5. Edge cases: bad category → অন্যান্য; missing photo → placeholder; broken `activities.js` → friendly message, rest of site intact (repeat quickly from Task 5 Step 5)
6. BN/EN toggle correct in header, hero identity, stats labels, about, commitments, videos, opinion, contact, footer
7. Mobile ≤ 480px and desktop: no horizontal scroll, grids collapse, lightbox usable
8. Console: zero errors on load and through all interactions
9. `node -e "$(cat activities.js); console.log(activities.length)"` → `6`

- [ ] **Step 3: Commit**

```bash
git add UPDATE_GUIDE.md
git commit -m "docs: Bangla update guide for the media cell"
```

- [ ] **Step 4: Handoff notes for the site owner (include in final report)**

- Seed activities are drafted placeholders — replace with real posts from facebook.com/mrmedia11 (edit `activities.js`, upload real photos)
- Confirm the ইউনিয়ন count (১৬) and voter total (৪,৬৪,৬৩১) on the stats band
- Confirm the hero quote wording
- `git push` publishes the redesign; the `campaign-site` git tag preserves the old site
