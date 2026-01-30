# Wallet Tracker - Feature-by-Feature Implementation Guide

## Overview
This document provides a detailed breakdown of each of the 12 GitHub issues, including acceptance criteria, dependencies, implementation tips, and testing guidance.

---

## Issue #1: Implement Core Data Models

**Priority**: CRITICAL  
**Depends On**: None  
**Required For**: All other features

### What This Issue Covers
- Define 4 core database entities
- Implement database schema
- Create ORM models
- Set up database migrations

### Acceptance Criteria
- ✅ All 4 entities (Transaction, IncomeSource, Statement, IgnoreRule) are implemented
- ✅ All relationships are correctly mapped
- ✅ Database schema matches DATA_MODELS.md
- ✅ Indexes are created for performance
- ✅ Foreign key constraints enforced
- ✅ Migrations can be applied and rolled back
- ✅ Sample data can be seeded
- ✅ No migrations pending before starting other features

### Implementation Steps
1. Design database schema (reference DATA_MODELS.md)
2. Create migration files
3. Implement ORM models in backend
4. Add validation rules
5. Create database indexes
6. Write database tests
7. Document schema changes

### Testing Checklist
- [ ] Create transactions with all required fields
- [ ] Create income sources with valid types
- [ ] Create statements with valid date ranges
- [ ] Create ignore rules with valid scope
- [ ] Verify foreign key relationships
- [ ] Verify cascade delete rules
- [ ] Verify unique constraints work
- [ ] Verify check constraints work

### Estimated Effort: 2-3 days

---

## Issue #2: Manual Transaction Entry UI/UX

**Priority**: HIGH  
**Depends On**: Issue #1 (Core Data Models)  
**Required For**: Issues #4, #6, #8, #9

### What This Issue Covers
- Build transaction entry form
- Implement category management
- Create success/error handling
- Add keyboard shortcuts
- Test on mobile

### Acceptance Criteria
- ✅ Form displays all required fields (amount, direction, category, date, note)
- ✅ Amount field accepts decimal values
- ✅ Direction is toggleable (Expense ↔ Income)
- ✅ Category dropdown or autocomplete available
- ✅ Date defaults to today, user can change
- ✅ Source field only shown for income
- ✅ Form validates before saving
- ✅ Success notification after save
- ✅ Clear for next entry (or option to edit)
- ✅ Full audit trail recorded
- ✅ Works on mobile devices
- ✅ Keyboard navigation supported

### Implementation Steps
1. Create form component
2. Add field validation
3. Implement API endpoint for save
4. Add category autocomplete
5. Create success/error notifications
6. Add keyboard shortcuts
7. Test on mobile
8. Add accessibility features

### UI/UX Checklist
- [ ] Form is clean and minimal
- [ ] Fields are clearly labeled
- [ ] Required fields marked
- [ ] Error messages are clear
- [ ] Success feedback is visible
- [ ] Mobile-friendly (touch targets > 48px)
- [ ] Colors are accessible (4.5:1 contrast)
- [ ] Tab order is logical

### Estimated Effort: 3-4 days

---

## Issue #3: Income Source Management UI

**Priority**: HIGH  
**Depends On**: Issue #1 (Core Data Models)  
**Required For**: Issues #5, #9, #12

### What This Issue Covers
- Create income source CRUD interface
- Implement active/inactive toggle
- Display source statistics
- Show transaction history per source

### Acceptance Criteria
- ✅ Add new income source form
- ✅ Edit income source details
- ✅ Toggle active/inactive status
- ✅ View total received per source
- ✅ View source-specific statistics
- ✅ View transactions by source
- ✅ Cannot delete sources (only deactivate)
- ✅ Inactive sources hidden from entry UI
- ✅ Historical data always preserved
- ✅ Confirmation before deactivating

### Implementation Steps
1. Create source list component
2. Build add/edit form
3. Implement active toggle
4. Add statistics view
5. Create transaction filter by source
6. Add confirmation dialogs
7. Test cascading rules

### Feature Checklist
- [ ] Create new source
- [ ] Edit source name and type
- [ ] Deactivate source
- [ ] View source stats
- [ ] View source transactions
- [ ] Inactive sources not in dropdowns
- [ ] History preserved after deactivate
- [ ] Confirmation dialogs work

### Estimated Effort: 2-3 days

---

## Issue #4: Transaction History & Filters

**Priority**: HIGH  
**Depends On**: Issue #1 (Core Data Models), #2 (Manual Entry)  
**Required For**: Issues #6, #8, #9

### What This Issue Covers
- Build chronological transaction list
- Implement filtering engine
- Add search functionality
- Create pagination

