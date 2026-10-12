# Product Spec — The Corporate Double Materiality Matrix 2026

**Version:** 1.1
**Date:** 12 October 2026
**Author:** Ajay
**Status:** Confirmed
> The Tool Architect sets this to **Confirmed** when the builder confirms the Phase 10 summary. The Project Governor will not accept a spec in any other status.

---

## Section 1 — Tool Summary

**Tool name:** The Corporate Double Materiality Matrix 2026

**What it does:** A single-page, interactive double materiality matrix that plots The Corporate's 16 material ESG topics as numbered, colour-coded dots on a 1–5 Financial Materiality × 1–5 Impact Materiality grid, over four background zones (Highly Material, Material, Watch List, Not Material) derived from the approved materiality thresholds. The user can toggle the Environmental, Social and Governance categories on and off and download the current view as a PNG.

**Who uses it:** Ajay, the sustainability professional responsible for The Corporate's 2026 Double Materiality Assessment. He opens the file locally on his own computer and presents it live in meetings to the Double Materiality Working Group and the Executive Leadership Team. Nobody else opens the tool directly; others see it on screen or as exported PNG images in slides and the report.

**Why it exists:** The Corporate has completed every step of its 2026 Double Materiality Assessment — topic identification (16 topics), stakeholder engagement (1,000 employees, 50 Tier-1 suppliers, 5 investors, 15 OEM customers, each group weighted 25%), aggregation of blended scores, and approval of materiality thresholds (12 October 2026). The one missing deliverable is the matrix itself: a clear, on-brand visual that shows where each topic falls against the approved thresholds and lets the presenter isolate E, S or G topics during discussion.

**Build status:** First build — no prior version.

---

## Section 2 — Classification

This section defines the architecture of the tool. Every downstream decision follows from this.

### Data Model

**Decision:** D1

| Label | What it means | This tool? |
|-------|--------------|-----------|
| D1 — Hardcoded | All data is written into the code by the developer. Users cannot input anything that persists. The tool displays what the developer put in. | Yes |
| D2 — Session | Data enters the tool during use and disappears when the tab closes. No database. Covers both uploaded files and form inputs. | No |
| D3 — Persisted | Data is written to a database and survives after the session ends. Supabase is required. | No |

**Reason:** The 16 topic scores are final, signed-off figures from the aggregated scores workbook; they are written into the file by the builder and the user never enters, uploads or saves data — filter state is visual only and resets on reload.

**D3 is triggered if any of the following are true — check all that apply:**
- [ ] Data must be retrievable after the session ends
- [ ] Multiple sessions contribute to the same dataset
- [ ] An audit trail or history is needed
- [ ] Data submitted by one person must be visible to another
- [ ] Results must be accessible via a URL after the session ends
- [ ] Files uploaded by users must be stored and retrievable later

None apply.

---

### Access Model

**Decision:** A1

| Label | What it means | This tool? |
|-------|--------------|-----------|
| A1 — Public | Anyone with the URL can use it. No login, no account required. | Yes |
| A2 — Authentication | Users must log in. All logged-in users see the same thing and have the same permissions. | No |
| A3 — Authorization | Users must log in and have different roles. Different roles see different data or have different permissions. | No |

**Reason:** No login is required — the tool is a local file opened by double-click on the builder's own computer. It is not published to any URL, so "anyone with the URL" in practice means only whoever has the file.

> **Promotion rule:** Auth requires a database. If the access model is A2 or A3, the data model is D3 — even when all displayed content is fixed. D1/D2 combined with A2/A3 are not valid classifications; they resolve to D3.

---

### If Access Model is A2 — complete both questions

N/A — Access Model is A1.

---

### If Access Model is A3 — define all roles

N/A — Access Model is A1.

---

### Tier

**Tier:** 1

