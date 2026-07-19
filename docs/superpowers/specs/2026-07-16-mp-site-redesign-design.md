# MP Site Redesign — Design Spec

**Date:** 2026-07-16
**Project:** vote4mujibur — transform the election campaign site into the official site of Prof. Mujibur Rahman, MP for Rajshahi-1 (Godagari–Tanore)

## Goal

The election is over and won. The site's job changes from "vote for me" to "here is what your MP is doing" — a continuously updated showcase of his activities, mirroring the content of the অধ্যাপক মুজিবুর রহমান মিডিয়া সেল Facebook page (facebook.com/mrmedia11), but owned, designed, and hosted independently.

## Decisions made during brainstorming

| Question | Decision |
|---|---|
| Scope | Full transformation into a sitting-MP site |
| Content flow | Manually curated (no Facebook embed/scraping) |
| Activity types | All: community visits, speeches/rallies, development & relief, parliament |
| Kept sections | About & qualifications, manifesto & vision, contact & opinion form, video gallery, constituency stats |
| Removed | Vote-day updates section, election results summary/table ("এক নজরে রাজশাহী-১"), campaign framing |
| Stats band content | Constituency facts (upazilas, unions, voters) — not election results |
| Maintainer | Non-technical media cell person, via GitHub web UI only |
| Activity language | Bangla only (static sections keep the BN/EN toggle) |
| Layout | News Timeline — single chronological feed with category filters |
| Visual style | Modern Official — white surfaces, single deep green `#00703c`, rounded cards, pill tags |
| Hero | Live Hero — identity left, newest activity auto-featured right |

## Architecture

Three artifacts, no build step, GitHub Pages compatible:

- **`index.html`** — the entire site (markup, styles, logic), redesigned in the Modern Official style. Developers own this file; the media cell never touches it.
- **`activities.js`** — a plain `<script>`-loaded file defining a single global `activities` array. The only file the media cell edits.
- **`activity-photos/`** — folder where activity images are uploaded via GitHub web UI.

Plus **`UPDATE_GUIDE.md`** — step-by-step Bangla instructions for adding an activity through the GitHub web interface.

Existing assets kept: `SolaimanLipi.ttf`, profile/manifesto/ishtehar images, manifesto PDFs, Supabase opinion form (URL + anon key unchanged).

## Page structure (top to bottom)

1. **Header** — white sticky nav, green bottom border: কার্যক্রম · পরিচিতি · অঙ্গীকার · ভিডিও · যোগাযোগ + BN/EN toggle.
2. **Live Hero** — left: portrait, name, "সংসদ সদস্য, রাজশাহী-১ (গোদাগাড়ী-তানোর)", a one-line quote, button to activities. Right: the newest activity (by date) rendered as a card with a "সর্বশেষ কার্যক্রম" badge — photo, title, category, date. Pulled automatically from `activities.js`; no separate maintenance.
3. **Constituency stats band** — three static tiles of constituency facts: উপজেলা (২টি — গোদাগাড়ী ও তানোর), ইউনিয়ন, মোট ভোটার. Values are hardcoded; the voter total is extracted from the existing results data during implementation and confirmed with the site owner. No election-result content here — the old results summary and table are removed entirely.
4. **কার্যক্রম (Activities)** — the centerpiece:
   - Filter chips: সব · জনসংযোগ · বক্তব্য ও সমাবেশ · উন্নয়ন ও ত্রাণ · সংসদ (+ অন্যান্য appears only if needed)
   - Cards newest-first: first photo, Bangla-formatted date, category pill, title, 2-line excerpt
   - 9 cards initially, "আরও দেখুন" load-more button reveals 9 more at a time
   - Clicking a card opens a lightbox/modal: photo carousel (all photos of that activity), full description, optional "ফেসবুকে দেখুন" link to the original post
