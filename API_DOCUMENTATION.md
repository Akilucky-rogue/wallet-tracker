# Wallet Tracker - API Documentation

## Overview

This document provides complete API documentation for the Wallet Tracker backend, including all endpoints, request/response formats, authentication, error handling, and usage examples.

**Base URL**: `http://localhost:5000/api/v1`

**Version**: 1.0.0

**Last Updated**: January 30, 2026

---

## Table of Contents

1. [Authentication](#authentication)
2. [Error Handling](#error-handling)
3. [Transactions API](#transactions-api)
4. [Income Sources API](#income-sources-api)
5. [Statements API](#statements-api)
6. [Ignore Rules API](#ignore-rules-api)
7. [Analytics API](#analytics-api)
8. [Import API](#import-api)
9. [Export API](#export-api)
10. [Rate Limiting](#rate-limiting)

---

## Authentication

### Overview
The API uses **Bearer Token** authentication. All requests must include an `Authorization` header.

**Note**: In a local/desktop environment, authentication may be simplified or bypassed.

### Token Format
```
Authorization: Bearer <token>
```

### Example Request
```bash
curl -H "Authorization: Bearer your_token_here" \
  http://localhost:5000/api/v1/transactions
```

---

## Error Handling

### Error Response Format
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request parameters",
    "details": [
      {
        "field": "amount",
        "message": "Amount must be greater than 0"
      }
    ]
  },
  "timestamp": "2026-01-30T14:30:00Z"
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|------------|-------------|
| VALIDATION_ERROR | 400 | Request validation failed |
| UNAUTHORIZED | 401 | Authentication required |
| FORBIDDEN | 403 | Access denied |
| NOT_FOUND | 404 | Resource not found |
| CONFLICT | 409 | Resource already exists |
| INTERNAL_ERROR | 500 | Server error |
| DATABASE_ERROR | 500 | Database operation failed |

### HTTP Status Codes
- **200**: Success
- **201**: Created
- **204**: No Content
- **400**: Bad Request
- **401**: Unauthorized
- **403**: Forbidden
- **404**: Not Found
- **409**: Conflict
- **422**: Unprocessable Entity
- **500**: Internal Server Error

---

## Transactions API

### 1. Get All Transactions

**Endpoint**: `GET /transactions`

**Description**: Retrieve all transactions with optional filtering and pagination.

**Query Parameters**:
```
?fromDate=2026-01-01&toDate=2026-01-31&direction=EXPENSE&category=Groceries&page=1&limit=20
```

| Parameter | Type | Required | Default | Description |
|-----------|------|----------|---------|-------------|
| fromDate | string (YYYY-MM-DD) | No | - | Start date filter (inclusive) |
| toDate | string (YYYY-MM-DD) | No | - | End date filter (inclusive) |
| direction | string | No | - | INCOME or EXPENSE |
| category | string | No | - | Category filter (exact match) |
| sourceId | string (UUID) | No | - | Filter by income source |
| search | string | No | - | Free text search in description |
| page | number | No | 1 | Page number |
| limit | number | No | 20 | Items per page (max 100) |
| sort | string | No | -date | Sort field: date, amount, category |

**Request Example**:
```bash
GET /api/v1/transactions?fromDate=2026-01-01&toDate=2026-01-31&direction=EXPENSE&page=1&limit=20
Authorization: Bearer <token>
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "transactions": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "amount": 50.00,
        "direction": "EXPENSE",
        "category": "Groceries",
        "description": "Weekly shopping",
        "date": "2026-01-30T10:30:00Z",
        "source": null,
        "sourceId": null,
        "statementId": null,
        "createdAt": "2026-01-30T10:35:00Z",
        "updatedAt": "2026-01-30T10:35:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 145,
      "totalPages": 8
    }
  }
}
```

---

### 2. Create Transaction

**Endpoint**: `POST /transactions`

**Description**: Create a new transaction.

**Request Headers**:
```
Content-Type: application/json
Authorization: Bearer <token>
```

**Request Body**:
```json
{
  "amount": 50.00,
  "direction": "EXPENSE",
  "category": "Groceries",
  "description": "Weekly shopping at supermarket",
  "date": "2026-01-30",
  "sourceId": null,
  "notes": "Optional notes"
}
```

**Field Descriptions**:
| Field | Type | Required | Rules |
|-------|------|----------|-------|
| amount | number | Yes | > 0, max 2 decimals |
| direction | string | Yes | INCOME or EXPENSE |
| category | string | Yes | 1-100 characters |
| description | string | No | 1-500 characters |
| date | string (YYYY-MM-DD) | Yes | Cannot be future date |
| sourceId | string (UUID) | No | Required if direction=INCOME |
| notes | string | No | 1-500 characters |

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "amount": 50.00,
    "direction": "EXPENSE",
    "category": "Groceries",
    "description": "Weekly shopping at supermarket",
    "date": "2026-01-30T00:00:00Z",
    "sourceId": null,
    "statementId": null,
    "createdAt": "2026-01-30T14:30:00Z",
    "updatedAt": "2026-01-30T14:30:00Z"
  }
}
```

**Error Response (400 Bad Request)**:
```json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Validation failed",
    "details": [
      {
        "field": "amount",
        "message": "Amount must be greater than 0"
      },
      {
        "field": "sourceId",
        "message": "Income transactions require a source"
      }
    ]
  }
}
```

---

### 3. Get Transaction by ID

**Endpoint**: `GET /transactions/{id}`

**Description**: Retrieve a specific transaction.

**Parameters**:
| Parameter | Type | Required |
|-----------|------|----------|
| id | string (UUID) | Yes |

**Request Example**:
```bash
GET /api/v1/transactions/550e8400-e29b-41d4-a716-446655440000
Authorization: Bearer <token>
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "amount": 50.00,
    "direction": "EXPENSE",
    "category": "Groceries",
    "description": "Weekly shopping",
    "date": "2026-01-30T00:00:00Z",
    "sourceId": null,
    "statementId": null,
    "createdAt": "2026-01-30T14:30:00Z",
    "updatedAt": "2026-01-30T14:30:00Z"
  }
}
```

---

### 4. Update Transaction

**Endpoint**: `PUT /transactions/{id}`

**Description**: Update an existing transaction.

**Request Body**:
```json
{
  "amount": 55.00,
  "category": "Groceries",
  "description": "Weekly shopping - updated",
  "date": "2026-01-30"
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "550e8400-e29b-41d4-a716-446655440000",
    "amount": 55.00,
    "direction": "EXPENSE",
    "category": "Groceries",
    "description": "Weekly shopping - updated",
    "date": "2026-01-30T00:00:00Z",
    "updatedAt": "2026-01-30T15:00:00Z"
  }
}
```

---

### 5. Delete Transaction

**Endpoint**: `DELETE /transactions/{id}`

**Description**: Delete a transaction (soft delete).

**Response (204 No Content)**:
```
No response body
```

---

## Income Sources API

### 1. Get All Income Sources

**Endpoint**: `GET /income-sources`

**Description**: Retrieve all income sources.

**Query Parameters**:
```
?isActive=true&sort=totalReceived
```

| Parameter | Type | Default |
|-----------|------|---------|
| isActive | boolean | - |
| sort | string | name |

**Request Example**:
```bash
GET /api/v1/income-sources?isActive=true
Authorization: Bearer <token>
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "sources": [
      {
        "id": "660e8400-e29b-41d4-a716-446655440001",
        "name": "Full-time Salary",
        "type": "SALARY",
        "isActive": true,
        "totalReceived": 45000.00,
        "createdAt": "2026-01-01T08:00:00Z",
        "updatedAt": "2026-01-30T15:20:00Z"
      },
      {
        "id": "660e8400-e29b-41d4-a716-446655440002",
        "name": "Freelance Work",
        "type": "FREELANCE",
        "isActive": true,
        "totalReceived": 18500.00,
        "createdAt": "2026-01-15T08:00:00Z",
        "updatedAt": "2026-01-25T15:20:00Z"
      }
    ]
  }
}
```

---

### 2. Create Income Source

**Endpoint**: `POST /income-sources`

**Description**: Create a new income source.

**Request Body**:
```json
{
  "name": "Investment Returns",
  "type": "INVESTMENT"
}
```

**Field Descriptions**:
| Field | Type | Required | Rules |
|-------|------|----------|-------|
| name | string | Yes | 1-100 characters, unique |
| type | string | Yes | SALARY, FREELANCE, INVESTMENT, BONUS, OTHER |

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440003",
    "name": "Investment Returns",
    "type": "INVESTMENT",
    "isActive": true,
    "totalReceived": 0.00,
    "createdAt": "2026-01-30T14:30:00Z",
    "updatedAt": "2026-01-30T14:30:00Z"
  }
}
```