| Tier | D+A combination | Stack | Deployment |
|------|----------------|-------|------------|
| 1 | D1+A1 or D2+A1 | Netlify only | Netlify |
| 2 | D3+A1 | Netlify + Supabase (no auth) | Netlify |
| 3 | D3+A2 or D3+A3 | Netlify + Supabase (auth + RLS) | Netlify |

> **Deployment deviation confirmed by the builder:** this Tier 1 tool is **not deployed to Netlify**. It is delivered as a single self-contained `index.html` file that the builder opens locally by double-clicking. See Section 4.

---

### Standalone or Stack

**This tool is:** Standalone — it does not share a database with any other tool (it has no database).

---

## Section 3 — Arms

Arms are capabilities added to the tool. They do not change the tier. Mark each arm active or not, and complete the detail only for active arms.

> **Document search and AI knowledge bases are outside this framework version.** Not requested for this tool.

---

### AI API Arm

**Active:** No

---

### Export Arm

**Active:** Yes

| Detail | Answer |
|--------|--------|
| Format | PNG image (neither CSV nor PDF). Generated entirely in the browser — no server, no library, works offline. |
| What is exported | A "Download as PNG" button exports the matrix **exactly as currently filtered**: the header (wordmark "[ C ] THE CORPORATE" and title "Double Materiality 2026" with "Author: Ajay"), the chart with its four background zones, axes and axis titles, all currently visible dots with their numbers, the numbered topic list (hidden categories shown greyed out exactly as on screen), the zone legend with threshold rules, the live zone counts, and the source footnote. The E/S/G toggle buttons and the Download button themselves are **not** included in the image. Export at 2× pixel density so it stays sharp in slides and print. Filename: `The-Corporate-Double-Materiality-2026.png`; when one or more categories are switched off, append the visible categories, e.g. `The-Corporate-Double-Materiality-2026_E-S.png`. |
| PDF design intent | N/A — no PDF export. |

---

### Email Arm

**Active:** No

---

### Scheduled Automation Arm

**Active:** No

---

## Section 4 — Stack and Deployment

### All Tiers

| Detail | Answer |
|--------|--------|
| Frontend framework | HTML/CSS/JS — one single-view page with modest interaction (three toggles, hover tooltips, one export button). Everything — markup, styles, script and data — lives in **one self-contained `index.html` file**. The chart is drawn as inline SVG with plain JavaScript. **No external libraries, no CDN links, no web fonts, no network requests of any kind** — the file must work fully offline in meeting rooms without Wi-Fi. |
| Deployment target | **None — local file.** Builder decision during the interview: no Netlify site, no shared link. The deliverable is `index.html`, opened by double-clicking it in a desktop browser (Chrome, Edge, Safari, Firefox current versions) via the `file://` protocol. |
| Netlify MCP | Not applicable — the tool is not deployed. |

**GitHub — pre-build requirement for all Tier 1, 2, and 3 tools:**
The user creates the GitHub repo before the first Claude Code session. The product-spec.md, CLAUDE.md, and PROGRESS.md must be uploaded to the repo root before Claude Code opens. Claude Code assumes the repo exists, commits changes regularly, and pushes to main. It does not create or configure the repo. GitHub is used for version safety only; it is **not** connected to Netlify.

**Offline/`file://` constraints Claude Code must respect:**
- No `fetch()` of local or remote files — the topic data is embedded in the script as a constant.
- No ES module imports from other files — a single inline `<script>`.
- PNG export must not taint the canvas: draw the export on a `<canvas>` from data (or serialise the inline SVG to a data URL) using only inline resources, so `canvas.toBlob()` succeeds when opened from `file://`.
- Fonts: Georgia (headings) and Arial (body), both system fonts, with fallbacks `Georgia, "Times New Roman", serif` and `Arial, Helvetica, sans-serif`.

---

### CONDITIONAL: Supabase project — only complete if Tier 2 or Tier 3

N/A — Tier 1, no database.

---

### CONDITIONAL: Only complete if this tool is part of a stack

N/A — standalone tool.

---

## Section 5 — Data Architecture

