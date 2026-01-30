# Wallet Tracker - Complete Feature Documentation

## Overview
The Wallet Tracker is a comprehensive personal finance management application designed with a privacy-first approach. All data is stored locally, and users maintain complete control over their financial information.

---

## Core Philosophy & Principles

### 1. **User Control & Transparency**
- Users make all decisions; no auto-operations
- Full visibility of all actions
- Complete audit trails for every operation
- Users can always see why something happened

### 2. **Data Privacy**
- All data stored locally on user's device
- No cloud sync or external transmission
- No credentials or sensitive data exported
- No automatic data collection

### 3. **Reversibility & Safety**
- All operations are reversible
- Confirmation required for destructive actions
- Deleted data can be restored from imports
- No permanent loss without explicit user confirmation

### 4. **Historical Accuracy**
- Only actual, verified transactions are analyzed
- No predictions or forecasting
- No estimated values
- Insights based purely on historical data

### 5. **Simplicity & Clarity**
- Minimal UI design
- Focus on numbers over decoration
- Clear, direct language
- No hidden operations

---

## Feature Specifications

### Feature 1: Core Data Models

**Purpose:** Establish foundational data entities for the entire application.

**Entities:**

#### Transaction
```
- id (UUID, primary key)
- amount (decimal, non-null)
- direction (enum: INCOME | EXPENSE, non-null)
- category (string, non-null)
- description (string, nullable)
- date (datetime, non-null)
- source (string, nullable) - for income only
- statement_id (UUID, foreign key, nullable) - links to Statement
- created_at (datetime, non-null)
- updated_at (datetime, non-null)
- is_deleted (boolean, default: false)
```

#### IncomeSource
```
- id (UUID, primary key)
- name (string, non-null, unique)
- type (enum: SALARY | FREELANCE | INVESTMENT | BONUS | OTHER, non-null)
- is_active (boolean, default: true)
- total_received (decimal, default: 0)
- created_at (datetime, non-null)
- updated_at (datetime, non-null)
```

#### Statement
```
- id (UUID, primary key)
- bank_name (string, non-null)
- period_start (date, non-null)
- period_end (date, non-null)
- transaction_count (integer, non-null)
- total_debit (decimal, non-null)
- total_credit (decimal, non-null)
- file_name (string, nullable)
- imported_at (datetime, non-null)
- created_at (datetime, non-null)
```

#### IgnoreRule
```
- id (UUID, primary key)
- match_text (string, non-null)
- scope (enum: THIS_IMPORT_ONLY | FUTURE_IMPORTS, non-null)
- is_enabled (boolean, default: true)
- created_at (datetime, non-null)
- statement_id (UUID, foreign key, nullable) - for THIS_IMPORT_ONLY scope
```

**Requirements:**
- Immutable data where applicable
- Proper indexing for performance
- Cascade delete rules defined
- Referential integrity enforced

---

### Feature 2: Manual Transaction Entry UI/UX

**Purpose:** Allow users to quickly add transactions without bank statement imports.

**Input Fields:**
- **Amount** (required): Numeric, positive value
- **Direction** (required): Expense or Income toggle
- **Category** (required): Predefined list or custom entry
- **Source** (conditional): Only shown for income transactions
- **Date** (required): Defaults to today, user can change
- **Note** (optional): Additional details

**Behavior:**
- No default values except date = today
- Validation on all required fields
- Immediate save to data store
- Full audit trail recorded
- Success notification displayed
- Clear for next entry or option to edit

**Design:**
- Simple, single-screen form
- Mobile-friendly
- Clear call-to-action
- Field labels visible and clear

---

### Feature 3: Income Source Management UI

**Purpose:** Enable users to manage multiple income sources and track by source.

**Operations:**
- **Create**: Add new income source with name and type
- **Read**: View all income sources (active and inactive)
- **Update**: Edit name, type, toggle active/inactive
- **View Stats**: Total received, monthly breakdown, trends

**Constraints:**
- Income source names must be unique
- Cannot delete; must deactivate
- Inactive sources hidden from entry UI but visible in history
- Historical data always preserved

**Display:**
- List of all sources (sorted by total received)
- For each source:
  - Name
  - Type badge
  - Active/Inactive toggle
  - Total received (lifetime)
  - Last received date
  - Stats button

---

### Feature 4: PDF Bank Statement Import: End-to-End Flow

**Purpose:** Automated import of bank statements with user control at every step.