---

### 3. Update Income Source

**Endpoint**: `PUT /income-sources/{id}`

**Description**: Update an income source.

**Request Body**:
```json
{
  "name": "Investment Returns - Updated",
  "type": "INVESTMENT",
  "isActive": true
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "660e8400-e29b-41d4-a716-446655440003",
    "name": "Investment Returns - Updated",
    "type": "INVESTMENT",
    "isActive": true,
    "totalReceived": 0.00,
    "updatedAt": "2026-01-30T15:00:00Z"
  }
}
```

---

### 4. Get Income Source Details

**Endpoint**: `GET /income-sources/{id}/details`

**Description**: Get detailed information about an income source including transactions and statistics.

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "source": {
      "id": "660e8400-e29b-41d4-a716-446655440001",
      "name": "Full-time Salary",
      "type": "SALARY",
      "isActive": true,
      "totalReceived": 45000.00
    },
    "statistics": {
      "transactionCount": 12,
      "averageAmount": 3750.00,
      "lastReceivedDate": "2026-01-28T00:00:00Z",
      "stability": "STABLE",
      "stabilityScore": 0.98,
      "monthlyTrend": [
        {
          "month": "2025-12",
          "amount": 5000.00,
          "transactionCount": 1
        },
        {
          "month": "2025-11",
          "amount": 5000.00,
          "transactionCount": 1
        }
      ]
    },
    "recentTransactions": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "amount": 5000.00,
        "date": "2026-01-28T00:00:00Z",
        "description": "Jan 2026 Salary"
      }
    ]
  }
}
```

---

## Statements API

### 1. Get All Statements

**Endpoint**: `GET /statements`

**Description**: Retrieve all imported statements.

**Query Parameters**:
```
?bankName=IDFC&page=1&limit=20
```

**Request Example**:
```bash
GET /api/v1/statements?page=1&limit=20
Authorization: Bearer <token>
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "statements": [
      {
        "id": "770e8400-e29b-41d4-a716-446655440002",
        "bankName": "IDFC First Bank",
        "periodStart": "2026-01-01",
        "periodEnd": "2026-01-31",
        "transactionCount": 45,
        "totalDebit": 5000.00,
        "totalCredit": 8000.00,
        "fileName": "statement_jan_2026.pdf",
        "importedAt": "2026-01-30T14:30:00Z",
        "createdAt": "2026-01-30T14:30:00Z"
      }
    ],
    "pagination": {
      "page": 1,
      "limit": 20,
      "total": 12,
      "totalPages": 1
    }
  }
}
```

---

### 2. Get Statement Details

**Endpoint**: `GET /statements/{id}`

**Description**: Get all transactions from a specific statement.

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "statement": {
      "id": "770e8400-e29b-41d4-a716-446655440002",
      "bankName": "IDFC First Bank",
      "periodStart": "2026-01-01",
      "periodEnd": "2026-01-31",
      "transactionCount": 45,
      "totalDebit": 5000.00,
      "totalCredit": 8000.00,
      "fileName": "statement_jan_2026.pdf",
      "importedAt": "2026-01-30T14:30:00Z"
    },
    "transactions": [
      {
        "id": "550e8400-e29b-41d4-a716-446655440000",
        "amount": 50.00,
        "direction": "EXPENSE",
        "date": "2026-01-30T00:00:00Z",
        "description": "Grocery purchase"
      }
    ]
  }
}
```