### CONDITIONAL: Only complete if Data Model is D3

N/A — Data Model is D1. For build reference, the hardcoded dataset is specified below.

**Embedded dataset (source: `Materiality_Aggregated_Scores.xlsx`, sheet `Aggregated`; topic numbers from "ESG Material Topics of Focus", The Corporate, 2026).** Embed exactly these values — do not round, re-order or alter names.

| # | Topic name | Category | Financial score (x) | Impact score (y) |
|---|-----------|----------|------------------:|----------------:|
| 01 | GHG Emissions (Scopes 1, 2, 3) | E | 3.73 | 4.30 |
| 02 | Energy Management & Renewable Transition | E | 3.58 | 3.80 |
| 03 | Water Stewardship | E | 3.28 | 3.78 |
| 04 | Hazardous Materials & Chemical Management | E | 4.03 | 4.33 |
| 05 | Waste Generation & Circular Economy | E | 3.43 | 3.88 |
| 06 | Upstream Supply Chain Emissions (Scope 3 Cat. 1) | E | 3.70 | 4.28 |
| 07 | Air Quality & Process Emissions | E | 3.18 | 3.55 |
| 08 | Product End-of-Life & E-Waste | E | 3.48 | 4.13 |
| 09 | Occupational Health & Safety | S | 3.95 | 4.20 |
| 10 | Labor Rights & Working Conditions | S | 4.43 | 4.58 |
| 11 | Responsible Minerals Sourcing | S | 4.13 | 4.55 |
| 12 | Community Impact & Local Employment | S | 3.13 | 3.70 |
| 13 | Workforce Development & Skills | S | 3.63 | 3.35 |
| 14 | Diversity, Equity & Inclusion | S | 3.20 | 3.43 |
| 15 | ESG Governance & Board Accountability | G | 3.70 | 3.48 |
| 16 | Multi-Jurisdictional Regulatory Compliance & Ethics | G | 4.33 | 4.05 |

Category totals: E = 8, S = 6, G = 2, total = 16.

**File storage:** No

**Derived or calculated data:** Yes — each topic's zone classification is calculated in the browser from its two scores (rules in Section 9). Zone counts are recalculated from the currently visible topics.

---

## Section 6 — Access and Permissions

### CONDITIONAL: Only complete if Access Model is A2 or A3

N/A — Access Model is A1, no user accounts.

---

## Section 7 — GDPR

### MANDATORY DECISION: Complete this section for every D3 tool.

N/A — this is a D1 tool. No data is stored and no personal data is collected.

---

## Section 8 — Screen and UI Structure

One view only. Layout targets a laptop or meeting-room screen (landscape, minimum 1280 × 720) and must fit without scrolling at 1366 × 768. Below ~900 px width, the side panel stacks beneath the chart.

### Matrix View (the only view)

- **Purpose:** Present where each of the 16 material topics falls on the double materiality matrix and let the presenter isolate E, S or G topics during discussion.

