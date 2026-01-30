# Wallet Tracker - GitHub Project Board Setup & Organization

## Overview
This document describes how to set up and manage the GitHub Project Board for the Wallet Tracker application, including columns, issue organization, and workflow.

---

## Project Board Structure

### Board Name
**Wallet Tracker - Development Pipeline**

### Board Type
**Table Board** (recommended for better organization and filtering)

---

## Columns & Workflow

### Column 1: 📋 Backlog
**Purpose**: Issues that are identified but not yet prioritized or assigned.

**Status**: Not started  
**Issues that belong here**:
- New feature requests
- Bug reports
- Enhancement ideas
- Issues awaiting clarification

**Auto-triggers**:
- New issues automatically added to this column

**Move to next when**:
- Issue is refined and ready for prioritization
- Acceptance criteria are clear
- Dependencies are identified

---

### Column 2: 🎯 Prioritized (Ready to Start)
**Purpose**: Clearly defined issues ready for development, with dependencies met.

**Status**: Queued for work  
**Issues that belong here**:
- Issues #1 (Core Models) - START HERE
- Issues with clear acceptance criteria
- Issues with dependencies completed
- Issues assigned to team members

**Criteria for issues in this column**:
- ✅ Acceptance criteria clearly defined
- ✅ Dependencies identified and available
- ✅ Estimated effort assigned
- ✅ No blockers
- ✅ Ready for immediate start

**Move to next when**:
- Developer starts working on the issue
- Work begins on the code

---

### Column 3: 🚀 In Progress
**Purpose**: Issues currently being worked on.

**Status**: Active development  
**Issues that belong here**:
- Issues being actively developed
- Issues in code review
- Issues with open PRs

**Rules**:
- Limit WIP (Work in Progress) to 3-4 concurrent issues
- Only move here when actively developing
- Move back to Prioritized if work is blocked

**Move to next when**:
- Pull Request is submitted
- Code review is requested
- All acceptance criteria are met and tested

---

### Column 4: 👀 In Review
**Purpose**: Code is complete, awaiting review.

**Status**: Waiting for approval  
**Issues that belong here**:
- Issues with submitted PRs
- Issues awaiting code review
- Issues with requested changes

**Associated PR**:
- Link PR in issue comments
- Track review progress

**Move to next when**:
- PR is approved
- No requested changes remain
- CI/CD checks pass

---

### Column 5: ✅ Testing
**Purpose**: Code is approved, now being tested.

**Status**: Quality assurance  
**Issues that belong here**:
- Issues with merged PRs
- Issues in QA/testing phase
- Issues being tested for bugs

**Testing includes**:
- Manual testing
- Automated test execution
- Cross-browser testing
- Mobile testing
- Accessibility testing

**Move to next when**:
- All tests pass
- No bugs found
- QA sign-off obtained

---

### Column 6: 🎉 Done
**Purpose**: Completed and deployed features.

**Status**: Finished  
**Issues that belong here**:
- Merged and tested issues
- Issues deployed to production
- Issues verified working

**Move here when**:
- All acceptance criteria met
- Code merged to main branch
- Feature deployed
- Verified working in production

---

## Issue Organization & Prioritization

### Priority Levels (Use Labels)

**🔴 Critical** (P0)
- Must complete before other features
- Blocks other issues
- Core to application
- Issues: #1

**🟠 High** (P1)
- Required for MVP
- Major features
- Should complete early
- Issues: #2, #3, #4, #5, #6, #8, #9, #10

**🟡 Medium** (P2)
- Enhances MVP
- Nice-to-have features
- Can delay if needed
- Issues: #7, #11, #12

**🟢 Low** (P3)
- Optional features
- Polish and refinement
- Can schedule for later

---

## Labels System

### Priority Labels
- `priority: critical` - P0, blocks everything
- `priority: high` - P1, MVP required
- `priority: medium` - P2, important
- `priority: low` - P3, nice-to-have

### Category Labels
- `feature: data-models` - Data layer
- `feature: ui-ux` - User interface
- `feature: analytics` - Analytics features
- `feature: import` - Import functionality
- `feature: export` - Export functionality
- `feature: management` - Data management