**Supported Format:**
- IDFC First Bank PDF statements

**Parsing Logic:**
- Extracts: Date, Description, Debit, Credit, Running Balance
- Validates each row
- Identifies transaction type (debit/credit)
- Handles multi-line descriptions

**Workflow:**

1. **Upload**
   - File input with drag-and-drop support
   - File validation (PDF format)
   - Show file name and size

2. **Parse**
   - Extract transactions from PDF
   - Validate date format and amounts
   - Calculate balances
   - No data saved yet

3. **Preview**
   - Show all extracted transactions
   - Display summary (count, total debit, total credit)
   - Allow user to review each row
   - Show parsing errors (if any)

4. **Duplicate Detection**
   - Compare against existing transactions
   - Flag potential duplicates
   - Allow user to mark as duplicate or accept

5. **Apply Ignore Rules**
   - Check against all active ignore rules
   - Show matched transactions
   - Separate ignored from approved transactions
   - Allow user to restore any ignored transactions

6. **User Approval**
   - Final confirmation before import
   - Show:
     - Number of transactions to be imported
     - Transactions being ignored
     - Affected date range
   - Explicit "Confirm Import" button
   - Cancel option available

7. **Batch Save**
   - Save all approved transactions
   - Create Statement record
   - Log import in audit trail
   - Show success message with count

**Requirements:**
- Do NOT save any data before user confirms
- No auto-ignores; all ignores are explicit user choices
- All actions fully auditable
- User can go back and restore ignored transactions later

---

### Feature 5: Home Dashboard Implementation

**Purpose:** Provide a quick overview of financial status at a glance.

**Sections:**

1. **Current Balance**
   - Displays user's current balance
   - Shows last updated time
   - Option to refresh

2. **This Month Overview**
   - Total Income (current month)
   - Total Expenses (current month)
   - Net Flow (Income - Expenses)
   - Color coding: Green (positive), Red (negative)

3. **Income Sources Summary**
   - List of active income sources
   - Amount received this month (per source)
   - Total from all sources
   - Only shows sources with transactions

4. **Non-Financial Reminders**
   - Optional reminders (not affecting balance)
   - User-configurable
   - Examples: Update password, Review spending, etc.

5. **Quick Actions**
   - Add Transaction button
   - Import Statement button
   - View Analytics button

**Design Guidelines:**
- Minimal, clean layout
- Neutral color palette
- Focus on numbers
- No unnecessary decoration
- Calm, professional appearance
- Responsive mobile design

---

### Feature 6: Transaction History & Filters

**Purpose:** Provide chronological view of all transactions with powerful filtering.

**Display:**
- Chronological list (newest first)
- Alternating row colors for readability
- Visual differentiation:
  - Income: Green badge or + symbol
  - Expense: Red badge or - symbol

**Columns Displayed:**
- Date
- Description
- Category
- Amount (with direction symbol)
- Balance (running balance after transaction)
- Source (if income)

**Filters:**
1. **Date Range**: From date - To date picker
2. **Category**: Multi-select dropdown
3. **Direction**: Income / Expense toggle
4. **Income Source**: Multi-select (only shown for income filter)
5. **Search**: Free text search in description

**Additional Features:**
- Clear all filters button
- Save filter presets
- Export filtered view
- Pagination (20-50 items per page)

**Foundation for Future:**
- Prepare for editable entries
- Prepare for audit logging UI
- Prepare for statement source linking

---

### Feature 7: Monthly Analysis: Charts and Metrics

**Purpose:** Understand monthly financial performance with trends.

**Metrics:**

1. **Monthly Summary Cards**
   - Total Income
   - Total Expenses
   - Net Surplus/Deficit
   - Comparison to previous month (% change)

2. **Daily Spend Bar Chart**
   - X-axis: Days of month (1-31)
   - Y-axis: Cumulative spending for that day
   - Color: Blue bars
   - Hover to see exact amount
   - Shows only days with transactions

3. **Month-on-Month Comparison**
   - Last 12 months comparison
   - Income line (green)
   - Expense line (red)
   - Net flow area (neutral color)
   - Legend and tooltip

4. **Month Navigation**
   - Previous/Next month buttons
   - Current month display
   - Quick month selector dropdown

**Design:**
- Clean, minimal charts
- No 3D effects
- No animations
- Clear labels and legend
- Responsive to screen size

**Data Source:**
- Historical, user-approved transactions only
- No forecasting
- No estimated values