- **What is visible (top to bottom, left to right):**

  1. **Header bar** (black `#000000` background, off-white `#F2F2F2` text):
     - Left: the wordmark **"[ C ] THE CORPORATE"** rendered as text (no image logo exists) — Arial, bold, letter-spaced, uppercase, exactly as it appears on the source documents.
     - Right or beneath: the title **"Double Materiality 2026"** in Georgia, with a small "Author: Ajay" line beneath it (Arial, taupe).
     - Thin taupe `#B6B09F` rule under the header.

  2. **Control bar** (sand `#EAE4D5` background):
     - Three toggle buttons: **"E — Environmental (8)"**, **"S — Social (6)"**, **"G — Governance (2)"**. Each carries a small dot swatch in its category colour. All three are **ON** at page load.
     - ON state: filled/solid with the category swatch and dark text; OFF state: outlined, muted, swatch hollow. State must be obvious from across a meeting room, and not rely on colour alone (use fill vs outline plus `aria-pressed`).
     - A **"Download as PNG"** button, right-aligned, visually distinct from the toggles (black button, off-white text).

  3. **The chart** (left, roughly two-thirds of the width, square plotting area):
     - **X axis:** "Financial Materiality" — scale 1.0 to 5.0, ticks and gridlines at every 0.5, labels at whole numbers (1, 2, 3, 4, 5). Threshold values 2.0, 3.0 and 3.5 must be readable.
     - **Y axis:** "Impact Materiality" — scale 1.0 to 5.0, same ticks.
     - **Background zones** (soft tints so dots stay readable; drawn exactly per the Section 9 geometry):
       - Highly Material — pale red `#F4CFCB`
       - Material — pale orange `#FAD9B5`
       - Watch List — light grey `#ECECEC`
       - Not Material — mid grey `#C9C9C9`
       Zones are separated by thin white lines. Each zone carries a small, unobtrusive name label in its corner (e.g. "HIGHLY MATERIAL" in the top-right corner, "NOT MATERIAL" in the bottom-left), dark text, small caps, that never sits under a dot.
     - **16 dots**, one per topic, positioned at (financial, impact) with true coordinates:
       - Environmental (E) — green `#2E8B3E`, white number
       - Social (S) — yellow `#F2C200`, black number
       - Governance (G) — red `#C62828`, white number
       - Every dot: 1.5–2 px dark outline `#1A1A1A`, radius large enough to hold a two-digit number legibly (~13–15 px at 1366 width), number = topic number (01–16, or 1–16 — consistent with the list).
     - **Overlap handling:** several dots are very close (01 at 3.73/4.30 and 06 at 3.70/4.28 nearly coincide; 02/15, 08/05, 09/01 are also near). Dots must not hide one another's numbers. Claude Code applies a small label-offset (collision) pass that nudges overlapping dots apart and draws a thin leader line from the nudged dot back to its true coordinate (a small dark point). The tooltip always reports the true scores.
     - **Hover tooltip** on each dot (and on keyboard focus): topic number + full topic name, category (Environmental / Social / Governance), "Financial: 3.73", "Impact: 4.30" (two decimals), and zone ("Highly Material"). Off-white card, dark text, taupe border. On hover the corresponding row in the topic list highlights, and hovering a list row highlights its dot.

  4. **Side panel** (right, roughly one-third of the width):
     - **Numbered topic list** grouped under three sub-headings — Environmental (01–08), Social (09–14), Governance (15–16) — each row: coloured number badge + topic name. When a category is toggled OFF, its sub-heading and rows turn grey (reduced opacity, grey badge), stay in place (no reflow), and stay readable.
     - **Zone counts** — four rows, each with its zone swatch: "Highly Material: 8", "Material: 8", "Watch List: 0", "Not Material: 0". Counts reflect **only currently visible topics** and update instantly when toggles change (e.g. with only G on: Highly Material 1, Material 1, Watch List 0, Not Material 0). A small line "Showing 16 of 16 topics" also updates.
     - **Zone legend** — the four swatches with their threshold rules, worded exactly:
       - **Highly Material** — 3.5 or above on BOTH Financial and Impact axes.
       - **Material** — 3.0 or above on EITHER axis, and not Highly Material.
       - **Watch List** — at least one axis 2.0–2.9, and neither axis reaches 3.0.
       - **Not Material** — both axes below 2.0.
       - Plus one line: "Highest matching classification applies."
     - **Dot legend** — three swatches: E = Environmental, S = Social, G = Governance.

  5. **Footnote** (beneath chart and panel, small Arial, taupe/dark grey):
     "Blended scores from 4 stakeholder groups at 25% each; thresholds approved 12 October 2026."

- **User actions:**
  - Click E, S or G to toggle that category on/off (any combination, including all off).
  - Hover (or keyboard-focus) a dot or a list row to see the tooltip / cross-highlight.
  - Click "Download as PNG" to save the current filtered view.