### Status Labels
- `status: ready` - Ready to work on
- `status: blocked` - Waiting for blocker
- `status: in-progress` - Currently being worked
- `status: review` - In code review
- `status: testing` - In QA/testing

### Type Labels
- `type: feature` - New feature
- `type: bug` - Bug fix
- `type: enhancement` - Enhancement
- `type: documentation` - Documentation
- `type: refactor` - Code refactor

### Effort Labels
- `effort: small` - 1-2 days
- `effort: medium` - 3-5 days
- `effort: large` - 1-2 weeks

### Phase Labels
- `phase: 1-foundation` - Phase 1
- `phase: 2-core` - Phase 2
- `phase: 3-dashboard` - Phase 3
- `phase: 4-analytics` - Phase 4
- `phase: 5-management` - Phase 5

---

## Implementation Phases on Board

### Phase 1: Foundation (Week 1)
```
┌─────────────────────────────────────────┐
│ Issue #1: Core Data Models              │
│ Priority: 🔴 Critical                   │
│ Labels: foundation, feature: data-models│
│ Dependencies: None                       │
│ Effort: 2-3 days                        │
└─────────────────────────────────────────┘
```

**All other issues wait for Phase 1 to complete**

### Phase 2: Core Features (Weeks 2-3)
```
┌─────────────────────────────────────────┐
│ Issue #2: Manual Transaction Entry      │
│ Depends On: #1                          │
│ Priority: 🟠 High                       │
│ Status: Ready after #1 done             │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #3: Income Source Management      │
│ Depends On: #1                          │
│ Priority: 🟠 High                       │
│ Status: Can start with #2               │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #4: Transaction History & Filters │
│ Depends On: #1, #2                      │
│ Priority: 🟠 High                       │
│ Status: Can start after #2 done         │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #5: PDF Bank Statement Import     │
│ Depends On: #1                          │
│ Priority: 🟠 High                       │
│ Status: Can start in parallel with #2   │
└─────────────────────────────────────────┘
```

### Phase 3: Dashboard (Week 4)
```
┌─────────────────────────────────────────┐
│ Issue #6: Home Dashboard Implementation │
│ Depends On: #1, #2, #3                  │
│ Priority: 🟠 High                       │
│ Status: Start after #2, #3 done         │
└─────────────────────────────────────────┘
```

### Phase 4: Analytics (Week 5)
```
┌─────────────────────────────────────────┐
│ Issue #8: Monthly Analysis              │
│ Depends On: #1, #2, #4                  │
│ Priority: 🟠 High                       │
│ Status: Start after #4 done             │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #9: Category Analytics            │
│ Depends On: #1, #2, #4                  │
│ Priority: 🟠 High                       │
│ Status: Can parallel with #8            │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #10: Income Analytics             │
│ Depends On: #1, #3, #2                  │
│ Priority: 🟠 High                       │
│ Status: Can parallel with #8, #9        │
└─────────────────────────────────────────┘
```

### Phase 5: Management & Reports (Week 6)
```
┌─────────────────────────────────────────┐
│ Issue #11: Ignore Rule Management       │
│ Depends On: #1, #5                      │
│ Priority: 🟡 Medium                     │
│ Status: Start after #5 done             │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #12: Statement Management         │
│ Depends On: #1, #5                      │
│ Priority: 🟡 Medium                     │
│ Status: Start after #5 done             │
└─────────────────────────────────────────┘

┌─────────────────────────────────────────┐
│ Issue #7: Export & Report Generation    │
│ Depends On: #1, #8, #9, #10             │
│ Priority: 🟡 Medium                     │
│ Status: Last feature to implement       │
└─────────────────────────────────────────┘
```

---

## Dependency Map

```
Issue #1: Core Data Models (Foundation)
    ├─→ Issue #2: Manual Entry
    │   ├─→ Issue #4: Transaction History
    │   │   ├─→ Issue #8: Monthly Analysis
    │   │   └─→ Issue #9: Category Analytics
    │   ├─→ Issue #6: Dashboard
    │   └─→ Issue #10: Income Analytics
    │
    ├─→ Issue #3: Income Source Management
    │   └─→ Issue #10: Income Analytics
    │
    ├─→ Issue #5: PDF Import
    │   ├─→ Issue #11: Ignore Rules
    │   └─→ Issue #12: Statement Management
    │
    └─→ Issues #7, #8, #9, #10: (Multiple dependencies)
```