### Acceptance Criteria
- ✅ All transactions displayed in chronological order (newest first)
- ✅ Visual differentiation for income (green) vs. expense (red)
- ✅ All transaction details visible (date, description, category, amount, balance)
- ✅ Date range filter works
- ✅ Category filter (multi-select) works
- ✅ Direction filter (Income/Expense) works
- ✅ Free-text search in description works
- ✅ Can combine multiple filters
- ✅ Clear all filters option
- ✅ Pagination (20-50 items per page)
- ✅ Performance acceptable (< 1 second load)

### Implementation Steps
1. Create transactions list component
2. Implement filtering logic
3. Add search functionality
4. Create pagination
5. Add visual indicators (income/expense colors)
6. Optimize queries
7. Add sorting options

### Filter Checklist
- [ ] Date range picker
- [ ] Category dropdown (multi-select)
- [ ] Direction toggle
- [ ] Income source dropdown
- [ ] Search box
- [ ] Clear filters button
- [ ] Active filters displayed
- [ ] Performance under 1 second

### Estimated Effort: 3-4 days

---

## Issue #5: PDF Bank Statement Import: End-to-End Flow

**Priority**: HIGH  
**Depends On**: Issue #1 (Core Data Models)  
**Required For**: Issues #11, #12

### What This Issue Covers
- PDF upload interface
- PDF parsing logic for IDFC format
- Transaction preview
- Duplicate detection
- Ignore rules application
- Final approval and batch save

### Acceptance Criteria
- ✅ Upload PDF with drag-and-drop support
- ✅ Parse IDFC First Bank PDF format correctly
- ✅ Extract date, description, debit, credit, balance
- ✅ Show parsed transactions in preview
- ✅ Detect duplicates against existing transactions
- ✅ Mark duplicate transactions
- ✅ Apply ignore rules
- ✅ Show ignored transactions separately
- ✅ Allow user to restore ignored transactions
- ✅ Show final confirmation with transaction count
- ✅ NO data saved before user confirms
- ✅ Batch save all approved transactions
- ✅ Create Statement record
- ✅ Show success message with count

### Implementation Steps
1. Create upload component
2. Implement PDF parsing (use pdf-parse or similar)
3. Build preview table
4. Implement duplicate detection algorithm
5. Integrate ignore rules
6. Create approval flow
7. Implement batch save
8. Add error handling

### Parsing Checklist
- [ ] Reads IDFC PDF format
- [ ] Extracts transaction date
- [ ] Extracts description/reference
- [ ] Extracts debit amount
- [ ] Extracts credit amount
- [ ] Extracts running balance
- [ ] Handles multi-line descriptions
- [ ] Validates date format
- [ ] Validates amount format

### Flow Checklist
- [ ] Upload works
- [ ] Preview shows correctly
- [ ] Duplicates detected
- [ ] Ignore rules applied
- [ ] Ignored transactions shown
- [ ] Can restore ignored transactions
- [ ] Confirmation dialog shows
- [ ] Save only on confirmation
- [ ] Success message shows count

### Estimated Effort: 5-7 days

---

## Issue #6: Home Dashboard Implementation

**Priority**: HIGH  
**Depends On**: Issues #1 (Models), #2 (Entries), #3 (Income Sources)  
**Required For**: None (but good foundation)

### What This Issue Covers
- Current balance display
- Monthly summary cards
- Income sources summary
- Non-financial reminders
- Quick action buttons

### Acceptance Criteria
- ✅ Current balance clearly displayed
- ✅ Last updated time shown
- ✅ This month income total
- ✅ This month expense total
- ✅ Net flow (positive/negative)
- ✅ Color coding: green (positive), red (negative)
- ✅ Income sources summary with this month amounts
- ✅ Quick action buttons (Add, Import, Analytics)
- ✅ Non-financial reminders section
- ✅ Mobile responsive
- ✅ Accessible colors (4.5:1 contrast)
- ✅ No unnecessary decoration
- ✅ Numbers-focused design

### Design Checklist
- [ ] Minimal, clean layout
- [ ] Neutral color palette
- [ ] Good use of whitespace
- [ ] Clear typography hierarchy
- [ ] Mobile-first responsive
- [ ] Touch targets > 48px
- [ ] No animations/distractions
- [ ] Calm aesthetic

### Estimated Effort: 2-3 days

---

## Issue #7: Export & Report Generation

**Priority**: MEDIUM  
**Depends On**: Issues #1, #8, #9, #10  
**Required For**: None

### What This Issue Covers
- Report generation engine
- PDF export with charts
- CSV export
- Excel export with sheets
- Date range selection

