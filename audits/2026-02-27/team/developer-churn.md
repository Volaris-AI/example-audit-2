---
genre: team
category: developer-churn
analysis-type: git-history
relevance:
  always-include: true
---

# Developer Churn Analysis

**Assessment Date:** _2026-02-27_
**Assessment Window:** _2 months (2025-12-27 to 2026-02-27)_
**Status:** _Complete_

---

<!-- analysis: git-history -->

## Executive Summary

**Team Stability Maturity:** _3_ / 5

**Current Team Size:** _1_ active human developer
**Churn Rate:** _0%_ (annual projected)

**Key Findings:**
- This repository is a shallow/grafted clone of jQuery v2.2.5-pre with only **2 total commits** in its entire visible git history, both dated 2026-02-27. This severely limits the depth of churn analysis.
- **1 human developer** (Riley Roberts) contributed 1 commit in the assessment window. 1 automated bot commit (`copilot-swe-agent[bot]`) was excluded from analysis.
- Observed churn rate is **0%** — no developers departed during the assessment window. However, this reflects the minimal history of the snapshot repository rather than a mature, stable team.
- Team Stability Maturity is scored **3 / 5 (Proficient)** because while churn is 0%, developer tenure cannot be validated from this shallow clone, and the single-developer composition is a bus-factor risk in itself.

---

## Churn Metrics

### Overall Team Stability

| Metric | Value | Status |
|--------|-------|--------|
| Active Developers (Assessment Period) | 1 | — |
| New Developers | 1 | — |
| Departed Developers | 0 | — |
| Churn Rate (Assessment Period) | 0% | 🟢 |
| Projected Annual Churn Rate | 0% | 🟢 |
| Average Developer Tenure (Visible) | 0 days (same-day first/last commit in clone) | ⚠️ Cannot determine |

> **Important Caveat:** The repository is a grafted/shallow clone. Both commits exist on the same date (2026-02-27). True developer tenure cannot be calculated from this history alone.

### Churn Rate Interpretation

| Rate | Status | Description |
|------|--------|-------------|
| 0-10% | 🟢 Excellent | Very stable team, low turnover |
| 11-20% | 🟡 Good | Healthy turnover, manageable |
| 21-30% | 🟠 Moderate | Elevated turnover, monitor closely |
| 31%+ | 🔴 High | Concerning turnover, investigate causes |

**Observed rate: 0% — however, insufficient history prevents full confidence in this rating.**

---

## Developer Activity Timeline

### Active Developers

Developers with commits in the assessment window (2025-12-27 to 2026-02-27). Automated bot commits are excluded.

| Developer | Email | First Commit | Last Commit | Tenure (Visible Days) | Commit Count | Status |
|-----------|-------|--------------|-------------|----------------------|--------------|--------|
| Riley Roberts | syanification@gmail.com | 2026-02-27 | 2026-02-27 | 0 (shallow clone) | 1 | Active |

> **Excluded (bot):** `copilot-swe-agent[bot]` (198982749+Copilot@users.noreply.github.com) — 1 automated commit (`4269cbb`) "Initial plan" on 2026-02-27.

### New Developers

Developers who made their first commit in the assessment window.

| Developer | Email | First Commit | Commit Count | Onboarding Status |
|-----------|-------|--------------|--------------|-------------------|
| Riley Roberts | syanification@gmail.com | 2026-02-27 | 1 | Active — first (and only) commit in window |

> Note: Because the clone is grafted, Riley Roberts' first visible commit is the root commit of the snapshot. This does not necessarily mean they are a new contributor to the jQuery project — they are the fork/snapshot owner.

### Departed Developers

Developers whose last commit was more than 60 days ago (likely no longer on team).

| Developer | Email | First Commit | Last Commit | Tenure (Days) | Last Commit Gap |
|-----------|-------|--------------|-------------|---------------|-----------------|
| — | — | — | — | — | — |

> No departed developers identified in the assessment window.

---

## Team Stability Maturity Score

The Team Stability Maturity score (1-5) is based on churn rate and team tenure:

### Scoring Rubric

| Level | Score | Annual Churn Rate | Avg Tenure | Description |
|-------|-------|-------------------|------------|-------------|
| **5** | Exceptional | 0-10% | 18+ months | Very stable team, minimal turnover |
| **4** | Strong | 11-15% | 12-18 months | Stable team with healthy growth |
| **3** | Proficient | 16-25% | 8-12 months | Functional but elevated turnover |
| **2** | Developing | 26-35% | 5-8 months | High turnover affecting productivity |
| **1** | Critical | 36%+ | <5 months | Very high turnover, major instability |

### Your Score: _3_ / 5

**Justification:**
- Annual churn rate: **0%** (no departures observed)
- Average tenure (visible): **0 days** — indeterminate due to shallow/grafted clone; cannot confirm tenure thresholds
- While the churn rate of 0% would suggest a score of 5, the **severe limitations of the git history** (only 2 commits, both on 2026-02-27, one from a bot) prevent confidence in that rating.
- The **single human developer** composition represents a bus-factor risk: all knowledge is concentrated in one person.
- Score of **3 (Proficient)** reflects the operational baseline given available evidence: no observable instability, but insufficient history to validate tenure or long-term stability.

---

## Churn Analysis

### Departure Patterns

**Recent Departures:** 0 developers in the past 2 months.

