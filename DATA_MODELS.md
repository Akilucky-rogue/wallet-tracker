# Wallet Tracker - Data Models & Database Schema

## Overview
This document defines the complete data model for the Wallet Tracker application, including entity definitions, relationships, constraints, and implementation details.

---

## Entity Relationship Diagram (ERD)

```
┌─────────────────┐         ┌──────────────────┐
│   IncomeSource  │         │    Transaction   │
├─────────────────┤         ├──────────────────┤
│ id (PK)         │◄────────│ id (PK)          │
│ name            │ 1:N     │ amount           │
│ type            │         │ direction        │
│ is_active       │         │ category         │
│ total_received  │         │ description      │
│ created_at      │         │ date             │
│ updated_at      │         │ source_id (FK)   │
└─────────────────┘         │ statement_id(FK) │
                            │ created_at       │
                            │ updated_at       │
                            │ is_deleted       │
                            └──────────────────┘
                                    ▲
                                    │
                            ┌───────┴────────┐
                            │                │
                    ┌──────────────────┐  ┌──────────────────┐
                    │    Statement     │  │   IgnoreRule     │
                    ├──────────────────┤  ├──────────────────┤
                    │ id (PK)          │  │ id (PK)          │
                    │ bank_name        │  │ match_text       │
                    │ period_start     │  │ scope            │
                    │ period_end       │  │ is_enabled       │
                    │ transaction_count│  │ created_at       │
                    │ total_debit      │  │ statement_id(FK) │
                    │ total_credit     │  └──────────────────┘
                    │ file_name        │
                    │ imported_at      │
                    │ created_at       │
                    └──────────────────┘
```

---

## Core Entities

### 1. Transaction
**Description**: Represents a single financial transaction (income or expense).

**Table Name**: `transactions`

**Columns**:

| Column | Type | Nullable | Unique | Default | Constraints |
|--------|------|----------|--------|---------|-------------|
| id | UUID | NO | YES | uuid_generate_v4() | PK |
| amount | DECIMAL(12,2) | NO | NO | - | > 0 |
| direction | ENUM | NO | NO | - | INCOME \| EXPENSE |
| category | VARCHAR(100) | NO | NO | - | - |
| description | VARCHAR(500) | YES | NO | NULL | - |
| date | DATETIME | NO | NO | - | - |
| source | VARCHAR(100) | YES | NO | NULL | - |
| source_id | UUID | YES | NO | NULL | FK → IncomeSource |
| statement_id | UUID | YES | NO | NULL | FK → Statement |
| created_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| updated_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| is_deleted | BOOLEAN | NO | NO | FALSE | - |

**Indexes**:
```sql
CREATE INDEX idx_transactions_date ON transactions(date);
CREATE INDEX idx_transactions_direction ON transactions(direction);
CREATE INDEX idx_transactions_category ON transactions(category);
CREATE INDEX idx_transactions_statement_id ON transactions(statement_id);
CREATE INDEX idx_transactions_source_id ON transactions(source_id);
CREATE INDEX idx_transactions_deleted ON transactions(is_deleted);
```

**Constraints**:
```sql
CONSTRAINT amount_positive CHECK (amount > 0);
CONSTRAINT valid_direction CHECK (direction IN ('INCOME', 'EXPENSE'));
CONSTRAINT valid_date CHECK (date <= CURRENT_TIMESTAMP);
```

**Example Data**:
```json
{
  "id": "550e8400-e29b-41d4-a716-446655440000",
  "amount": 50.00,
  "direction": "EXPENSE",
  "category": "Groceries",
  "description": "Weekly shopping at supermarket",
  "date": "2026-01-30T10:30:00Z",
  "source": null,
  "source_id": null,
  "statement_id": null,
  "created_at": "2026-01-30T10:35:00Z",
  "updated_at": "2026-01-30T10:35:00Z",
  "is_deleted": false
}
```

---

### 2. IncomeSource
**Description**: Represents a source of income (salary, freelance, investment, etc.).

**Table Name**: `income_sources`

**Columns**:

| Column | Type | Nullable | Unique | Default | Constraints |
|--------|------|----------|--------|---------|-------------|
| id | UUID | NO | YES | uuid_generate_v4() | PK |
| name | VARCHAR(100) | NO | YES | - | - |
| type | ENUM | NO | NO | - | SALARY \| FREELANCE \| INVESTMENT \| BONUS \| OTHER |
| is_active | BOOLEAN | NO | NO | TRUE | - |
| total_received | DECIMAL(12,2) | NO | NO | 0.00 | >= 0 |
| created_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| updated_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |

**Indexes**:
```sql
CREATE INDEX idx_income_sources_is_active ON income_sources(is_active);
CREATE INDEX idx_income_sources_type ON income_sources(type);
```

**Constraints**:
```sql
CONSTRAINT total_received_non_negative CHECK (total_received >= 0);
CONSTRAINT valid_type CHECK (type IN ('SALARY', 'FREELANCE', 'INVESTMENT', 'BONUS', 'OTHER'));
CONSTRAINT name_non_empty CHECK (name <> '');
```