### Acceptance Criteria
- ✅ Monthly report generation
- ✅ Yearly summary generation
- ✅ Category breakdown report
- ✅ Income source summary report
- ✅ PDF format with charts
- ✅ CSV format (raw data)
- ✅ Excel format (multiple sheets)
- ✅ Custom date range selector
- ✅ Only user-logged data exported
- ✅ No credentials exported
- ✅ No forecasting data
- ✅ Local generation (no cloud)
- ✅ Audit log records exports
- ✅ Export < 5 seconds

### Implementation Steps
1. Design report templates
2. Implement PDF generation (jsPDF/PDFKit)
3. Implement CSV export
4. Implement Excel export
5. Add date range selectors
6. Create export preview
7. Implement audit logging
8. Test all formats

### Export Checklist
- [ ] PDF generates correctly
- [ ] PDF includes charts
- [ ] PDF is readable on all devices
- [ ] CSV format is correct
- [ ] Excel has multiple sheets
- [ ] All data is accurate
- [ ] No sensitive data exported
- [ ] File names are descriptive
- [ ] Download works

### Estimated Effort: 4-5 days

---

## Issue #8: Monthly Analysis: Charts and Metrics

**Priority**: HIGH  
**Depends On**: Issues #1 (Models), #2 (Entries), #4 (History)  
**Required For**: Issue #12 (Reports)

### What This Issue Covers
- Monthly summary cards
- Daily spend bar chart
- Month-on-month comparison
- Previous/next month navigation

### Acceptance Criteria
- ✅ Monthly totals (income, expense, net) displayed correctly
- ✅ Totals update when new transactions added
- ✅ Daily spend bar chart shows correctly
- ✅ Month-on-month comparison shows 12 months
- ✅ Previous/next month navigation works
- ✅ Charts update dynamically
- ✅ Month selector dropdown works
- ✅ Tooltip shows exact values
- ✅ No animations or unnecessary effects
- ✅ Clean, minimal design
- ✅ Mobile responsive

### Chart Checklist
- [ ] Daily spend chart renders
- [ ] Bar heights are accurate
- [ ] X-axis labels correct
- [ ] Y-axis scale appropriate
- [ ] Tooltip shows exact values
- [ ] Previous/next buttons work
- [ ] Month selector works
- [ ] Colors are accessible
- [ ] Charts responsive to screen size

### Estimated Effort: 3-4 days

---

## Issue #9: Category Analytics: Trends & Distribution

**Priority**: HIGH  
**Depends On**: Issues #1 (Models), #2 (Entries), #4 (History)  
**Required For**: Issue #12 (Reports)

### What This Issue Covers
- Category pie chart
- Category totals table
- Category trends over time
- Category comparison

### Acceptance Criteria
- ✅ Pie chart shows all categories
- ✅ Percentages calculated correctly
- ✅ Category totals table displays all categories
- ✅ Sortable by amount or name
- ✅ Trends chart shows last 12 months
- ✅ Each category has own line
- ✅ Can select/deselect categories
- ✅ Comparison view works
- ✅ Date range filter works
- ✅ Colors are neutral/accessible
- ✅ No gamification elements
- ✅ No pressure colors

### Chart Checklist
- [ ] Pie chart accuracy
- [ ] Category labels visible
- [ ] Legend displays
- [ ] Trends chart accuracy
- [ ] Line colors distinguishable
- [ ] Tooltip shows values
- [ ] Category toggle works
- [ ] Date filters work
- [ ] Responsive design

### Estimated Effort: 3-4 days

---

## Issue #10: Income Analytics: Source Tracking & Insights

**Priority**: HIGH  
**Depends On**: Issues #1 (Models), #3 (Income Sources), #2 (Entries)  
**Required For**: Issue #12 (Reports)

### What This Issue Covers
- Income per source display
- Monthly breakdown by source
- Stability classification
- Derived insights calculation

### Acceptance Criteria
- ✅ Income per source (current month) correct
- ✅ Monthly breakdown shows 12 months
- ✅ Stability classification accurate
- ✅ Stability badge displayed
- ✅ Most stable source calculated correctly
- ✅ Most volatile source calculated correctly
- ✅ Best income month identified correctly
- ✅ Zero-income months identified
- ✅ Average income per source calculated
- ✅ Trend lines rendered correctly
- ✅ Insights update when data changes
- ✅ Charts update dynamically

### Stability Logic
- **Stable**: Standard deviation < 15% of average
- **Volatile**: Standard deviation >= 15% of average

### Calculation Checklist
- [ ] Income totals correct
- [ ] Percentages accurate
- [ ] Stability classification correct
- [ ] Trend calculations accurate
- [ ] Best/worst month identified
- [ ] Average calculated correctly
- [ ] Updates on new transaction

### Estimated Effort: 3-4 days

---

## Issue #11: Ignore Rule Management: Full Control UI

