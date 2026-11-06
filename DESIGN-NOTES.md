# PAIMANA Portfolio Dashboard: redesign notes

Prototype: `index.html` (one self-contained file; Chart.js 4.4.1 from cdnjs). Open it in any browser.
It runs on **generated sample data** calibrated to the Ministry of Power portfolio (106 projects, ₹5,85,130 cr original cost) plus the three real outlier cases from the requirement doc. Replace `generateProjects()` with the Flash Report feed. All business rules live in `RULE`, `derive()`, `classify()` and `computeMetrics()`.

## 1. Research findings

**Current dashboard (LMDashboard/DashBoard)**
- Six pastel KPI tiles and a state map compete for attention; nothing says what needs action.
- The agency donut repeats "Adani Transmission Limited" six times in the legend. It is unreadable.
- The "Cost overview" funnel implies stages that don't exist. It also hides the escalation amount and the balance to spend.
- Each card has a Chart/Data switch. This is a good pattern and is kept.
- Filters need a separate **Show Data** click, with no sign of which filters are active.
- The Data tables mix cost formats, e.g. `4500 (4500)`.
- None of the analytics framework from the doc is present.

**CRIP site (design reference)**
- Noto Sans, navy `#0A3372`, sky blue `#0B98D5`, saffron `#F47920`, 5px radius inputs, UX4G accessibility widget.
- The redesign uses the same font and colours so both PAIMANA products feel like one system.
- Both portals are behind login and captcha, so tokens were taken from the public CRIP page. The current layout came from the screenshots provided.

## 2. Information architecture

```
Government header (GIGW, same as the live PAIMANA / CRIP portal):
  strip: Government of India ↗ · Skip to Main Content · A− A A+ · Contrast · Accessibility "More" · Role
  logo bar: MoSPI emblem logo | Data for Development | PAIMANA ……… signed-in user (name, ministry)
  Logos: assets/ (taken from paimana-crip.mospi.gov.in), embedded in index.html so it works as one file.
Nav rail (collapsible): Dashboard · Project Explorer · Review Cases · Log out
Page header: title · Flash Report month · data-as-of date · Export CSV
Scope + filters: Under Implementation | Newly Added | Reported | Not Reported (with counts)
                 Agency · State/UT · Cost band · Revision status · active-filter chips · Reset
Tabs:
 ├─ Overview: 6 KPIs → 2×2 chart pairs: Cost waterfall + India State map · Agency bars + Physical progress · Cost overrun + Time overrun
 ├─ Performance Analytics (badge = flagged count): 3 metric groups → 3 early-warning scatter plots
 │    with Non-revised / Revised switch → flagged-projects table (search, filter, sort)
 │    Selecting a chart lists only the projects it flags (charts 1–2: lagging or lapsed; chart 3: misaligned),
 │    within that chart's group. The table's filter has the three chart names under "By chart" and stays in sync
 │    with the selected card; select the card again (or choose "All flags") to clear. Dots still open project details.
 └─ Data Scrutiny (badge = outlier count): rule summary → outlier table with failing cells highlighted
Project page (full page, replaces the earlier side drawer): opens from any flagged-projects or data-scrutiny row,
  or any dot on the early-warning charts. Link: #project-<ID>. Back link and Esc return to the list at the same scroll position.
  Sections follow "project dashboard.docx":
   1. Title, ID · agency · State, revision and reporting pills, flags
   2. Six headline tiles: Original cost · Revised cost · Expenditure (with change from last quarter) · Start date ·
      Original commissioning · Revised commissioning (with months of delay)
   3. Physical progress and Financial progress donuts, with time elapsed, gap to schedule and balance to spend
   4. Time overrun (TOR) and Cost overrun (COR), each with the reported causes
   5. Project details: sector, implementing agency, location, implementation mode, brief description
   6. Project timeline: start, schedule/cost revisions, original and revised commissioning, today
   7. Project progress timeline: quarterly physical vs financial progress line chart. Both lines draw left to
      right, one quarter after another (~1.8 s, linear) when the page opens; Replay re-runs it. Donuts sweep in.
      No animation when the viewer has reduced motion switched on.
  Sample-data note: the Flash Report fields used here are real model fields; quarterly history, TOR/COR causes,
  last-quarter change and implementation mode are derived sample values until the feed supplies them.
```

Key changes from the current page:
- Filters apply instantly. The Show Data button is removed and active filters show as removable chips.
- A **cost waterfall** replaces the funnel: Original → Escalation → Revised → Expenditure → Balance.
- **Agency donut → ranked horizontal bars**, with a Projects / Revised cost / Expenditure switch. Agencies beyond the top 8 are grouped as "Others".
- **India State map (choropleth) plus ranked list.** The map is embedded in the page (about 46 KB, no external requests). Hovering a State shows its project count and original cost; tapping on a phone shows the same details, and a second tap filters. Selecting a State filters the page, and keyboard users can tab through States. Boundaries follow India's official map: Ladakh includes Gilgit-Baltistan and Aksai Chin, and Jammu & Kashmir is shown in full. Source: open data from github.com/udit-001/india-maps-data, merged from districts to States. *Before go-live, have the boundaries checked against the Survey of India outline.*
- Every chart keeps its Chart/Table toggle. Tables use one format: `Original (**Revised**)`.

