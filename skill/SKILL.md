---
name: snap-scout
description: >
  Turn a single reference photo into a one-page scouting report — location,
  current state, shooting conditions, and visually-matched alternatives.
  Use when the user shares a photo and asks where it was taken, wants a
  location report, says "스냅스카웃 돌려줘" / "이 사진 정찰해줘" / "snap-scout this",
  or pastes a Pinterest pin URL asking for analysis. Do NOT use for general
  reverse image search without a photographer's intent.
---

# SNAP-SCOUT Skill

A reference photo → field-report generator for snap photographers.
Built for photographer 채빈. Output: a magazine-style HTML report that
answers "where is this, can I shoot there now, what are the alternatives."

Live collection: https://tryoutmarket.github.io/tm-lab-snap-scout/

## 1. When to invoke
1. User pastes a reference photo and asks "여기 어디?" / "이 공간 어디야?"
2. User says "snap-scout / 스냅스카웃 / 이걸로 리포트 만들어줘"
3. User shares a Pinterest pin URL and asks to identify / analyze it
4. User wants a scouting report for their next snap shoot location

Do NOT invoke for general "what is this object" questions or photos without
a clear photography-scouting intent.

## 2. Inputs

| Input | Form | Required |
|-------|------|----------|
| Reference photo | local path or URL or Pinterest pin URL | yes |
| User notes | optional context (city hint, planned date, mood) | no |
| Report slug | optional kebab-case override | no |

If only a Pinterest URL is given, fetch the pin image first (`i.pinimg.com/...`).

## 3. Workflow

Run the steps in order. Skip a step only when explicitly impossible
(e.g., no text in photo → skip OCR; no EXIF → skip EXIF).

### 3.1 Allocate report folder
1. Read `/Users/koo/Documents/GitHub/tryoutmarket-workspace/tm-lab/snap-scout/`
2. Find highest existing `NN_sample-report.html` or `reports/NNN_*` number
3. Allocate next number `NNN` (zero-padded)
4. Create `tm-lab/snap-scout/reports/NNN_{slug}/` with subfolder `gallery/`
5. Copy the input photo to `reports/NNN_{slug}/reference.jpg`

### 3.2 Identify the location (max 3 parallel signals)
1. **Visual OCR** — read any foreign-language text (Chinese/Japanese/etc.)
   visible in the photo. These are the strongest signals.
2. **Visual signatures** — architecture style, terrain (steep stairs, narrow
   alley, neon density), light direction, season cues, vehicle plates.
3. **EXIF** — if local file, check GPS coords via `exiftool` or equivalent.
4. **WebSearch** with the OCR text + visual cues to converge on a candidate.
5. If the photo is a Pinterest pin, also try to find the same pin via
   reverse search; the pin description often reveals location.

Output of this step: candidate location (place name + city + country),
confidence level (high/medium/low), and the 3–5 specific clues that led there.

### 3.3 Verify current state
1. WebSearch for the candidate location with recent date filters
2. Look for: closure / renovation / reopening / access change
3. Check Google Maps reviews dated within last 6 months
4. Note any active festivals/traditions at the location

### 3.4 Shooting conditions
Compile a quick-reference block:
1. Entry fee / permits required
2. Best light direction & golden hour timing (cite local sunrise/sunset)
3. Crowd patterns (weekday vs weekend, time of day)
4. Wedding shoot specific notes (tripod allowed? size restrictions?)
5. Nearby parking / transit

### 3.5 Visual match — alternatives
Goal: 8–10 reference images of the same place + similar-mood places.

1. **If Pinterest pin URL provided** — open the pin's related-pins feed
   via Playwright (`browser_navigate` to the pin page → scroll → extract
   `i.pinimg.com/736x/...` URLs). No login needed for related pins.
2. **If only photo** — Pinterest search with OCR text + place name (e.g.
   `觀音堂 香港 婚紗`) on `pinterest.com/search/pins/?q=...`. Login-free works.
3. **Lightweight curl download** — once you have pin IDs, batch download
   with `/usr/bin/curl -o gallery/rNN.jpg <url>` (use full path; PATH may
   not include curl).
4. Visually filter to top 8–10 best matches. Prefer matches by:
   - Same location (highest weight)
   - Same mood/tone
   - Similar geometry (stairs, alley, signage density)
5. Save URLs of the source pins alongside images for `[N]` citations.

### 3.6 Render the report
1. Read reference example: `references/example-001-source.html` (in this
   skill folder) — clone its head/style/structure verbatim
2. Replace all content for the new place — keep CSS and layout identical
3. Sections in order:
   - Cover (reference.jpg + meta table)
   - Verdict bar (one-line: shootable? difficulty? best month?)
   - 01 Identification (with numbered visual clues)
   - 02 Shooting Conditions
   - 03 Current State
   - 04 Alternatives (text list of similar places)
   - 05 Overseas Mode (visa, season, coordinator notes — only if abroad)
   - 06 Caution (anything that could ruin the shoot)
   - 07 Gallery (10 Pinterest visual matches with pin-page links)
   - Sources (numbered `[N]` footnotes — never skip; this matters to user)
4. Write to `reports/NNN_{slug}/report.html`
5. `open reports/NNN_{slug}/report.html`

### 3.7 Update the index
1. Add a new `<a class="report-card">` to `tm-lab/snap-scout/index.html`
   in the `.reports` grid (insert before the first `soon` card)
2. Card fields: №NNN, country tag, date, title (place name + locale),
   sub (one-line description), foot ("WHY-IT-MATCHED note")
3. Cover image: use `reports/NNN_{slug}/reference.jpg`
4. Update the `.coll-count strong` published number

### 3.8 Offer to publish
Ask the user: "리포트 됐어. GitHub Pages에 푸시할까?"
If yes:
```
cd /Users/koo/Documents/GitHub/tryoutmarket-workspace/tm-lab/snap-scout
git add . && git commit -m "Report №NNN — {place}" && git push
```
Build takes ~30s. Live URL pattern:
`https://tryoutmarket.github.io/tm-lab-snap-scout/reports/NNN_{slug}/report.html`

## 4. Design tone (NON-NEGOTIABLE)
1. **Magazine B black & white gothic** — Pretendard + Inter + JetBrains Mono
2. Numbered sections (`01`, `02`, ...) — user requires numbering on everything
3. Sources box at bottom with `[N]` footnotes — citations are critical to user
4. No emojis, no decorative icons, no rounded corners on cards
5. Bilingual headers: Korean primary + English smaller line beneath
6. Mono font for meta/labels, sans for body

## 5. Hard rules
1. NEVER overwrite existing files. New reports go to a new numbered folder.
2. NEVER commit `docs/` or `_drafts/` — they are gitignored for a reason.
3. Sources block is mandatory. If you can't cite something, mark it
   "추정" / "estimated" rather than fabricating a source.
4. The reference photo MUST be embedded, not a placeholder.
5. Pinterest gallery images MUST link to their pin pages (clickable cards).
6. Confidence levels are honest — if location is uncertain say so in the
   verdict bar and Identification section.

## 6. v2+ roadmap (not yet implemented — do not attempt)
- v2: Pinterest board URL → batch report for all pins
- v3: Photographer's own work → signature DB extraction
- v4: Combine v2 + v3 for personalized location curation

If user asks for these, say they're roadmap items and ask if they want to
prototype them as a follow-up — don't fake them.

## 7. References
- `references/example-001-source.html` — finished example (觀音堂).
  Always clone its head/CSS/structure for new reports; only the content
  inside each section changes.
