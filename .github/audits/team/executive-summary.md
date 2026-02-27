---
genre: team
category: executive-summary
analysis-type: git-history
relevance:
  file-patterns: []
  keywords: []
  config-keys: []
  always-include: true
---

# Team Assessment Executive Summary

<!-- analysis: git-history -->

## Assessment Overview
**Assessment Period:** [START_DATE] to [END_DATE]  
**Team Size:** [X] active developers  
**Assessment Completed:** [COMPLETION_DATE]

---

## Executive Summary

**Team Health Score:** [X] / 100 (Based on Team Stability)

**Team Stability Maturity:** [X] / 5

**Key Focus Areas:**
- Developer churn and team stability
- Security vulnerability attribution and accountability

**Assessment Verdict:** [One paragraph summary of team stability and security awareness]

---

## At a Glance

<details open>
<summary><b>📊 Team Stability Metrics</b></summary>

| Metric | Value | Status |
|--------|-------|--------|
| **Team Stability Maturity** | [X] / 5 | [🟢/🟡/🔴] |
| **Active Developers** | [N] | — |
| **Churn Rate (Annual Projected)** | [X]% | [🟢/🟡/🔴] |
| **Average Developer Tenure** | [X] months | [🟢/🟡/🔴] |
| **New Developers** | [N] | — |
| **Departed Developers** | [N] | — |

</details>

<details open>
<summary><b>🔒 Security Accountability Metrics</b></summary>

| Metric | Value |
|--------|-------|
| **Total Vulnerabilities Analyzed** | [N] |
| **Developers with Committed Vulnerabilities** | [N] |
| **Developers with Approved Vulnerabilities** | [N] |
| **Critical Vulnerabilities** | [N] |
| **High Vulnerabilities** | [N] |
| **Medium Vulnerabilities** | [N] |
| **Low Vulnerabilities** | [N] |

</details>

---

## Team Health Score Calculation

**Score:** [X] / 100

The team health score is **entirely based on team stability (churn)**:

**Base Score:** Team Stability Maturity × 20 = [X × 20]

**Adjustments:**
- Tenure bonus: +[X] points (if average tenure > 18 months)
- Departure penalty: -[X] points (if recent departures exceed threshold)

**Final Score:** [X] / 100

### Score Interpretation

| Score Range | Rating | Description |
|-------------|--------|-------------|
| **90-100** | 🟢 Excellent | Very stable team, minimal turnover |
| **75-89** | 🟡 Good | Stable team with healthy growth |
| **55-74** | 🟠 Fair | Functional but elevated turnover |
| **30-54** | 🔴 Poor | High turnover affecting productivity |
| **0-29** | 🔴 Critical | Very high turnover, major instability |

---

## Key Findings

### Team Stability

<details open>
<summary><b>Churn Analysis</b></summary>

**Stability Assessment:**
- [Summary of team stability based on churn metrics]
- [Analysis of departure patterns]
- [Analysis of onboarding patterns]

**Positive Indicators:**
- ✅ [Positive finding 1]
- ✅ [Positive finding 2]

**Areas of Concern:**
- ⚠️ [Concern 1]
- ⚠️ [Concern 2]

</details>

### Security Accountability

<details open>
<summary><b>Vulnerability Attribution</b></summary>

**Top Contributors to Vulnerabilities (Commits):**

| Developer | Critical | High | Medium | Low | Total |
|-----------|----------|------|--------|-----|-------|
| [Name] | [N] | [N] | [N] | [N] | [N] |
| [Name] | [N] | [N] | [N] | [N] | [N] |
| [Name] | [N] | [N] | [N] | [N] | [N] |

**Top Approvers of Vulnerabilities (Reviews):**

| Developer | Critical | High | Medium | Low | Total |
|-----------|----------|------|--------|-----|-------|
| [Name] | [N] | [N] | [N] | [N] | [N] |
| [Name] | [N] | [N] | [N] | [N] | [N] |
| [Name] | [N] | [N] | [N] | [N] | [N] |

**Patterns:**
- [Analysis of vulnerability patterns]
- [Identification of knowledge gaps or training needs]

</details>

---

## Recommendations

### For Team Stability

**To Reduce Churn:**
- [Recommendation 1]
- [Recommendation 2]

**To Improve Onboarding:**
- [Recommendation 1]
- [Recommendation 2]

**To Mitigate Knowledge Loss:**
- [Recommendation 1]
- [Recommendation 2]

### For Security Awareness

**For Development Team:**
- [Recommendation based on vulnerability attribution patterns]
- [Recommendation for secure coding training]

**For Code Review Process:**
- [Recommendation for improving security in code reviews]
- [Recommendation for review checklists or guidelines]

**For Security Training:**
- [Recommendation for targeted training based on vulnerability types]
- [Recommendation for security champions program]

---

<details>
<summary><b>📋 Methodology</b></summary>

## Assessment Methodology

### Team Stability Analysis

**Data Collection:**
1. Git log analysis for all contributors
2. First and last commit identification for each developer
3. Commit activity analysis over assessment window

**Churn Calculation:**
- **Churn Rate** = (Number of Departures / Average Team Size) × 100
- **Departure Detection:** Developer is considered "departed" if last commit was more than 60-90 days ago

**Team Stability Maturity Scoring:**

| Level | Score | Annual Churn Rate | Avg Tenure | Description |
|-------|-------|-------------------|------------|-------------|
| **5** | Exceptional | 0-10% | 18+ months | Very stable team, minimal turnover |
| **4** | Strong | 11-15% | 12-18 months | Stable team with healthy growth |
| **3** | Proficient | 16-25% | 8-12 months | Functional but elevated turnover |
| **2** | Developing | 26-35% | 5-8 months | High turnover affecting productivity |
| **1** | Critical | 36%+ | <5 months | Very high turnover, major instability |

### Vulnerability Attribution Analysis

**Data Collection:**
1. Gather all vulnerabilities from security audit findings
2. For each vulnerability, use `git blame -L [start_line],[end_line] [file_path] --line-porcelain` to identify the commit
3. Extract commit author (who wrote the code)
4. Identify PR reviewer (who approved the code) from merge commit or GitHub API

**Attribution Criteria:**
- **Committed by:** The developer who wrote and committed the vulnerable code
- **Approved by:** The reviewer who approved the PR containing the vulnerable code

**Limitations:**
- Git blame shows the most recent change to a line, not necessarily the original introduction
- PR reviewers may not be available in git history for squash-merged PRs
- Some vulnerabilities span multiple commits or files
- Automated commits are excluded from attribution

</details>

---

## Detailed Reports

- [Vulnerability Attribution Analysis](vulnerability-attribution.md)
- [Developer Churn Analysis](developer-churn.md)

---

**Assessment Date:** [DATE]