**Example Data**:
```json
{
  "id": "660e8400-e29b-41d4-a716-446655440001",
  "name": "Full-time Salary",
  "type": "SALARY",
  "is_active": true,
  "total_received": 45000.00,
  "created_at": "2026-01-01T08:00:00Z",
  "updated_at": "2026-01-30T15:20:00Z"
}
```

---

### 3. Statement
**Description**: Represents an imported bank statement, tracking the import session.

**Table Name**: `statements`

**Columns**:

| Column | Type | Nullable | Unique | Default | Constraints |
|--------|------|----------|--------|---------|-------------|
| id | UUID | NO | YES | uuid_generate_v4() | PK |
| bank_name | VARCHAR(100) | NO | NO | - | - |
| period_start | DATE | NO | NO | - | - |
| period_end | DATE | NO | NO | - | period_end >= period_start |
| transaction_count | INTEGER | NO | NO | 0 | >= 0 |
| total_debit | DECIMAL(12,2) | NO | NO | 0.00 | >= 0 |
| total_credit | DECIMAL(12,2) | NO | NO | 0.00 | >= 0 |
| file_name | VARCHAR(255) | YES | NO | NULL | - |
| imported_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| created_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |

**Indexes**:
```sql
CREATE INDEX idx_statements_bank_name ON statements(bank_name);
CREATE INDEX idx_statements_period ON statements(period_start, period_end);
CREATE INDEX idx_statements_imported_at ON statements(imported_at);
```

**Constraints**:
```sql
CONSTRAINT period_valid CHECK (period_end >= period_start);
CONSTRAINT transaction_count_non_negative CHECK (transaction_count >= 0);
CONSTRAINT total_debit_non_negative CHECK (total_debit >= 0);
CONSTRAINT total_credit_non_negative CHECK (total_credit >= 0);
```

**Example Data**:
```json
{
  "id": "770e8400-e29b-41d4-a716-446655440002",
  "bank_name": "IDFC First Bank",
  "period_start": "2026-01-01",
  "period_end": "2026-01-31",
  "transaction_count": 45,
  "total_debit": 5000.00,
  "total_credit": 8000.00,
  "file_name": "statement_jan_2026.pdf",
  "imported_at": "2026-01-30T14:30:00Z",
  "created_at": "2026-01-30T14:30:00Z"
}
```

---

### 4. IgnoreRule
**Description**: Represents a rule to ignore certain transactions during imports.

**Table Name**: `ignore_rules`

**Columns**:

| Column | Type | Nullable | Unique | Default | Constraints |
|--------|------|----------|--------|---------|-------------|
| id | UUID | NO | YES | uuid_generate_v4() | PK |
| match_text | VARCHAR(255) | NO | NO | - | - |
| scope | ENUM | NO | NO | - | THIS_IMPORT_ONLY \| FUTURE_IMPORTS |
| is_enabled | BOOLEAN | NO | NO | TRUE | - |
| created_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| updated_at | DATETIME | NO | NO | CURRENT_TIMESTAMP | - |
| statement_id | UUID | YES | NO | NULL | FK → Statement |

**Indexes**:
```sql
CREATE INDEX idx_ignore_rules_enabled ON ignore_rules(is_enabled);
CREATE INDEX idx_ignore_rules_scope ON ignore_rules(scope);
CREATE INDEX idx_ignore_rules_statement ON ignore_rules(statement_id);
```

**Constraints**:
```sql
CONSTRAINT valid_scope CHECK (scope IN ('THIS_IMPORT_ONLY', 'FUTURE_IMPORTS'));
CONSTRAINT match_text_non_empty CHECK (match_text <> '');
```

**Business Rules**:
- Rules with scope='THIS_IMPORT_ONLY' must reference a statement_id
- Rules with scope='FUTURE_IMPORTS' should have statement_id=NULL

**Example Data**:
```json
{
  "id": "880e8400-e29b-41d4-a716-446655440003",
  "match_text": "Overdraft Fee",
  "scope": "FUTURE_IMPORTS",
  "is_enabled": true,
  "created_at": "2026-01-25T12:00:00Z",
  "updated_at": "2026-01-30T10:00:00Z",
  "statement_id": null
}
```

---

## Relationships

### 1. Transaction → IncomeSource
- **Type**: Many-to-One
- **Foreign Key**: transactions.source_id → income_sources.id
- **Cardinality**: Many transactions per income source
- **Nullable**: YES (for expense transactions)
- **On Delete**: SET NULL (preserve transaction history)
- **On Update**: CASCADE

### 2. Transaction → Statement
- **Type**: Many-to-One
- **Foreign Key**: transactions.statement_id → statements.id
- **Cardinality**: Many transactions per statement
- **Nullable**: YES (for manually entered transactions)
- **On Delete**: NO ACTION (prevent orphaned transactions)
- **On Update**: CASCADE

