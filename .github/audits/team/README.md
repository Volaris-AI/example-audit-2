# Team Assessment Templates

## Overview

This directory contains templates for team assessment focused on **developer churn** and **security vulnerability attribution**. The team health score is entirely based on team stability (churn).

## Assessment Period

**Standard Assessment Window**: Last 2 months of commits

This timeframe provides enough data to identify patterns while remaining relevant to current team dynamics.

## Assessment Templates

### 1. Vulnerability Attribution
- **[vulnerability-attribution.md](vulnerability-attribution.md)** - Links security vulnerabilities to the developers who committed and approved them

### 2. Developer Churn
- **[developer-churn.md](developer-churn.md)** - Analyzes team stability through developer churn metrics (first and last commits)

### 3. Executive Summary
- **[executive-summary.md](executive-summary.md)** - Team assessment executive summary with health score based on churn

## Assessment Process

### Phase 1: Data Collection

1. **Extract Git History**
   ```bash
   # Get commits from assessment window
   git log --since="2 months ago" --all --pretty=format:"%H|%an|%ae|%ad|%s" --date=iso
   
   # Get first and last commits for each developer
   git log --all --author="[email]" --format='%ad' --date=short
   
   # Count commits per developer
   git shortlog -sn --since="2 months ago" --all
   ```

2. **Gather Security Vulnerabilities**
   - Read security audit findings from `audits/[date]/security/`
   - Extract file paths and line numbers for each vulnerability
   - Use `git blame` to attribute vulnerabilities to developers

3. **Identify Team Members**
   - List all active contributors in the assessment window
   - Note any team changes (new hires, departures)

### Phase 2: Analysis

1. **Vulnerability Attribution** (vulnerability-attribution.md)
   - For each security vulnerability, use `git blame -L [line],[line] [file] --line-porcelain`
   - Identify the commit author (who wrote the vulnerable code)
   - Identify the PR reviewer (who approved the vulnerable code)
   - Create attribution tables showing vulnerabilities per developer

2. **Developer Churn Analysis** (developer-churn.md)
   - Calculate churn rate based on departures and team size
   - Analyze first and last commits for each developer
   - Identify team stability patterns
   - Calculate Team Stability Maturity score (1-5)

### Phase 3: Synthesis

1. **Executive Summary** (executive-summary.md)
   - Aggregate churn metrics
   - Summarize vulnerability attribution
   - Calculate team health score (entirely based on churn)
   - Provide recommendations

## Team Health Scoring

**Team Health Score is entirely based on team stability (churn):**

**Base Score:** Team Stability Maturity × 20

**Adjustments:**
- Tenure bonus: +5 points (if average tenure > 18 months)
- Departure penalty: -10 points (if recent departures exceed threshold)

### Team Stability Maturity Rubric (1-5)

| Level | Annual Churn Rate | Avg Tenure | Description |
|-------|-------------------|------------|-------------|
| **5** | 0-10% | 18+ months | Very stable team, minimal turnover |
| **4** | 11-15% | 12-18 months | Stable team with healthy growth |
| **3** | 16-25% | 8-12 months | Functional but elevated turnover |
| **2** | 26-35% | 5-8 months | High turnover affecting productivity |
| **1** | 36%+ | <5 months | Very high turnover, major instability |

## Key Principles

### 🎯 Evidence-Based
- Every assessment must cite specific commits or vulnerabilities
- Include commit SHAs, file paths, and line numbers for reference
- Use git blame for accurate attribution

### 📊 Objective Metrics
- Use quantifiable data wherever possible (churn rate, tenure, vulnerability counts)
- Track trends over time (improving vs declining)
- Link vulnerabilities to specific developers for accountability

### 🔒 Constructive
- Focus on patterns and learning opportunities, not punishment
- Use vulnerability attribution to identify training needs
- Frame findings as opportunities for improvement

## Analysis Tools

### Git Commands
```bash
# List all contributors
git log --all --format='%aN|%aE' | sort -u

# Get first commit for a developer
git log --all --author="[email]" --format='%ad' --date=short --reverse | head -1

# Get last commit for a developer
git log --all --author="[email]" --format='%ad' --date=short | head -1

# Count commits in assessment window
git shortlog -sn --since="2 months ago" --until="now" --all

# Git blame for vulnerability attribution
git blame -L [start_line],[end_line] [file_path] --line-porcelain
```

### Common Analysis Patterns

**Identify Departed Developers:**
```bash
# Find developers with no commits in last 60 days
git log --all --since="60 days ago" --format='%aE' | sort -u > recent.txt
git log --all --format='%aE' | sort -u > all.txt
comm -13 recent.txt all.txt
```

**Calculate Churn Rate:**
- Churn Rate = (Departures / Average Team Size) × 100
- Annual Projected Churn = (Assessment Period Churn / Days in Period) × 365

## Common Pitfalls to Avoid

❌ **Don't:**
- Use this data to punish developers for vulnerabilities
- Focus only on individual blame without addressing systemic issues
- Make assessments without sufficient data
- Ignore context (complexity, time pressure, knowledge gaps)

✅ **Do:**
- Use vulnerability attribution to identify training needs
- Look for patterns across multiple developers
- Consider the review process (why did vulnerabilities get approved?)
- Focus on improving the overall security culture
- Celebrate improvements in churn and stability

## Assessment Timeline

The team audit focuses on two main areas and can be completed efficiently:

```
Phase 1: Data Collection (1-2 days)
├── Extract git history (all contributors, dates)
├── Gather security audit findings
└── Prepare for analysis

Phase 2: Analysis (2-3 days)
├── Vulnerability attribution (git blame for each vulnerability)
├── Developer churn analysis (first/last commits, tenure)
└── Calculate metrics and scores

Phase 3: Synthesis (1-2 days)
├── Complete executive summary
├── Generate recommendations
└── Finalize report
```

## Output Directory Structure

Team audit outputs are organized as:
```
audits/YYYY-MM-DD/team/
├── vulnerability-attribution.md
├── developer-churn.md
└── executive-summary.md
```

## Version History

- **v2.0** (2026-02): Refocused on vulnerability attribution and developer churn only
- **v1.0** (2024-01): Initial comprehensive team assessment templates
