# NFL Fan Dashboard — Tableau Build Guide

## Data Sources (5 CSV files)

| File | Primary Use | Key Fields |
|---|---|---|
| `nfl_team_popularity.csv` | Overview & Correlation sheets | team, conference, tv_viewers_m, google_trends_index, stadium_attendance_pct, composite_score, win_pct, home_state_fan_pct |
| `nfl_season_trends.csv` | Trend lines sheet | season, team, google_trends_index, tv_viewers_m, stadium_attendance_pct |
| `nfl_fan_geography.csv` | Map sheet | team, state, state_abbr, fan_interest_index |
| `nfl_win_loss.csv` | Win/Loss detail & Correlation | season, team, wins, losses, win_pct, made_playoffs, super_bowl_win |
| `nfl_championship_effect.csv` | Championship lift chart | relative_year, label, avg_popularity_change_pct |

---

## Recommended Data Model

Connect all 5 CSVs. Use **Relationships** (not Joins) in Tableau's logical layer:

```
nfl_team_popularity  ──[team]──  nfl_season_trends
nfl_team_popularity  ──[team]──  nfl_fan_geography
nfl_team_popularity  ──[team]──  nfl_win_loss
nfl_championship_effect  (standalone — no join needed)
```

---

## Sheet 1 — Team Popularity Ranking (Horizontal Bar Chart)

**Data source:** `nfl_team_popularity`

**Steps:**
1. Drag `team` → Rows
2. Drag `composite_score` → Columns
3. Sort descending by `composite_score`
4. Drag `conference` → Color (use AFC = Steel Blue, NFC = Gold)
5. Drag `composite_score` → Label, format as integer
6. Right-click X axis → Edit axis → Fixed range 0–100, title "Composite Score (0–100)"
7. Add a **Reference Line** at 70 (dashed) labeled "Top Tier Threshold"
8. Format: remove gridlines, set bar size to ~60%, title "Team Popularity Ranking"

**Calculated field — Composite Score (verify):**
```
([tv_viewers_m] / 21.4 * 33)
+ ([google_trends_index] / 98 * 33)
+ ([stadium_attendance_pct] / 100 * 34)
```

---

## Sheet 2 — Fan Geography Map (Filled Map)

**Data source:** `nfl_fan_geography`

**Steps:**
1. Double-click `state` — Tableau auto-geocodes to a filled US map
2. Drag `fan_interest_index` → Color → choose **Blue-Teal sequential** palette, reversed (high = dark)
3. Drag `team` → Filters shelf → show as a **Single Value dropdown** quick filter
4. Drag `fan_interest_index` → Tooltip
5. Drag `state` → Tooltip
6. Edit tooltip: `<State>: Interest Index <fan_interest_index>`
7. Format: remove state borders or set to 1pt white, title "Fan Interest by State"
8. Set map layers: washout background style, turn off irrelevant layers

**Key insight annotation:** Add a floating text box — *"Cowboys fans appear in all 50 states; Packers skew 72% Wisconsin"*

---

## Sheet 3 — Home vs Away Fan Ratio (Stacked Bar)

**Data source:** `nfl_team_popularity`

**Calculated fields to create:**
```
// Away fan pct
[Away Fan %] = 100 - [home_state_fan_pct]
```

**Steps:**
1. Reshape into long format using Tableau's **Pivot** on the data source:
   - Select `home_state_fan_pct` column → right-click → Pivot
   - Rename pivot field names: "Fan Type", pivot field values: "Fan %"
   - Add a second row manually for Away % using a calculated field
2. **Simpler approach — use a Gantt chart trick:**
   - Drag `team` → Rows, sort by `home_state_fan_pct` descending
   - Drag `home_state_fan_pct` → Columns (this is the Home bar)
   - Duplicate: drag `[Away Fan %]` → Columns (second axis)
   - Right-click second axis → Dual Axis → Synchronize
   - Set first mark type = Bar (Gold color), second = Gantt Bar (Blue)
3. Format: Add labels showing % on each segment, title "Home State vs National Fanbase"

---

## Sheet 4 — Popularity Trend Lines (2015–2024)

**Data source:** `nfl_season_trends`

**Steps:**
1. Drag `season` → Columns, right-click → Discrete (to show all years)
2. Drag `google_trends_index` → Rows
3. Drag `team` → Color — assign specific colors:
   - Chiefs: #FF4D6D · Cowboys: #60A5FA · Patriots: #818CF8
   - Eagles: #34D399 · Steelers: #FDE047 · Rams: #FB923C
4. Drag `team` → Filters → Show filter as **Multi-select checkbox**
5. Change mark type to **Line**, set size to Medium
6. Add **Circle** marks on each data point (Marks card → add second layer)
7. Right-click Y axis → Edit axis → Fixed 0–100, title "Google Trends Index"
8. Add annotation on Chiefs 2019–2024 surge: *"Post-Mahomes era: +250% growth"*
9. Title: "Team Popularity Trend (2015–2024)"