---

### 3. Delete Statement

**Endpoint**: `DELETE /statements/{id}`

**Description**: Delete a statement and all its associated transactions.

**Response (204 No Content)**:
```
No response body
```

**Important**: This only deletes transactions from the specific statement, not manual entries or other imports.

---

## Ignore Rules API

### 1. Get All Ignore Rules

**Endpoint**: `GET /ignore-rules`

**Description**: Retrieve all ignore rules.

**Query Parameters**:
```
?isEnabled=true&scope=FUTURE_IMPORTS
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "rules": [
      {
        "id": "880e8400-e29b-41d4-a716-446655440003",
        "matchText": "Interest Charge",
        "scope": "FUTURE_IMPORTS",
        "isEnabled": true,
        "matchCount": 12,
        "createdAt": "2026-01-25T12:00:00Z",
        "updatedAt": "2026-01-30T10:00:00Z"
      }
    ]
  }
}
```

---

### 2. Create Ignore Rule

**Endpoint**: `POST /ignore-rules`

**Description**: Create a new ignore rule.

**Request Body**:
```json
{
  "matchText": "Overdraft Fee",
  "scope": "FUTURE_IMPORTS",
  "statementId": null
}
```

**Field Descriptions**:
| Field | Type | Required | Rules |
|-------|------|----------|-------|
| matchText | string | Yes | 1-255 characters |
| scope | string | Yes | THIS_IMPORT_ONLY or FUTURE_IMPORTS |
| statementId | string (UUID) | Conditional | Required if scope=THIS_IMPORT_ONLY |

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "id": "880e8400-e29b-41d4-a716-446655440004",
    "matchText": "Overdraft Fee",
    "scope": "FUTURE_IMPORTS",
    "isEnabled": true,
    "createdAt": "2026-01-30T14:30:00Z",
    "updatedAt": "2026-01-30T14:30:00Z"
  }
}
```

---

### 3. Update Ignore Rule

**Endpoint**: `PUT /ignore-rules/{id}`

**Description**: Update an ignore rule.

**Request Body**:
```json
{
  "matchText": "Overdraft Fee - Updated",
  "isEnabled": true
}
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "id": "880e8400-e29b-41d4-a716-446655440004",
    "matchText": "Overdraft Fee - Updated",
    "scope": "FUTURE_IMPORTS",
    "isEnabled": true,
    "updatedAt": "2026-01-30T15:00:00Z"
  }
}
```

---

### 4. Delete Ignore Rule

**Endpoint**: `DELETE /ignore-rules/{id}`

**Response (204 No Content)**:
```
No response body
```

---

## Analytics API

### 1. Get Monthly Analysis

**Endpoint**: `GET /analytics/monthly`

**Description**: Get monthly financial analysis.

**Query Parameters**:
```
?month=2026-01&compare=true
```

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| month | string (YYYY-MM) | Yes | Month to analyze |
| compare | boolean | No | Include comparison to previous month |

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "month": "2026-01",
    "summary": {
      "totalIncome": 8500.00,
      "totalExpense": 5200.00,
      "netFlow": 3300.00
    },
    "comparison": {
      "previousMonth": "2025-12",
      "previousIncome": 7200.00,
      "incomeChange": 1300.00,
      "incomeChangePercent": 18.06,
      "previousExpense": 5500.00,
      "expenseChange": -300.00,
      "expenseChangePercent": -5.45
    },
    "dailyBreakdown": [
      {
        "day": 1,
        "amount": 0.00,
        "transactionCount": 0
      },
      {
        "day": 2,
        "amount": 50.00,
        "transactionCount": 1
      }
    ]
  }
}
```