---

### Feature 8: Category Analytics: Trends & Distribution

**Purpose:** Understand spending patterns by category.

**Visualizations:**

1. **Expense Category Pie Chart**
   - Shows all expense categories
   - Percentage of total spending
   - Click to filter transaction history
   - No explosions or animations
   - Clear labels and legend

2. **Category Totals Table**
   - List all categories
   - Total amount per category
   - Percentage of total
   - Sortable by amount or name
   - Count of transactions per category

3. **Category Trends Line Chart**
   - Last 12 months
   - One line per category
   - Show all or select specific categories
   - Tooltip with exact values

4. **Comparison Options**
   - View trends over:
     - Last 3 months
     - Last 6 months
     - Last 12 months
     - Custom date range

**Design Requirements:**
- Clean, accessible color palette
- No gamification elements
- No "pressure" colors (aggressive reds)
- Neutral, calming aesthetic
- Numbers-focused
- High contrast for accessibility

**Data:**
- User-approved transactions only
- Ignore manually marked as ignored
- Always accurate

---

### Feature 9: Income Analytics: Source Tracking & Insights

**Purpose:** Understand income composition and source stability.

**Analysis:**

1. **Income Per Source (Current Month)**
   - List all active sources
   - Amount received this month
   - Total from all sources
   - Empty state if no income this month

2. **Monthly Breakdown by Source**
   - Table: Months as rows, Sources as columns
   - Stacked bar chart: Monthly totals by source
   - Show last 12 months

3. **Stability Analysis**
   - Classify sources as Stable or Volatile
   - Logic:
     - **Stable**: Consistent amounts month-to-month (std dev < 15%)
     - **Volatile**: Irregular amounts
   - Show classification badge

4. **Derived Insights (Historical Only)**
   - Most stable income source
   - Most volatile income source
   - Best income month (highest total received)
   - Worst income month (lowest total)
   - Average monthly income per source
   - Zero-income months (no income received)
   - Best source (highest total received)

5. **Trends**
   - 12-month trend line per source
   - Average line across all sources
   - Comparison view (best vs. worst source)

**Visualizations:**
- Stacked bar chart (monthly by source)
- Line chart (trend per source)
- Metric cards (insights summary)
- Source comparison table

**Data Source:**
- Historical, actual transactions only
- No predictions
- No forecasting

---

### Feature 10: Ignore Rule Management: Full Control UI

**Purpose:** User-controlled filtering with complete transparency.

**Core Principles:**
- NO auto-created rules
- User makes all decisions
- Rules always visible
- Fully reversible
- Complete audit trail

**Operations:**

1. **Create Rules**
   - Text input field for match pattern
   - Scope selector:
     - "This import only" (applies to current PDF)
     - "Future imports" (applies to all future PDFs)
   - Save button
   - Display confirmation

2. **View Rules**
   - List all rules (active and disabled)
   - For each rule:
     - Match text
     - Scope
     - Status (enabled/disabled)
     - Created date
     - Actions (edit, disable, delete, restore)
   - Count of transactions matched by each rule

3. **Edit Rules**
   - Change match text
   - Change scope
   - Save changes
   - Display confirmation

4. **Enable/Disable Rules**
   - Toggle without deleting
   - Disabled rules don't apply
   - Can be re-enabled anytime
   - Shows disabled status clearly

5. **Delete Rules**
   - Confirmation required
   - Shows impact (number of transactions affected)
   - Deleted rules shown in history

6. **Restore Ignored Transactions**
   - View all transactions ignored by rules
   - Select transactions to restore
   - Restore moves to pending approval
   - Audit trail recorded

**Requirements:**
- Rules only created by explicit user action
- All rules visible and accessible
- Ignored transactions always restorable
- No hidden operations
- Full audit log of all rule changes

---

### Feature 11: Statement Management: Import & Delete Flow

**Purpose:** Manage imported PDF statements and maintain data integrity.

**Operations:**

1. **View Imported Statements**
   - List all imported statements
   - For each statement:
     - Bank name
     - Period (start date - end date)
     - Number of transactions imported
     - Total debit
     - Total credit
     - Import date/time
     - File name (if available)

2. **View Statement Details**
   - Click statement to see:
     - All transactions from that statement
     - Summary (count, total debit, total credit)
     - Date range
     - Source information