- **What happens next:**
  - Toggling OFF: that category's dots disappear from the chart entirely (not faded), its list rows grey out, zone counts and "Showing X of 16" update. Toggling ON reverses this. A short fade/scale transition (≤200 ms) is acceptable but not required.
  - All toggles OFF: the chart shows only the zones and axes, with a centred muted message "No categories selected", counts all 0, "Showing 0 of 16 topics".
  - Download: browser saves the PNG described in Section 3; nothing on screen changes.
  - Reload: everything resets to all categories ON.

---

## Section 9 — Logic and Calculations

**What is calculated or scored:** (1) The materiality classification of each topic. (2) The geometry of the four background zones. (3) Zone counts for visible topics.

**Inputs:** Each topic's Financial score (x) and Impact score (y) from the embedded dataset (Section 5), on a 1.0–5.0 scale. Plus the current E/S/G toggle state.

**Formula or rules:**

Source: "Materiality Threshold Definitions", Stakeholder Engagement Strategy, The Corporate, approved 12 October 2026. **Priority rule:** classifications are evaluated top-down; the highest matching classification always applies. A strong score on one axis cannot be overridden by a weak score on the other.

Per topic, with F = financial score and I = impact score:

```
if (F >= 3.5 && I >= 3.5)        -> "Highly Material"
else if (F >= 3.0 || I >= 3.0)   -> "Material"
else if (F >= 2.0 || I >= 2.0)   -> "Watch List"
else                             -> "Not Material"
```

Boundary interpretation (confirmed in interview): the memo's "2.0–2.9" range is treated as **2.0 ≤ score < 3.0**, so no score (e.g. 2.95) can fall into a gap between Watch List and Material. Exactly 3.5 counts as Highly Material (if the other axis also ≥ 3.5); exactly 3.0 counts as Material; exactly 2.0 counts as Watch List.

**Zone geometry on the chart** (x = Financial, y = Impact, both 1.0–5.0) — drawn to match the rules above exactly, painted bottom layer first:
1. **Watch List** — the whole region where max(F, I) ≥ 2.0 (paint the full plot area light grey first, then overlay the others).
2. **Not Material** — square x ∈ [1.0, 2.0), y ∈ [1.0, 2.0).
3. **Material** — the L-shaped region where x ≥ 3.0 OR y ≥ 3.0 (i.e. the plot area minus the square x < 3.0, y < 3.0).
4. **Highly Material** — square x ∈ [3.5, 5.0], y ∈ [3.5, 5.0], painted on top.

Resulting picture: a red square in the top-right corner, an orange L-band wrapping it (top strip from y = 3.0 and right strip from x = 3.0), a light-grey L-band from 2.0 to 3.0, and a mid-grey square in the bottom-left corner.

**Zone counts:** count classifications over topics whose category toggle is ON.

**Output:** A zone label per topic (used in tooltip and counts) and the zone background on the chart.

**Expected results with the 2026 data (use as a verification check):**
- **Highly Material (8):** 01 GHG Emissions, 02 Energy Management, 04 Hazardous Materials, 06 Upstream Supply Chain Emissions, 09 Occupational Health & Safety, 10 Labor Rights, 11 Responsible Minerals, 16 Multi-Jurisdictional Regulatory Compliance.
- **Material (8):** 03 Water Stewardship, 05 Waste Generation, 07 Air Quality, 08 Product End-of-Life, 12 Community Impact, 13 Workforce Development, 14 DEI, 15 ESG Governance.
- **Watch List (0)** and **Not Material (0)** — zones are still drawn and their counts shown as 0.

**Edge cases:**
- Scores are fixed and all lie within 1.0–5.0; no missing values. The code should still clamp any value outside 1–5 to the plot edge rather than draw off-chart (defensive, for future data edits).
- All toggles off: see Section 8 ("No categories selected").
- Near-coincident dots: collision handling per Section 8; tooltip shows true values.

---

## Section 10 — Brand and Visual Direction