---

## Tracking Progress

### Weekly Review Meeting
**Schedule**: Every Friday 4:00 PM

**Agenda**:
1. Review issues moved to Done this week
2. Check blockers in In Progress
3. Plan next week's priorities
4. Discuss any major risks

**Metrics to review**:
- Velocity (issues completed per week)
- Lead time (issue creation to completion)
- Cycle time (start to completion)
- Blocked issues count

### Dashboard View
Pin the following columns on your board:

```
Prioritized (Ready) → In Progress → In Review → Testing → Done
```

This shows the flow at a glance.

---

## GitHub Project Board Automation

### Auto-add to Board
- Create rule: All issues → add to Backlog column

### Auto-move on Actions
- When PR created → move to In Review
- When PR merged → move to Testing
- When issue closed → move to Done

### Filters & Views

**View: "This Week"**
- Filter by: Updated in last 7 days
- Shows current activity

**View: "Phase 1 - Foundation"**
- Filter by: `phase: 1-foundation`
- Shows only Phase 1 issues

**View: "High Priority"**
- Filter by: `priority: critical`, `priority: high`
- Shows critical path

**View: "Blocked"**
- Filter by: `status: blocked`
- Shows issues waiting for resolution

---

## Team Assignments

### Recommended Team Structure (for 3 developers)

**Developer 1: Backend/Data Layer**
- Focus: Issues #1, #5, #12, #7
- Skills: Database, PDF parsing, API endpoints

**Developer 2: Frontend/UI**
- Focus: Issues #2, #3, #4, #6, #11
- Skills: React/Vue, UI/UX, forms, validation

**Developer 3: Analytics**
- Focus: Issues #8, #9, #10, #6 (dashboard)
- Skills: Data visualization, charts, analytics logic

---

## Sprint Planning (Optional)

### Sprint 1 (Week 1)
- Issue #1: Core Data Models

### Sprint 2 (Week 2)
- Issue #2: Manual Transaction Entry
- Issue #3: Income Source Management

### Sprint 3 (Week 3)
- Issue #4: Transaction History & Filters
- Issue #5: PDF Bank Statement Import

### Sprint 4 (Week 4)
- Issue #6: Home Dashboard Implementation

### Sprint 5 (Week 5)
- Issue #8: Monthly Analysis
- Issue #9: Category Analytics
- Issue #10: Income Analytics

### Sprint 6 (Week 6)
- Issue #11: Ignore Rule Management
- Issue #12: Statement Management
- Issue #7: Export & Report Generation

---

## Definition of Done (DoD)

An issue is "Done" when:

- ✅ All acceptance criteria met
- ✅ Code written and tested
- ✅ Unit tests written (> 80% coverage)
- ✅ Integration tests passing
- ✅ Code review approved
- ✅ No console errors
- ✅ Mobile responsive
- ✅ Accessibility checked (WCAG 2.1 AA)
- ✅ Documentation updated
- ✅ Merged to main branch
- ✅ Tested in QA environment
- ✅ Verified working in production

---

## How to Use This on GitHub

1. **Create Project Board**:
   - Go to: https://github.com/Akilucky-rogue/wallet-tracker/projects
   - Click "New project"
   - Select "Table" template
   - Name: "Wallet Tracker - Development Pipeline"

2. **Create Columns**:
   - Add columns: Backlog → Prioritized → In Progress → In Review → Testing → Done

3. **Add Issues**:
   - Go to each issue
   - Add to project board
   - Set custom field: Priority, Phase, Effort

4. **Configure Automation**:
   - Project settings → Automation
   - Set up auto-move rules

5. **Create Filters**:
   - Save views for different perspectives
   - "High Priority", "This Week", "Phase X", etc.

6. **Share with Team**:
   - Add team members
   - Share board link in README

---