## 3. How the doc maps to the UI

| Requirement | Where | Rule used |
|---|---|---|
| Baseline Slippage Rate | Analytics → Project planning | (Revised + Need revision) ÷ Total active × 100 |
| 80-20 Need Revision | flag, metric, table filter | not revised AND time > 80% AND (physical < 20% OR expenditure < 20%) |
| Cost-increase quantum | Planning sub-metric | Σ(revised − original) ÷ Σ original, cost-revised (upward) projects only |
| Time-span increase quantum | Planning sub-metric | Σ(revised end − original end) ÷ Σ original span, schedule-revised projects |
| Multiple revisions (cost and schedule separately) | Planning sub-metrics | count with ≥ 2 revisions ÷ total projects |
| Execution Performance Rate | Analytics → Project execution | non-revised, time > 25%; on track = time − physical < 30 |
| Administrative Lag Rate | Analytics → Monitoring | Need revision ÷ (Revised + Need revision) × 100 |
| Scatter 1: Time vs Progress | Early-warning charts | Non-revised = schedule not revised; Revised = schedule revised |
| Scatter 2: Time vs Expenditure | Early-warning charts | Non-revised = cost not revised; Revised = cost revised |
| Scatter 3: Progress vs Expenditure | Early-warning charts | Non-revised = neither revised; Revised = either revised |
| 4-status colours (charts 1 and 2) | dot colour **and shape** | Lapsed: time > 100% and incomplete, or expenditure > 100% of cost · Initiation: time < 25% · Lagging: gap ≥ 30 · otherwise As per baseline |
| Aligned / Misaligned (chart 3) | dot colour and shape | \|expenditure − physical\| < 30 means aligned |
| Outlier handling | Data Scrutiny tab | removed from every chart and metric, listed with the failing cells |

Each scatter plot also draws a y = x "on baseline" line, the 30-point threshold line with a shaded zone, the initiation band and a 100%-time marker. A reviewer can see why a dot is flagged without knowing the formula. Each metric card has a "How it is calculated" disclosure that quotes the formula.

## 4. Ambiguities in the doc: assumptions to confirm with the client

1. **Lagging baseline** is described as "significant delta (<30 points)". Every other rule in the doc uses ">30". The prototype treats **≥ 30** as lagging.
2. **"AND/OR"** in the 80-20 rule is read as **OR**.
3. **Administrative Lag formula** has unbalanced brackets. It is read as `Need ÷ (Revised + Need) × 100`.
4. **Initiation phase** is "< 25%" for the charts but "≤ 25" for the execution metric. Both are kept exactly as written.
5. **Expenditure %** is measured against the latest cost (revised if available, otherwise original).
6. **Outlier thresholds** have no numbers in the doc. The prototype uses: expenditure > 200% of cost; revised commissioning before start; original span > 20 years. All three are configurable in `RULE`.
7. **Outliers stay in Overview totals** so the totals match the Flash Report. They are excluded only from analytics.
8. **Month-over-month deltas** need revision history per report. The sample data has static revisions, so the deltas read "no change"; with real monthly snapshots they will move.

## 5. Test and audit log

Functional (scripted in the browser):
- All three metrics match an independent recalculation from the doc's formulas.
- All three doc outliers are caught with the correct reason (617989 at 8,908% of cost; 618106 revised date before start; 603293 with a 2056 date). None of them appears in any chart.
- Scatter point counts equal the group selection for all three charts.
- The agency filter, chips, Clear all, month switch, tab switching, legend toggles, table sorting and search, and the project page all work. Escape or Back returns from the project page to the list at the same scroll position.

Visual:
- Checked at 1440 px light, 1440 px dark (theme switch), and 390 px phone width.
- No horizontal page overflow at 390 px. Wide tables scroll inside their own container.

Issues found and fixed during the audit:
- Hash-only tab switching failed in sandboxed previews. Switching is now click-driven.
- An agency label was clipped. Short names are used on the axis, with full names in the tooltip.
- KPI labels wrapped unevenly. Labels were shortened.
- The scatter x-axis ran to 200%. It now ends at 140%, and points beyond that sit on the edge with their true value in the tooltip.
- Reference-line labels overlapped dots. They were repositioned.
- The issue column in the scrutiny table was cut off. It now sits next to the project name.
- On phones the filters filled the first screen. They are now behind a Filters button.
- The waterfall labels collided on phones. A compact `₹x.xxL` format is used on narrow charts.

Accessibility:
- Status is never shown by colour alone: dots also differ in shape, and legends and flags have text labels.
- Every chart has a table view. Focus outlines are visible. Table rows can be opened with the keyboard. The theme follows the OS and has a manual switch. Reduced motion is respected.

Known gaps for the build team:
- Tabs use `role="tab"` but don't have arrow-key navigation yet.
- The amber "warning" colour is below 3:1 contrast on white. Shape and label carry the meaning.
- Project Explorer and Review Cases are placeholders.