**Brand reference:** No brand skill file — brand derived from The Corporate's own documents ("ESG Material Topics of Focus" and "Stakeholder Engagement Strategy"), confirmed by the builder.

If no brand skill:
- **Primary colour:** Black `#000000` (header bar, headings, buttons); text on dark = off-white `#F2F2F2`.
- **Secondary colour:** Sand `#EAE4D5` (control bar, panel backgrounds) and Taupe `#B6B09F` (rules, borders, footnote/secondary text). Off-white `#F2F2F2` as page background.
- **Font:** Georgia for titles and section headings; Arial for body, labels, axis text, buttons, tooltips. System fonts only — no web-font downloads.
- **Logo:** No image logo available. The logo is the text wordmark **"[ C ] THE CORPORATE"** as used at the top of both source documents; render it as styled text.

**Data colours (builder-specified, outside the brand palette by design):**
- Dots: E green `#2E8B3E`, S yellow `#F2C200`, G red `#C62828`, all with dark `#1A1A1A` outline.
- Zones (soft tints so red/green/yellow dots remain visible on every zone — in particular the red G dot 16 sits in the red Highly Material zone): Highly Material `#F4CFCB`, Material `#FAD9B5`, Watch List `#ECECEC`, Not Material `#C9C9C9`.
- Light theme only (it is presented on projectors and exported to slides).

**Visual feel:** Professional and corporate — restrained, high-contrast, generous whitespace, large enough type to read on a projector from the back of a room (axis titles ≥ 14 px, dot numbers ≥ 11 px bold, list ≥ 13 px).

**Reference or inspiration:** The cover and table styling of The Corporate's two source documents (black header band with off-white text, sand table headers, Georgia display type).

---

## Section 11 — API and Credentials

| Service | What it does in this tool | Key required | Where key is stored |
|---------|--------------------------|-------------|-------------------|
| None | The tool uses no external service, API, database or network request. | None | N/A |

> **Security rule — no exceptions:** No API key, token, password, or credential may appear in any HTML file, any JavaScript file, or any file committed to GitHub. (No credentials exist for this tool.)

**Credentials readiness — filled during the architect interview for every active arm:**

| Credential | Status | Where to get it |
|-----------|--------|----------------|
| None required | N/A — the only active arm (PNG export) runs entirely in the browser | N/A |

---

## Section 12 — Out of Scope — Phase 2

The builder confirmed there are no features to defer for this version. Claude Code must build only what is specified in this document — in particular, no score editing, no file upload, no prior-year comparison, no CSV/PDF export, and no deployment.

| Deferred feature | Reason it is deferred |
|-----------------|----------------------|
| None recorded | Builder confirmed nothing is consciously deferred |

---

## Section 13 — Acceptance Criteria