3. **Delete Statement**
   - Delete button on each statement
   - Confirmation dialog:
     - Show count of transactions
     - Clearly state: "Only transactions from this statement will be deleted"
     - Show: "Manual entries and other imports will NOT be affected"
   - Explicit "Confirm Delete" and "Cancel" buttons
   - After deletion:
     - Remove transactions from this statement
     - Delete statement record
     - Audit log entry created
     - Show success message

**Requirements:**
- Prevent accidental deletion with confirmation
- Never delete manual entries
- Never delete transactions from other imports
- Show impact before deletion
- Complete audit trail

---

### Feature 12: Export & Report Generation

**Purpose:** Allow users to export and share financial data.

**Report Types:**

#### Monthly Report
- Month selector
- Includes:
  - All transactions for the month (daily list)
  - Monthly summary (income, expenses, net)
  - Category breakdown
  - Income sources summary
  - Charts (daily spend, category pie)

#### Yearly Summary
- Year selector
- Includes:
  - Monthly summaries (12 months table)
  - Annual totals (income, expenses, net)
  - Best and worst month analysis
  - Category trends across year
  - Income source trends across year

#### Category Breakdown
- Date range selector
- Includes:
  - All categories with totals
  - Percentage distribution
  - Monthly trend per category
  - Top 5 spending categories

#### Income Source Summary
- Date range selector
- Includes:
  - Income per source (totals)
  - Monthly breakdown by source
  - Stability metrics per source
  - Trend analysis

**Export Formats:**

1. **PDF**
   - Professionally formatted
   - Include charts and visualizations
   - Include summary tables
   - A4 page size
   - Print-friendly

2. **CSV**
   - Raw transaction data
   - Columns: Date, Amount, Direction, Category, Source, Description, Balance
   - One transaction per row
   - Can be imported to Excel

3. **Excel**
   - Multiple sheets:
     - Summary sheet
     - Transactions sheet
     - Categories sheet
     - Income sources sheet
   - Formatted with colors and borders
   - Summary charts (basic)

**Security & Privacy:**
- Only export user-logged transactions
- NO forecasting data
- NO credentials or auth tokens
- NO passwords or sensitive data
- NO live sync information
- NO unconfirmed transactions
- Generated locally (no cloud transmission)
- User controls what's included

**Export Dialog:**
- Report type selector
- Date range picker
- Format selector (PDF/CSV/Excel)
- Options:
  - Include charts (for PDF/Excel)
  - Include transaction details
  - Group by category/source
- Generate button
- Show progress
- Download when ready

**Requirements:**
- Quick generation (< 5 seconds)
- Accurate data
- No data loss
- User can regenerate anytime
- Audit log records all exports

---

## Data Privacy & Security

### Principles
1. **Local Storage Only**
   - No cloud synchronization
   - No external API calls for data transmission
   - All data stays on user's device

2. **No Automatic Collection**
   - No analytics tracking
   - No usage statistics
   - No behavior monitoring

3. **Export Security**
   - No credentials exported
   - No auth tokens exported
   - No API keys exported
   - User explicitly selects what to include

4. **Audit Trail**
   - All operations logged
   - User can review audit trail
   - Timestamp and action recorded
   - No logs transmitted externally

### Recommended Implementation
- Use browser's local storage or IndexedDB
- Implement end-to-end encryption for local storage
- Use HTTPS for any local server communication
- Implement proper session management
- No third-party analytics libraries

---

## Performance Requirements

- **Load Time**: Page load < 2 seconds
- **Chart Rendering**: Charts render < 1 second
- **Data Operations**: Save/delete < 500ms
- **PDF Parsing**: Parse typical statement < 5 seconds
- **Export Generation**: Generate report < 5 seconds

---

## Accessibility Requirements

- **WCAG 2.1 Level AA** compliance
- **Color Contrast**: 4.5:1 for text
- **Keyboard Navigation**: All features accessible via keyboard
- **Screen Reader**: Proper ARIA labels and semantic HTML
- **Mobile**: Touch targets minimum 48x48px
- **Responsive**: Works on mobile, tablet, desktop

---

## Browser Support

- **Chrome**: Latest 2 versions
- **Firefox**: Latest 2 versions
- **Safari**: Latest 2 versions
- **Edge**: Latest 2 versions
- **Mobile**: iOS 12+, Android 8+

---

## Conclusion

The Wallet Tracker is designed with user privacy, transparency, and control as foundational principles. Every feature prioritizes user choice and data accuracy over convenience or automation.