5. **পরিচিতি (About)** — bio and qualifications merged, rewritten to sitting-MP framing.
6. **অঙ্গীকার (Commitments)** — manifesto/vision reframed ("যে অঙ্গীকার নিয়ে নির্বাচিত হয়েছি"), keeping ishtehar images and PDF downloads.
7. **ভিডিও গ্যালারি** — existing YouTube embeds, restyled to match.
8. **যোগাযোগ (Contact)** — Supabase opinion form as-is, contact info, links to ProfMujib.Official and mrmedia11 Facebook pages.
9. **Footer** — dark green, minimal.

Bilingual behavior: static sections (about, commitments, contact, labels) keep BN/EN content pairs and the toggle. Activity entries are Bangla-only and display identically in both language modes.

## Activity data format

```js
// activities.js — নতুন কার্যক্রম যোগ করার নিয়ম উপরে কমেন্টে
const activities = [
    {
        date: "2026-07-12",                       // বছর-মাস-দিন (ইংরেজি সংখ্যায়)
        category: "উন্নয়ন ও ত্রাণ",                 // নিচের তালিকা থেকে হুবহু কপি করুন
        title: "তানোরে নতুন রাস্তার উদ্বোধন",
        description: "তানোর উপজেলায় ৩ কিলোমিটার পাকা রাস্তার উদ্বোধন করা হয়...",
        photos: ["activity-photos/2026-07-12-rasta-1.jpg"],
        link: "https://www.facebook.com/mrmedia11/posts/..."   // ঐচ্ছিক
    },
];
```

- Valid categories (exact strings): `জনসংযোগ`, `বক্তব্য ও সমাবেশ`, `উন্নয়ন ও ত্রাণ`, `সংসদ`
- `date` is ISO (`YYYY-MM-DD`) in ASCII digits for sorting; the site renders it in Bangla ("১২ জুলাই ২০২৬") — digit conversion and Bangla month names implemented in `index.html`
- `photos` is an array of repo-relative paths; first photo is the card thumbnail
- `link` is optional
- File header contains Bangla instructions and a ready-to-copy blank template

## Mistake-proofing (media cell edits must not break the site)

- **Order-independent:** site sorts by `date` descending; new entries can be pasted anywhere in the array
- **Unknown/misspelled category:** card still renders, tagged `অন্যান্য`, visible under সব
- **Missing/broken photo:** neutral placeholder shown (`onerror` fallback); card never breaks layout
- **Empty `photos` array:** card renders text-only with placeholder thumbnail
- **Invalid date:** entry sorts last and shows the raw string rather than crashing
- **Syntax error in `activities.js`:** the file loads in its own `<script>` tag, so a parse failure cannot break `index.html`'s script; the main script checks `typeof activities === "undefined"` and renders "কার্যক্রম লোড করা যাচ্ছে না — activities.js ফাইলটি পরীক্ষা করুন" in the activities section while the rest of the site works normally (the hero falls back to identity-only)

## Visual design tokens (Modern Official)

- Background `#f7f9f8`, surfaces white, text `#0b1f16`, secondary `#5a6e64`
- Single accent green `#00703c`; light green tint `#e6f2ec` for category pills
- Rounded cards (10px), soft shadows, pill-shaped tags and buttons
- Typography unchanged: SolaimanLipi / Noto Sans Bengali / Hind Siliguri stack
- Mobile-first responsive; cards go single-column on small screens

## Out of scope

- No backend, CMS, or admin UI — GitHub web UI is the CMS
- No automated Facebook sync or scraping
- No English translations of activities
- No changes to the Supabase form backend

## Verification plan

Seed with ~6 real activities (drafted from the Facebook page, corrected by the site owner). Manual checklist:

1. Filters show the right cards; সব shows everything
2. Load-more paging works past 9 items
3. Lightbox: multi-photo carousel, description, FB link, close/escape
4. Live Hero shows the newest activity; updates when a newer entry is added
5. Edge cases: bad category → অন্যান্য; missing photo file → placeholder; broken `activities.js` → friendly message, rest of site intact
6. BN/EN toggle still works on all static sections
7. Mobile layout (≤480px) and desktop both render correctly