### 3. IgnoreRule → Statement
- **Type**: Many-to-One
- **Foreign Key**: ignore_rules.statement_id → statements.id
- **Cardinality**: Many rules per statement (for THIS_IMPORT_ONLY scope)
- **Nullable**: YES (for FUTURE_IMPORTS scope)
- **On Delete**: CASCADE (delete rules when statement is deleted)
- **On Update**: CASCADE

---

## Queries & Access Patterns

### Frequently Used Queries

**1. Get all transactions for a date range:**
```sql
SELECT * FROM transactions
WHERE date BETWEEN ? AND ?
  AND is_deleted = FALSE
ORDER BY date DESC;
```

**2. Get total income and expenses for a month:**
```sql
SELECT 
  SUM(CASE WHEN direction = 'INCOME' THEN amount ELSE 0 END) as total_income,
  SUM(CASE WHEN direction = 'EXPENSE' THEN amount ELSE 0 END) as total_expenses
FROM transactions
WHERE MONTH(date) = ? AND YEAR(date) = ?
  AND is_deleted = FALSE;
```

**3. Get spending by category:**
```sql
SELECT category, SUM(amount) as total, COUNT(*) as count
FROM transactions
WHERE direction = 'EXPENSE'
  AND date BETWEEN ? AND ?
  AND is_deleted = FALSE
GROUP BY category
ORDER BY total DESC;
```

**4. Get income by source:**
```sql
SELECT 
  s.name, 
  SUM(t.amount) as total, 
  COUNT(*) as count
FROM transactions t
LEFT JOIN income_sources s ON t.source_id = s.id
WHERE t.direction = 'INCOME'
  AND t.date BETWEEN ? AND ?
  AND t.is_deleted = FALSE
GROUP BY s.id, s.name
ORDER BY total DESC;
```

**5. Find potential duplicates:**
```sql
SELECT *
FROM transactions
WHERE amount = ? AND date = ? AND category = ?
  AND is_deleted = FALSE;
```

---

## Data Integrity & Constraints

### Business Rules
1. **Amount**: Must be positive (> 0)
2. **Direction**: Must be INCOME or EXPENSE
3. **Date**: Cannot be in future
4. **Income Source**: Required for direction='INCOME'
5. **Category**: Required for all transactions
6. **Statement Period**: end_date >= start_date
7. **Ignore Rule Scope**: 
   - THIS_IMPORT_ONLY requires statement_id
   - FUTURE_IMPORTS requires statement_id = NULL

### Audit Trail
- All timestamps use UTC timezone
- created_at and updated_at are immutable for created_at
- updated_at updated on any modification
- Soft deletes (is_deleted flag) for transactions

---

## Storage Recommendations

### Database Engine
- **PostgreSQL** (recommended)
  - Full ACID compliance
  - UUID support
  - JSON support
  - Good indexing capabilities

- **MySQL 8.0+**
  - ACID compliance
  - UUID support
  - Adequate performance

### Indexes
- Primary keys on all tables
- Foreign keys with indexes
- Composite index on (date, direction) for transaction queries
- Index on (is_deleted) for filtering

### Backups
- Daily automated backups
- Point-in-time recovery capability
- Local backup storage (no cloud transmission)

---

## Performance Optimization

### Pagination
- Use limit/offset for transaction lists
- Default: 20-50 rows per page

### Caching
- Cache frequently accessed IncomeSource list
- Cache category list
- Cache monthly summaries (update on new transaction)

### Denormalization (Optional)
- Store total_received on IncomeSource (updated on new income transaction)
- Store transaction_count, total_debit, total_credit on Statement

---

## Migration Strategy

### Initial Setup
1. Create tables in order: IncomeSource → Statement → Transaction → IgnoreRule
2. Create indexes
3. Create constraints
4. Create views (optional)

### Version Control
- Use migration tool (Flyway, Liquibase, Alembic)
- Track all schema changes
- Maintain backward compatibility where possible

---

## Example Data Model in Code

### TypeScript Interface
```typescript
interface Transaction {
  id: string; // UUID
  amount: number;
  direction: 'INCOME' | 'EXPENSE';
  category: string;
  description?: string;
  date: Date;
  source?: string;
  sourceId?: string; // UUID
  statementId?: string; // UUID
  createdAt: Date;
  updatedAt: Date;
  isDeleted: boolean;
}

interface IncomeSource {
  id: string; // UUID
  name: string;
  type: 'SALARY' | 'FREELANCE' | 'INVESTMENT' | 'BONUS' | 'OTHER';
  isActive: boolean;
  totalReceived: number;
  createdAt: Date;
  updatedAt: Date;
}

interface Statement {
  id: string; // UUID
  bankName: string;
  periodStart: Date;
  periodEnd: Date;
  transactionCount: number;
  totalDebit: number;
  totalCredit: number;
  fileName?: string;
  importedAt: Date;
  createdAt: Date;
}

interface IgnoreRule {
  id: string; // UUID
  matchText: string;
  scope: 'THIS_IMPORT_ONLY' | 'FUTURE_IMPORTS';
  isEnabled: boolean;
  createdAt: Date;
  updatedAt: Date;
  statementId?: string; // UUID
}
```

---