**Optional dual axis:** Add `tv_viewers_m` as a second axis to show TV ratings alongside trends.

---

## Sheet 5 — Win Rate vs Popularity (Scatter / Bubble Chart)

**Data source:** `nfl_team_popularity`

**Steps:**
1. Drag `win_pct` → Columns
2. Drag `composite_score` → Rows
3. Change mark type to **Circle**
4. Drag `stadium_attendance_pct` → Size (larger bubble = more attendance)
5. Drag `conference` → Color (AFC = #38BDF8 Cyan, NFC = #F5C518 Gold)
6. Drag `team` → Label → set label to "Always show"
7. Right-click X axis → Edit axis → Fixed 30–80, title "Win Rate (%)"
8. Right-click Y axis → Edit axis → Fixed 25–100, title "Composite Popularity Score"
9. Add **Trend Line**: Analytics pane → drag "Trend Line" → Linear, per conference
10. Add **Reference Lines** at 50% win rate (vertical) and 60 score (horizontal) to create quadrants
11. Title: "Win Rate vs Fan Popularity"

**Quadrant labels (floating text boxes):**
- Top-right: "Champions & Beloved"
- Top-left: "Legacy Brands" (Cowboys, Packers)
- Bottom-right: "Winners, Still Growing"
- Bottom-left: "Rebuilding"

---

## Sheet 6 — Season Win/Loss Record (Heat Map)

**Data source:** `nfl_win_loss`

**Steps:**
1. Drag `season` → Columns
2. Drag `team` → Rows, sort by average `win_pct` descending
3. Drag `win_pct` → Color → **Orange-Blue Diverging** palette, center at 50
4. Drag `wins` → Label
5. Drag `made_playoffs` → Shape or add a border indicator
6. Add `super_bowl_win` as a secondary mark: filter to 1, show as a star/trophy shape
7. Title: "Season-by-Season Win Rate (2015–2024)"

---

## Sheet 7 — Championship Popularity Lift (Bar Chart)

**Data source:** `nfl_championship_effect`

**Steps:**
1. Drag `label` → Columns (set sort order manually: Year Before → Champ Year → 1yr → 2yr → 3yr)
2. Drag `avg_popularity_change_pct` → Rows
3. Color bars individually: Champ Year & 1yr After = Gold, others = Blue-Gray
4. Drag `avg_popularity_change_pct` → Label, format with "+" prefix
5. Add a reference line at 0
6. Title: "Avg Popularity Boost After Winning Championship"

---

## Dashboard Assembly

### Layout: 1200 × 900px (Desktop) or 1400 × 1000px

```
┌────────────────────────────────────────────────┐
│  HEADER — Title + 4 KPI text tiles             │
├──────────────┬─────────────────────────────────┤
│ TAB 1        │ TAB 2        │ TAB 3  │ TAB 4   │
│ Overview     │ Geography    │ Trends │ Corr.   │
├──────────────┴─────────────────────────────────┤
│ [Active sheet content fills this area]         │
│                                                │
│                                                │
└────────────────────────────────────────────────┘
```

**Tab navigation — use Dashboard Actions:**
1. Add 4 blank Button objects at the top as tabs
2. For each: Format Button → set text + background color
3. Dashboard → Actions → Add Action → **Navigate to Sheet**
   - Source: Button click
   - Target: corresponding sheet

### KPI Tiles (top row)
Use **Text objects** with large font for:
- Most Popular: Cowboys (94/100)
- Avg TV Viewers: 16.4M
- Avg Attendance: 94.2%
- Top Trends: Chiefs (98 index)

Format each with a colored left border using a thin colored **Blank object** beside it.

### Filters to make global (apply to all sheets):
- `team` filter (from Sheet 4) → right-click → Apply to All Using This Data Source
- `conference` filter → same

### Color legend
Add a single shared **Color Legend** object from the Conference dimension (AFC/NFC).

---

## Formatting Tips

| Element | Recommendation |
|---|---|
| Background | #0E1726 (dark navy) — set in Format → Dashboard |
| Font | Use **Tableau Book** or import Barlow via image title workaround |
| Grid lines | None — remove in Format → Lines |
| Borders | 1pt, rgba white 15% |
| Tooltips | Dark bg (#0E1726), white title, light gray body |
| Sheet titles | Hide all individual sheet titles; use a single dashboard title |
| Padding | 8px inner padding on all containers |

---

## Publishing

- **Tableau Public:** File → Save to Tableau Public (free, shareable link)
- **Tableau Server/Cloud:** Server → Publish Workbook → set permissions
- **Export as PDF:** File → Print to PDF → select "Dashboard" layout