**Priority**: MEDIUM  
**Depends On**: Issue #1 (Core Data Models), #5 (PDF Import)  
**Required For**: None (enhances Issue #5)

### What This Issue Covers
- Create ignore rules
- Edit existing rules
- Enable/disable rules
- Delete rules
- Restore ignored transactions
- Rule audit trail

### Acceptance Criteria
- ✅ Create new rule form works
- ✅ Match text input validated
- ✅ Scope selector (This/Future) works
- ✅ View all rules list
- ✅ Edit existing rules
- ✅ Enable/disable toggle works (no deletion)
- ✅ Delete rules with confirmation
- ✅ View ignored transactions list
- ✅ Restore ignored transactions
- ✅ Audit trail of rule changes
- ✅ Rule count shown per rule
- ✅ NO auto-created rules
- ✅ All rules visible to user

### Rule Management Checklist
- [ ] Create rule form displays
- [ ] Scope selector shows options
- [ ] Rules saved correctly
- [ ] Rules list displays all
- [ ] Edit form pre-fills data
- [ ] Enable/disable toggle works
- [ ] Delete requires confirmation
- [ ] Ignore list shows transactions
- [ ] Restore works
- [ ] Audit trail tracks changes

### Estimated Effort: 3-4 days

---

## Issue #12: Statement Management: Import & Delete Flow

**Priority**: MEDIUM  
**Depends On**: Issue #1 (Models), #5 (PDF Import)  
**Required For**: None

### What This Issue Covers
- View imported statements list
- View statement details
- Delete statements safely
- Prevent accidental deletion
- Audit trail of deletions

### Acceptance Criteria
- ✅ All statements displayed in list
- ✅ Shows bank, period, count, date for each
- ✅ Click statement to view details
- ✅ Details show all transactions from statement
- ✅ Delete button available
- ✅ Confirmation dialog required
- ✅ Dialog shows impact (transaction count)
- ✅ Dialog clearly states other data won't be deleted
- ✅ Confirmation and cancel buttons
- ✅ After delete: transactions removed, statement deleted
- ✅ Manual entries never deleted
- ✅ Other imports unaffected
- ✅ Audit log entry created

### Statement Management Checklist
- [ ] List displays all statements
- [ ] Bank name shown
- [ ] Period shows correctly
- [ ] Transaction count accurate
- [ ] Click to view details works
- [ ] Details show transactions
- [ ] Delete button visible
- [ ] Confirmation dialog shows
- [ ] Dialog info is clear
- [ ] Delete works correctly
- [ ] Success message shows

### Estimated Effort: 2-3 days

---

## Implementation Order & Dependencies

### Phase 1: Foundation (Week 1)
1. **Issue #1**: Core Data Models
   - All other issues depend on this
   - Must complete before any other work

### Phase 2: Core Features (Weeks 2-3)
2. **Issue #2**: Manual Transaction Entry (depends: #1)
3. **Issue #3**: Income Source Management (depends: #1)
4. **Issue #4**: Transaction History & Filters (depends: #1, #2)
5. **Issue #5**: PDF Bank Statement Import (depends: #1)

### Phase 3: Dashboard (Week 4)
6. **Issue #6**: Home Dashboard (depends: #1, #2, #3)

### Phase 4: Analytics (Week 5)
7. **Issue #8**: Monthly Analysis (depends: #1, #2, #4)
8. **Issue #9**: Category Analytics (depends: #1, #2, #4)
9. **Issue #10**: Income Analytics (depends: #1, #3, #2)

### Phase 5: Management & Reports (Week 6)
10. **Issue #11**: Ignore Rule Management (depends: #1, #5)
11. **Issue #12**: Statement Management (depends: #1, #5)
12. **Issue #7**: Export & Reports (depends: #1, #8, #9, #10)

---

## Testing Strategy

### Unit Tests
- Data model validation
- Calculation logic (stability, totals, averages)
- Filter logic
- Ignore rule matching

### Integration Tests
- API endpoints
- Database operations
- File uploads
- Report generation

### E2E Tests
- Complete import flow
- Complete transaction entry flow
- Analytics calculations
- Report export

### Browser/Device Tests
- Chrome, Firefox, Safari, Edge (latest 2 versions)
- iOS Safari (12+), Android Chrome (8+)
- Tablet and mobile responsiveness
- Touch interactions

---

## Success Metrics

- ✅ All acceptance criteria met for each issue
- ✅ Code coverage > 80%
- ✅ Performance: pages load < 2 seconds
- ✅ Charts render < 1 second
- ✅ No console errors
- ✅ Accessibility: WCAG 2.1 Level AA
- ✅ Mobile responsive
- ✅ All filters work correctly
- ✅ No data loss
- ✅ Audit trails complete

---