---

### 2. Get Category Analytics

**Endpoint**: `GET /analytics/categories`

**Description**: Get spending breakdown by category.

**Query Parameters**:
```
?fromDate=2026-01-01&toDate=2026-01-31&includeIncome=false
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "dateRange": {
      "fromDate": "2026-01-01",
      "toDate": "2026-01-31"
    },
    "totalExpense": 5200.00,
    "categories": [
      {
        "category": "Groceries",
        "total": 1820.00,
        "percentage": 35.0,
        "transactionCount": 12,
        "trend": [
          {
            "month": "2025-12",
            "amount": 1650.00
          },
          {
            "month": "2026-01",
            "amount": 1820.00
          }
        ]
      }
    ]
  }
}
```

---

### 3. Get Income Analytics

**Endpoint**: `GET /analytics/income`

**Description**: Get income analysis by source.

**Query Parameters**:
```
?fromDate=2026-01-01&toDate=2026-01-31
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "dateRange": {
      "fromDate": "2026-01-01",
      "toDate": "2026-01-31"
    },
    "totalIncome": 8500.00,
    "sources": [
      {
        "sourceId": "660e8400-e29b-41d4-a716-446655440001",
        "name": "Full-time Salary",
        "type": "SALARY",
        "amount": 5000.00,
        "percentage": 58.82,
        "transactionCount": 1,
        "stability": "STABLE",
        "stabilityScore": 0.98
      }
    ],
    "insights": {
      "mostStableSource": {
        "name": "Full-time Salary",
        "stabilityScore": 0.98
      },
      "mostVolatileSource": {
        "name": "Freelance Work",
        "stabilityScore": 0.45
      },
      "bestMonth": "2026-01",
      "bestMonthAmount": 8500.00,
      "worstMonth": "2025-11",
      "worstMonthAmount": 6200.00
    }
  }
}
```

---

## Import API

### 1. Upload PDF Statement

**Endpoint**: `POST /import/upload`

**Description**: Upload and parse a PDF bank statement.

**Request Format**: `multipart/form-data`

**Fields**:
| Field | Type | Required |
|-------|------|----------|
| file | file | Yes |
| bankName | string | Yes |

**Request Example**:
```bash
curl -X POST http://localhost:5000/api/v1/import/upload \
  -H "Authorization: Bearer <token>" \
  -F "file=@statement.pdf" \
  -F "bankName=IDFC First Bank"
```