**Potential Risk Areas:**
- **Bus-factor risk:** With a single visible human contributor, there is no redundancy. Any departure would represent 100% team turnover.
- The underlying jQuery codebase (v2.2.5-pre) was developed by a larger team (see `AUTHORS.txt`: John Resig, Dave Methvin, Timmy Willison, Richard Gibson, Michał Gołębiowski-Owczarek, and many others). None of these historical contributors appear in the visible git history of this snapshot.
- The shallow clone architecture means knowledge and context from the original jQuery contributors is not captured in this repository's git history.

### Onboarding Patterns

**Recent Hires:** 1 developer in the past 2 months (Riley Roberts — fork/snapshot owner).

**Onboarding Effectiveness:**
- Average time to first meaningful commit: 0 days (single-day snapshot)
- Average commits in first month: 1 commit on first day
- This is a snapshot/fork repository context; traditional onboarding metrics do not apply in a meaningful way.

---

## Team Health Indicators

### Positive Indicators

- ✅ **Zero observed departures** in the 2-month assessment window — no active turnover
- ✅ **No competing bot noise** — automated bot commits (copilot-swe-agent[bot]) are clearly distinguishable and appropriately excluded
- ✅ **No churn rate** — the visible team composition is completely stable within the assessment window

### Areas of Concern

- ⚠️ **Extremely limited git history**: Only 2 commits, both on 2026-02-27. This is a shallow/grafted clone with no meaningful history for trend analysis.
- ⚠️ **Single human developer (bus-factor = 1)**: Riley Roberts is the sole visible human contributor. Any departure or unavailability would be catastrophic for continuity.
- ⚠️ **Indeterminate tenure**: Average tenure cannot be calculated from available data. The scoring rubric requires tenure data to award scores of 4 or 5.
- ⚠️ **No visibility into historical jQuery contributors**: The `AUTHORS.txt` lists dozens of contributors to the original jQuery project, but none of their history is available in this clone. Knowledge risk from original contributors cannot be assessed.

---

## Analysis Methodology

### Data Collection

1. **Identified all contributors:**
   ```
   git log --all --format='%aN|%aE' | sort -u
   ```
   Result: 2 contributors (Riley Roberts, copilot-swe-agent[bot])

2. **Got first and last commit for each developer:**
   ```
   git log --all --author="syanification@gmail.com" --format='%ad' --date=short
   ```
   Result: Single date `2026-02-27` for Riley Roberts

3. **Counted commits per developer in assessment window:**
   ```
   git shortlog -sn --since="2025-12-27" --until="2026-02-27" --all
   ```
   Result: 1 commit — copilot-swe-agent[bot]; 1 commit — Riley Roberts

4. **Confirmed assessment window coverage:**
   ```
   git log --since="2 months ago" --all --oneline
   ```
   Result: Both commits fall within the 2-month window

### Churn Calculation

- **Churn Rate** = (Number of Departures / Average Team Size) × 100
- Departures in window: 0
- Average team size (human): 1
- **Assessment Period Churn: 0/1 × 100 = 0%**
- **Annual Projected Churn: 0%**

### Limitations

- **Shallow/grafted clone**: The repository has only 2 commits (both 2026-02-27). True historical churn cannot be assessed.
- **Bot exclusion**: `copilot-swe-agent[bot]` excluded from all human metrics.
- **Tenure indeterminate**: First and last visible commits are the same date for the sole human developer; cannot compute actual tenure.
- **Original jQuery contributors not visible**: `AUTHORS.txt` lists 200+ contributors but their commit history is not available in this clone. The original jQuery project team's stability cannot be analyzed.

---

## Recommendations

### To Reduce Churn

- Establish a documented succession plan for Riley Roberts given the single-contributor risk.
- If this repository is intended to host ongoing development (as opposed to being a static snapshot), onboard at least one additional developer to reduce bus-factor risk.

### To Improve Onboarding

- Document the repository purpose clearly (snapshot vs. active fork) to set expectations for future contributors.
- If active development is intended, establish contribution guidelines and link to the upstream jQuery project's practices.

### To Mitigate Knowledge Loss

- Given that the rich jQuery commit history is not present in this clone, consider either deepening the git history (`git fetch --unshallow`) or maintaining clear documentation linking this snapshot to its upstream source.
- Preserve the `AUTHORS.txt` reference to original jQuery contributors as a knowledge attribution artifact.

---

## Notes

- **Assessment Window:** 2025-12-27 to 2026-02-27
- **Departure Threshold:** 60 days of inactivity
- **Team Size:** 1 unique active human contributor in the assessment window
- **Bot Exclusions:** `copilot-swe-agent[bot]` (198982749+Copilot@users.noreply.github.com)
- **Git History Limitation:** Repository is a grafted/shallow clone of jQuery v2.2.5-pre. All source files arrived via a single root commit (`8eb4bee`) dated 2026-02-27. This fundamentally limits any historical analysis.
- **Commit SHAs Referenced:**
  - `8eb4bee8266a1435f9d4ebb01cd1de98a8199ea9` — Riley Roberts — "Use github.token as fallback when AUDIT_PAT secret is not set" — 2026-02-27
  - `4269cbb0c0f223b4483e82c7129b90737451b754` — copilot-swe-agent[bot] — "Initial plan" — 2026-02-27 (excluded)