| # | What to verify | Expected result | Done? |
|---|---------------|-----------------|-------|
| 1 | Open `index.html` by double-click with Wi-Fi off | Page loads fully with no console errors and no network requests (check DevTools Network tab is empty) | [ ] |
| 2 | Header | Shows "[ C ] THE CORPORATE" wordmark and title "Double Materiality 2026" with "Author: Ajay" in brand colours and fonts | [ ] |
| 3 | Axes | X = "Financial Materiality" 1–5, Y = "Impact Materiality" 1–5, gridlines at 0.5 steps | [ ] |
| 4 | Background zones | Four zones drawn with the exact geometry of Section 9: Highly Material square from 3.5/3.5, Material L from 3.0, Watch List L from 2.0, Not Material square below 2.0/2.0; colours match Section 10; zone name labels visible | [ ] |
| 5 | Dot positions and colours | 16 dots at the exact coordinates in Section 5; 8 green (E), 6 yellow (S), 2 red (G); each numbered and outlined | [ ] |
| 6 | Overlap | Dots 01 and 06 (and any other near pair) are both readable; leader lines connect nudged dots to true positions | [ ] |
| 7 | Classification | Tooltips and counts give exactly the 8 Highly Material / 8 Material / 0 / 0 split listed in Section 9 | [ ] |
| 8 | Boundary rule | A test value set in a scratch copy confirms 3.5/3.5 → Highly Material, 3.0/1.0 → Material, 2.95/1.0 → Watch List, 2.0/1.0 → Watch List, 1.99/1.99 → Not Material | [ ] |
| 9 | Toggles | All ON at load; each toggle removes/restores only its category's dots; list rows grey out; works in every combination | [ ] |
| 10 | Live counts | With only G on: Highly Material 1, Material 1, Watch List 0, Not Material 0, "Showing 2 of 16 topics"; all off: all 0 and "No categories selected" | [ ] |
| 11 | Tooltip | Hover and keyboard focus on any dot show number, full name, category, both scores to two decimals, zone; list ↔ dot cross-highlight works | [ ] |
| 12 | Side panel | Numbered list grouped E/S/G, zone legend with the exact threshold wording, dot legend, zone counts | [ ] |
| 13 | Footnote | "Blended scores from 4 stakeholder groups at 25% each; thresholds approved 12 October 2026." is visible | [ ] |
| 14 | PNG export | Button downloads a 2× PNG from `file://` in Chrome and Edge; image contains header, chart, visible dots only, list (greyed rows as on screen), legend, counts and footnote, but no buttons; filename per Section 3 | [ ] |
| 15 | Fit | Whole view visible without scrolling at 1366 × 768 and 1920 × 1080; stacks cleanly below ~900 px | [ ] |
| 16 | Single file | Deliverable is one `index.html` with no external references (no `src=`/`href=` to other files or URLs) | [ ] |

---

## Section 14 — Build Path

**This tool's tier:** Tier 1

---

### Pre-build steps — complete these before opening Claude Code

- [x] Tool Architect skill — interview complete, this spec is written and confirmed by the builder
- [ ] Project Governor skill — CLAUDE.md and PROGRESS.md produced from this spec
- [ ] GitHub repo created by the builder
- [ ] product-spec.md uploaded to the GitHub repo root
- [ ] CLAUDE.md uploaded to the GitHub repo root
- [ ] PROGRESS.md uploaded to the GitHub repo root
- [ ] Brand skill file uploaded to the GitHub repo root — N/A, brand is defined in Section 10
- [ ] Netlify connected to the GitHub repo — N/A, the tool is not deployed
- [ ] All credentials identified — N/A, no credentials required

> Claude Code organizes these files into the correct folder structure (docs/, .claude/skills/) automatically at the start of the first session.

---

### Tier 1 — build session

- [ ] Open Claude Code in the project folder (GitHub repo; no Netlify connection)
- [ ] Claude Code runs First Session Setup: creates docs/, moves reference files
- [ ] Claude Code reads product-spec.md, CLAUDE.md, and PROGRESS.md
- [ ] Claude Code builds the tool as a single self-contained `index.html` at the repo root
- [ ] Test locally: open `index.html` by double-click (file://) with network disabled and run every Section 13 criterion
- [ ] **Deployment:** none — commit and push to main for version safety; the builder copies `index.html` to their computer and opens it for meetings

---

## Section 15 — Open Questions

| Question | Who answers it | Blocking? |
|----------|---------------|-----------|
| None — all decisions were resolved in the interview. | — | No |

---

## Section 16 — Tool Version History

| Version | Date | What changed in the tool |
|---------|------|--------------------------|
| v1.0 | 12 October 2026 | Initial build |
| v1.1 | 12 October 2026 | Assessment year changed to 2026 throughout; title changed to "Double Materiality 2026"; "Author: Ajay" added to header and PNG export; PNG filename year changed to 2026; threshold approval date changed to 12 October 2026 |

---

*This spec is written for Claude Code. It assumes zero prior context. Every decision, rule, and requirement must be explicit enough that the builder can hand this document to Claude Code without a single verbal explanation.*