**Response (200 OK)**:
```json
{
  "success": true,
  "data": {
    "importId": "import-12345",
    "fileName": "statement_jan_2026.pdf",
    "bankName": "IDFC First Bank",
    "periodStart": "2026-01-01",
    "periodEnd": "2026-01-31",
    "transactionCount": 45,
    "parsedTransactions": [
      {
        "tempId": "temp-1",
        "date": "2026-01-31",
        "description": "Salary",
        "debit": 0,
        "credit": 5000.00,
        "balance": 15432.50
      }
    ]
  }
}
```

---

### 2. Confirm Import

**Endpoint**: `POST /import/confirm`

**Description**: Confirm and save imported transactions.

**Request Body**:
```json
{
  "importId": "import-12345",
  "approvedTransactions": ["temp-1", "temp-2"],
  "ignoredTransactions": [],
  "ignoredByRules": []
}
```

**Response (201 Created)**:
```json
{
  "success": true,
  "data": {
    "statementId": "770e8400-e29b-41d4-a716-446655440002",
    "imported": 43,
    "ignored": 2,
    "duplicates": 0,
    "message": "43 transactions imported successfully"
  }
}
```

---

## Export API

### 1. Generate Report

**Endpoint**: `POST /export/generate`

**Description**: Generate a report in specified format.

**Request Body**:
```json
{
  "reportType": "MONTHLY",
  "format": "PDF",
  "fromDate": "2026-01-01",
  "toDate": "2026-01-31",
  "options": {
    "includeCharts": true,
    "includeSummary": true,
    "includeTransactions": true
  }
}
```

**Field Descriptions**:
| Field | Type | Required | Options |
|-------|------|----------|---------|
| reportType | string | Yes | MONTHLY, YEARLY, CATEGORY, INCOME_SOURCE |
| format | string | Yes | PDF, CSV, EXCEL |
| fromDate | string | Yes | YYYY-MM-DD |
| toDate | string | Yes | YYYY-MM-DD |
| options | object | No | includeCharts, includeSummary, includeTransactions |

**Response (200 OK - streaming file)**:
```
File binary content
Content-Type: application/pdf (or application/csv, application/vnd.ms-excel)
Content-Disposition: attachment; filename="report.pdf"
```

---

## Rate Limiting

### Rate Limit Headers
All responses include rate limit information:

```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1643554200
```

### Limits
- **Authenticated requests**: 1000 requests per hour
- **Public endpoints**: 100 requests per hour
- **File uploads**: 10 uploads per hour

### Error Response (429 Too Many Requests)
```json
{
  "success": false,
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Too many requests. Please try again later.",
    "retryAfter": 3600
  }
}
```

---

## Best Practices

### 1. Error Handling
Always check `success` field and handle errors gracefully.

### 2. Pagination
For large datasets, always use pagination to improve performance.

### 3. Caching
Cache frequently accessed data (income sources, categories) to reduce API calls.

### 4. Timestamps
All timestamps are in ISO 8601 format (UTC). Convert to local time on client side.

### 5. Date Formats
Always use YYYY-MM-DD for dates and YYYY-MM for months.

### 6. Validation
Validate all inputs on client side before sending to API.

---

## Code Examples

### JavaScript/Node.js
```javascript
const fetchTransactions = async (fromDate, toDate) => {
  try {
    const response = await fetch(
      `http://localhost:5000/api/v1/transactions?fromDate=${fromDate}&toDate=${toDate}`,
      {
        headers: {
          'Authorization': `Bearer ${token}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    if (!response.ok) throw new Error('API request failed');
    
    const data = await response.json();
    return data.data.transactions;
  } catch (error) {
    console.error('Error fetching transactions:', error);
  }
};
```

### Python
```python
import requests

def create_transaction(amount, direction, category, date):
    headers = {
        'Authorization': f'Bearer {token}',
        'Content-Type': 'application/json'
    }
    
    payload = {
        'amount': amount,
        'direction': direction,
        'category': category,
        'date': date
    }
    
    response = requests.post(
        'http://localhost:5000/api/v1/transactions',
        headers=headers,
        json=payload
    )
    
    if response.status_code == 201:
        return response.json()['data']
    else:
        raise Exception(f"Error: {response.json()}")
```

### cURL
```bash
# Get transactions
curl -H "Authorization: Bearer <token>" \
  "http://localhost:5000/api/v1/transactions?fromDate=2026-01-01&toDate=2026-01-31"

# Create transaction
curl -X POST http://localhost:5000/api/v1/transactions \
  -H "Authorization: Bearer <token>" \
  -H "Content-Type: application/json" \
  -d '{
    "amount": 50.00,
    "direction": "EXPENSE",
    "category": "Groceries",
    "date": "2026-01-30"
  }'
```

---
