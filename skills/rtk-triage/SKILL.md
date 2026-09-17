---
name: rtk-triage
description: >
  Triage orchestrator for RTK repo: runs issue-triage and pr-triage,
  cross-analyzes data to detect double coverage, security holes, P0 issues without PRs,
  and dirty PRs.
---

# RTK Triage Orchestrator

Orchestrator for RTK issue & PR triage with cross-analysis.

## When to Use

- Weekly or before sprint planning
- When PR/issue backlog grows
- To identify duplicate efforts before reviewing

## 4-Phase Workflow

### Phase 1 — Data Gathering
```bash
rtk gh issue list --state open --limit 150
rtk gh pr list --state open --limit 200
```

### Phase 2 — Individual Triage
Categorize issues (Bug/Feature/Duplicate) and PRs (XS/S/M/L/XL, CI status).

### Phase 3 — Cross Analysis
- Detect double coverage (2 PRs for 1 issue)
- Identify unhandled security findings
- Find P0/P1 bugs lacking PRs
- Analyze dirty PR root causes

### Phase 4 — Report Generation
Output summary table and save report to `claudedocs/RTK-YYYY-MM-DD.md`.
