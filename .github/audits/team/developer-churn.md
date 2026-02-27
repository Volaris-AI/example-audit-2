---
genre: team
category: developer-churn
analysis-type: git-history
relevance:
  always-include: true
---

# Developer Churn Analysis

**Assessment Date:** _[YYYY-MM-DD]_
**Assessment Window:** _[X months]_
**Status:** _[Draft / In Progress / Complete]_

---

<!-- analysis: git-history -->

## Executive Summary

**Team Stability Maturity:** _[1-5]_ / 5

**Current Team Size:** _[Number]_ active developers
**Churn Rate:** _[Percentage]_% (annual projected)

**Key Findings:**
- _[Summary of churn patterns]_
- _[Summary of team stability]_

---

## Churn Metrics

### Overall Team Stability

| Metric | Value | Status |
|--------|-------|--------|
| Active Developers (Assessment Period) | _[N]_ | _[Status]_ |
| New Developers | _[N]_ | _[Status]_ |
| Departed Developers | _[N]_ | _[Status]_ |
| Churn Rate (Assessment Period) | _[N]_% | _[🟢/🟡/🔴]_ |
| Projected Annual Churn Rate | _[N]_% | _[🟢/🟡/🔴]_ |
| Average Developer Tenure | _[N]_ months | _[🟢/🟡/🔴]_ |

### Churn Rate Interpretation

| Rate | Status | Description |
|------|--------|-------------|
| 0-10% | 🟢 Excellent | Very stable team, low turnover |
| 11-20% | 🟡 Good | Healthy turnover, manageable |
| 21-30% | 🟠 Moderate | Elevated turnover, monitor closely |
| 31%+ | 🔴 High | Concerning turnover, investigate causes |

---

## Developer Activity Timeline

### Active Developers

Developers with commits in the assessment window.

| Developer | Email | First Commit | Last Commit | Tenure (Days) | Commit Count | Status |
|-----------|-------|--------------|-------------|---------------|--------------|--------|
| _[Name]_ | _[Email]_ | _[Date]_ | _[Date]_ | _[N]_ | _[N]_ | _[Active/New/Departing]_ |

### New Developers

Developers who made their first commit in the assessment window.

| Developer | Email | First Commit | Commit Count | Onboarding Status |
|-----------|-------|--------------|--------------|-------------------|
| _[Name]_ | _[Email]_ | _[Date]_ | _[N]_ | _[Ramping up/Productive]_ |

### Departed Developers

Developers whose last commit was more than [threshold] days ago (likely no longer on team).

| Developer | Email | First Commit | Last Commit | Tenure (Days) | Last Commit Gap |
|-----------|-------|--------------|-------------|---------------|-----------------|
| _[Name]_ | _[Email]_ | _[Date]_ | _[Date]_ | _[N]_ | _[N]_ days ago |

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

### Your Score: _[1-5]_ / 5

**Justification:**
- Annual churn rate: _[N]_%
- Average tenure: _[N]_ months
- _[Additional context about team stability]_

---

## Churn Analysis

### Departure Patterns

**Recent Departures:** _[Number]_ developers in the past _[timeframe]_

**Potential Risk Areas:**
- _[Analysis of knowledge areas at risk due to departures]_
- _[Analysis of single points of failure]_

### Onboarding Patterns

**Recent Hires:** _[Number]_ developers in the past _[timeframe]_

**Onboarding Effectiveness:**
- Average time to first meaningful commit: _[N]_ days
- Average commits in first month: _[N]_
- _[Assessment of onboarding quality]_

---

## Team Health Indicators

### Positive Indicators

- ✅ _[Positive finding 1]_
- ✅ _[Positive finding 2]_
- ✅ _[Positive finding 3]_

### Areas of Concern

- ⚠️ _[Concern 1]_
- ⚠️ _[Concern 2]_
- ⚠️ _[Concern 3]_

---

## Analysis Methodology

### Data Collection

1. **Identify all contributors:**
   ```bash
   git log --all --format='%aN|%aE' | sort -u
   ```

2. **Get first and last commit for each developer:**
   ```bash
   git log --all --author="[email]" --format='%ad' --date=short | head -1  # last commit
   git log --all --author="[email]" --format='%ad' --date=short | tail -1  # first commit
   ```

3. **Count commits per developer in assessment window:**
   ```bash
   git shortlog -sn --since="[start_date]" --until="[end_date]" --all
   ```

### Churn Calculation

**Churn Rate** = (Number of Departures / Average Team Size) × 100

- **Assessment Period Churn:** Based on departures in the assessment window
- **Annual Projected Churn:** Assessment period churn extrapolated to 12 months

**Departure Detection:** A developer is considered "departed" if:
- Last commit was more than [threshold] days ago (typically 60-90 days)
- No activity in assessment window despite historical contributions

### Limitations

- Git history may not reflect all team changes (e.g., non-coding roles)
- Departures are inferred from commit inactivity, not HR data
- External contractors may appear as churn if they complete their engagement
- Part-time contributors may be incorrectly flagged as departed

---

## Recommendations

### To Reduce Churn

- _[Recommendation 1 based on churn analysis]_
- _[Recommendation 2]_

### To Improve Onboarding

- _[Recommendation 1 based on new developer patterns]_
- _[Recommendation 2]_

### To Mitigate Knowledge Loss

- _[Recommendation 1 based on departure patterns]_
- _[Recommendation 2]_

---

## Notes

- **Assessment Window:** _[Start date]_ to _[End date]_
- **Departure Threshold:** _[N]_ days of inactivity
- **Team Size:** Counted as unique active contributors in the assessment window
