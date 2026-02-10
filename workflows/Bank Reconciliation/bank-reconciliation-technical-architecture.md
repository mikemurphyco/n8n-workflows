# Bank Reconciliation Workflow - Technical Architecture

**Version:** 1.0
**Date:** January 29, 2026
**Workflow Type:** Scheduled Monthly Automation

---

## 🏗️ Architecture Overview

This workflow automates monthly bank reconciliation by connecting Truist Bank (via Plaid), FreshBooks, Google AdSense, and Google Sheets. The workflow fetches transactions, performs intelligent matching, updates accounting records, and generates reports.

**Execution Pattern:** Monthly scheduled run on the 5th at 9:00 AM EST

---

## 📦 n8n Nodes Availability

Based on research of n8n's 2026 capabilities:

| Service | Node Type | Availability | Implementation Notes |
|---------|-----------|--------------|---------------------|
| **Google Sheets** | Built-in App Node | ✅ Official | Full CRUD support |
| **Plaid** | Community Node | ✅ Community (`n8n-nodes-plaid` v2.0+) | Install via Community Nodes |
| **FreshBooks** | HTTP Request | ⚠️ No official node | Use HTTP Request with OAuth 2.0 |
| **Google AdSense** | HTTP Request | ⚠️ No official node | Use HTTP Request with OAuth 2.0 |
| **Email** | Built-in App Node | ✅ Official | SMTP or Gmail node |
| **Schedule** | Built-in Trigger | ✅ Official | Cron Trigger node |

### Installation Requirements

**Community Nodes to Install:**
```bash
# In n8n: Settings → Community Nodes → Install
n8n-nodes-plaid
```

**OAuth 2.0 Credentials Needed:**
- FreshBooks API (create OAuth app in FreshBooks)
- Google AdSense API (Google Cloud Console)
- Google Sheets API (Google Cloud Console)

---

## 🔄 Workflow Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                    SCHEDULE TRIGGER                              │
│                 Every Month on 5th @ 9 AM                        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                  CALCULATE DATE RANGE                            │
│             Function Node: Previous Month Dates                  │
│          Output: startDate, endDate (YYYY-MM-DD)                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                  PARALLEL DATA FETCH                             │
│  ┌──────────────┬──────────────┬──────────────┬──────────────┐ │
│  │   Plaid      │  FreshBooks  │  FreshBooks  │   AdSense    │ │
│  │ Transactions │   Expenses   │    Income    │   Earnings   │ │
│  └──────┬───────┴──────┬───────┴──────┬───────┴──────┬───────┘ │
└─────────┼──────────────┼──────────────┼──────────────┼─────────┘
          │              │              │              │
          └──────────────┴──────┬───────┴──────────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│                     MERGE NODE                                   │
│          Combine all fetched data streams                        │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              TRANSACTION MATCHING ENGINE                         │
│       Function Node: JavaScript matching algorithm               │
│   • Exact matching (amount + date + vendor)                     │
│   • Fuzzy matching (Levenshtein distance)                       │
│   • Confidence scoring                                           │
│   • Duplicate detection                                          │
│   Output: matched[], unmatched[], needsReview[]                 │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│                    SPLIT FLOW (SWITCH)                           │
│     Route transactions based on matching results                 │
│  ┌──────────────┬──────────────┬──────────────────────────┐    │
│  │   Matched    │  Unmatched   │   Needs Review (>$100)   │    │
│  │  (Update)    │  (Create)    │      (Flag)              │    │
│  └──────┬───────┴──────┬───────┴──────────┬───────────────┘    │
└─────────┼──────────────┼──────────────────┼────────────────────┘
          │              │                  │
          ▼              ▼                  ▼
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│   Update     │  │   Create     │  │  Add to      │
│  FreshBooks  │  │  FreshBooks  │  │ Review List  │
│   Expenses   │  │   Expenses   │  │              │
│ (HTTP PATCH) │  │  (HTTP POST) │  │ (Set Node)   │
└──────┬───────┘  └──────┬───────┘  └──────┬───────┘
       │                 │                  │
       └─────────────────┴──────┬───────────┘
                                │
                                ▼
┌─────────────────────────────────────────────────────────────────┐
│              INCOME RECONCILIATION                               │
│   • Match AdSense earnings to bank deposits                      │
│   • Create "Other Income" entries in FreshBooks                  │
│   • Categorize affiliate payments                                │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              CALCULATE SUMMARY METRICS                           │
│       Function Node: Aggregate totals                            │
│   • Total income by source                                       │
│   • Total expenses by category                                   │
│   • Net income                                                   │
│   • Discrepancy amount (bank vs FreshBooks)                      │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│               IF NODE: Check Discrepancy                         │
│         Is discrepancy > $5 OR > 5%?                             │
│          ┌─────YES────┐     ┌─────NO─────┐                      │
│          ▼             │     ▼            │                      │
│    Add to Review      │   Continue       │                      │
└──────────────────────────────┬────────────────────────────────┘
                               │
                               ▼
┌─────────────────────────────────────────────────────────────────┐
│            UPDATE GOOGLE SHEETS                                  │
│   Google Sheets Node: Append Row                                │
│   • Month/Year                                                   │
│   • Income totals by source                                      │
│   • Expenses by category                                         │
│   • Net income                                                   │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              GENERATE EMAIL REPORT                               │
│       Function Node: Build HTML email                            │
│   • Success summary                                              │
│   • Metrics (transactions, matches, new entries)                 │
│   • Review items (if any)                                        │
│   • Links to FreshBooks & Google Sheet                           │
└────────────────────────┬────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│               SEND EMAIL NODE                                    │
│           Email report to Mike                                   │
└─────────────────────────────────────────────────────────────────┘
                         │
                         ▼
┌─────────────────────────────────────────────────────────────────┐
│              ERROR HANDLING (IF ERRORS)                          │
│   Error Trigger → Log Error → Send Alert Email                  │
└─────────────────────────────────────────────────────────────────┘
```

---

## 🔧 Node-by-Node Implementation Details

### 1. Schedule Trigger Node
**Type:** `n8n-nodes-base.scheduleTrigger`
**Configuration:**
```json
{
  "rule": {
    "interval": [
      {
        "field": "cronExpression",
        "expression": "0 9 5 * *"
      }
    ],
    "timezone": "America/New_York"
  }
}
```

---

### 2. Calculate Date Range (Function Node)
**Type:** `n8n-nodes-base.function`
**Purpose:** Calculate previous month's start and end dates
**JavaScript Code:**
```javascript
// Calculate previous month date range
const now = new Date();
const lastMonth = new Date(now.getFullYear(), now.getMonth() - 1, 1);
const startDate = new Date(lastMonth.getFullYear(), lastMonth.getMonth(), 1);
const endDate = new Date(lastMonth.getFullYear(), lastMonth.getMonth() + 1, 0);

// Format as YYYY-MM-DD
const formatDate = (date) => {
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  return `${year}-${month}-${day}`;
};

return [{
  json: {
    startDate: formatDate(startDate),
    endDate: formatDate(endDate),
    monthName: lastMonth.toLocaleString('en-US', { month: 'long' }),
    year: lastMonth.getFullYear()
  }
}];
```

---

### 3. Fetch Truist Transactions (Plaid Node)
**Type:** `n8n-nodes-plaid` (Community Node)
**Operation:** Get Transactions
**Configuration:**
```json
{
  "resource": "transaction",
  "operation": "getAll",
  "accessToken": "={{$credentials.plaid.accessToken}}",
  "startDate": "={{$node['Calculate Date Range'].json.startDate}}",
  "endDate": "={{$node['Calculate Date Range'].json.endDate}}",
  "options": {
    "includePersonalFinanceCategory": true
  }
}
```

**Output Fields Used:**
- `transaction_id`
- `date`
- `name` (merchant name)
- `amount` (positive for credits, negative for debits)
- `category`
- `pending` (boolean)

---

### 4. Fetch FreshBooks Expenses (HTTP Request Node)
**Type:** `n8n-nodes-base.httpRequest`
**Method:** GET
**URL:** `https://api.freshbooks.com/accounting/account/{{accountId}}/expenses/expenses`
**Authentication:** OAuth2
**Query Parameters:**
```json
{
  "search[date_min]": "={{$node['Calculate Date Range'].json.startDate}}",
  "search[date_max]": "={{$node['Calculate Date Range'].json.endDate}}"
}
```

**Response Fields Used:**
- `expenseid`
- `date`
- `vendor`
- `amount.amount`
- `categoryid`
- `status`

---

### 5. Fetch FreshBooks Income (HTTP Request Node)
**Type:** `n8n-nodes-base.httpRequest`
**Method:** GET
**URL:** `https://api.freshbooks.com/accounting/account/{{accountId}}/other_incomes/other_incomes`
**Authentication:** OAuth2
**Query Parameters:**
```json
{
  "search[date_min]": "={{$node['Calculate Date Range'].json.startDate}}",
  "search[date_max]": "={{$node['Calculate Date Range'].json.endDate}}"
}
```

---

### 6. Fetch Google AdSense Earnings (HTTP Request Node)
**Type:** `n8n-nodes-base.httpRequest`
**Method:** GET
**URL:** `https://adsense.googleapis.com/v2/accounts/{{accountId}}/reports:generate`
**Authentication:** OAuth2 (Google)
**Query Parameters:**
```json
{
  "dateRange": "CUSTOM",
  "startDate.year": "={{$node['Calculate Date Range'].json.startDate.split('-')[0]}}",
  "startDate.month": "={{$node['Calculate Date Range'].json.startDate.split('-')[1]}}",
  "startDate.day": "={{$node['Calculate Date Range'].json.startDate.split('-')[2]}}",
  "endDate.year": "={{$node['Calculate Date Range'].json.endDate.split('-')[0]}}",
  "endDate.month": "={{$node['Calculate Date Range'].json.endDate.split('-')[1]}}",
  "endDate.day": "={{$node['Calculate Date Range'].json.endDate.split('-')[2]}}",
  "metrics": ["ESTIMATED_EARNINGS"]
}
```

---

### 7. Merge Node
**Type:** `n8n-nodes-base.merge`
**Mode:** Merge By Position
**Purpose:** Combine all fetched data streams into single flow

---

### 8. Transaction Matching Engine (Function Node)
**Type:** `n8n-nodes-base.function`
**Purpose:** Intelligent transaction matching with fuzzy logic

**JavaScript Algorithm (Simplified):**
```javascript
// Get data from previous nodes
const bankTransactions = $input.all()[0].json.transactions; // Plaid
const fbExpenses = $input.all()[1].json.expenses; // FreshBooks
const fbIncome = $input.all()[2].json.income; // FreshBooks

// Levenshtein distance for fuzzy matching
function levenshtein(a, b) {
  const matrix = [];
  for (let i = 0; i <= b.length; i++) {
    matrix[i] = [i];
  }
  for (let j = 0; j <= a.length; j++) {
    matrix[0][j] = j;
  }
  for (let i = 1; i <= b.length; i++) {
    for (let j = 1; j <= a.length; j++) {
      if (b.charAt(i - 1) === a.charAt(j - 1)) {
        matrix[i][j] = matrix[i - 1][j - 1];
      } else {
        matrix[i][j] = Math.min(
          matrix[i - 1][j - 1] + 1,
          matrix[i][j - 1] + 1,
          matrix[i - 1][j] + 1
        );
      }
    }
  }
  return matrix[b.length][a.length];
}

// Normalize strings for comparison
function normalize(str) {
  return str.toLowerCase().trim().replace(/[^a-z0-9]/g, '');
}

// Calculate confidence score (0-100)
function calculateConfidence(bankTx, fbExpense) {
  let score = 0;

  // Amount match (40 points)
  const amountDiff = Math.abs(Math.abs(bankTx.amount) - Math.abs(fbExpense.amount));
  if (amountDiff <= 0.01) score += 40;
  else if (amountDiff <= 1) score += 30;
  else if (amountDiff <= 5) score += 20;

  // Date proximity (30 points)
  const dateDiff = Math.abs(new Date(bankTx.date) - new Date(fbExpense.date));
  const daysDiff = dateDiff / (1000 * 60 * 60 * 24);
  if (daysDiff <= 1) score += 30;
  else if (daysDiff <= 3) score += 20;
  else if (daysDiff <= 5) score += 10;

  // Vendor name similarity (30 points)
  const distance = levenshtein(
    normalize(bankTx.name),
    normalize(fbExpense.vendor)
  );
  const maxLength = Math.max(bankTx.name.length, fbExpense.vendor.length);
  const similarity = 1 - (distance / maxLength);
  score += Math.round(similarity * 30);

  return score;
}

// Match transactions
const matched = [];
const unmatched = [];
const needsReview = [];

for (const bankTx of bankTransactions) {
  if (bankTx.pending) continue; // Skip pending
  if (bankTx.amount > 0) continue; // Skip deposits (handle separately)

  let bestMatch = null;
  let bestScore = 0;

  for (const fbExpense of fbExpenses) {
    const score = calculateConfidence(bankTx, fbExpense);
    if (score > bestScore) {
      bestScore = score;
      bestMatch = fbExpense;
    }
  }

  if (bestScore >= 80) {
    // High confidence match
    matched.push({
      bankTransaction: bankTx,
      fbExpense: bestMatch,
      confidence: bestScore,
      action: 'update' // Update if amounts differ
    });
  } else if (bestScore >= 60 && Math.abs(bankTx.amount) < 100) {
    // Medium confidence, low amount - auto-create
    unmatched.push({
      bankTransaction: bankTx,
      confidence: bestScore,
      action: 'create'
    });
  } else {
    // Low confidence or high amount - needs review
    needsReview.push({
      bankTransaction: bankTx,
      confidence: bestScore,
      bestMatch: bestMatch,
      action: 'review'
    });
  }
}

return [{
  json: {
    matched,
    unmatched,
    needsReview,
    summary: {
      totalBankTransactions: bankTransactions.length,
      matchedCount: matched.length,
      unmatchedCount: unmatched.length,
      needsReviewCount: needsReview.length
    }
  }
}];
```

---

### 9. Switch Node (Route Transactions)
**Type:** `n8n-nodes-base.switch`
**Mode:** Rules
**Rules:**
- **Output 1 (Matched):** `={{$json.action}} === 'update'`
- **Output 2 (Unmatched):** `={{$json.action}} === 'create'`
- **Output 3 (Needs Review):** `={{$json.action}} === 'review'`

---

### 10. Update FreshBooks Expenses (HTTP Request Node)
**Type:** `n8n-nodes-base.httpRequest`
**Method:** PUT
**URL:** `https://api.freshbooks.com/accounting/account/{{accountId}}/expenses/expenses/{{$json.fbExpense.expenseid}}`
**Authentication:** OAuth2
**Body:**
```json
{
  "expense": {
    "amount": {
      "amount": "={{Math.abs($json.bankTransaction.amount)}}"
    }
  }
}
```

---

### 11. Create FreshBooks Expenses (HTTP Request Node)
**Type:** `n8n-nodes-base.httpRequest`
**Method:** POST
**URL:** `https://api.freshbooks.com/accounting/account/{{accountId}}/expenses/expenses`
**Authentication:** OAuth2
**Body:**
```json
{
  "expense": {
    "date": "={{$json.bankTransaction.date}}",
    "vendor": "={{$json.bankTransaction.name}}",
    "amount": {
      "amount": "={{Math.abs($json.bankTransaction.amount)}}"
    },
    "categoryid": "={{$json.bankTransaction.category_id}}",
    "notes": "Auto-imported from bank statement"
  }
}
```

---

### 12. Update Google Sheets (Google Sheets Node)
**Type:** `n8n-nodes-base.googleSheets`
**Operation:** Append
**Configuration:**
```json
{
  "operation": "append",
  "sheetId": "={{$credentials.googleSheets.sheetId}}",
  "range": "Sheet1!A:Z",
  "options": {
    "valueInputMode": "USER_ENTERED"
  }
}
```

**Data to Append:**
```json
{
  "values": [[
    "={{$node['Calculate Date Range'].json.monthName}} {{$node['Calculate Date Range'].json.year}}",
    "={{$json.summary.totalIncome}}",
    "={{$json.summary.youtubeIncome}}",
    "={{$json.summary.affiliateIncome}}",
    "={{$json.summary.softwareExpenses}}",
    "={{$json.summary.servicesExpenses}}",
    "={{$json.summary.otherExpenses}}",
    "={{$json.summary.totalExpenses}}",
    "={{$json.summary.netIncome}}",
    "={{$now.toISO()}}"
  ]]
}
```

---

### 13. Generate Email Report (Function Node)
**Type:** `n8n-nodes-base.function`
**Purpose:** Build HTML email report

**JavaScript Code (Simplified):**
```javascript
const summary = $json.summary;
const needsReview = $json.needsReview || [];
const month = $node['Calculate Date Range'].json.monthName;
const year = $node['Calculate Date Range'].json.year;

const html = `
<!DOCTYPE html>
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    .summary { background: #f0f0f0; padding: 20px; }
    .metric { font-size: 24px; font-weight: bold; color: #2c3e50; }
    .section { margin: 20px 0; }
    .alert { background: #fff3cd; padding: 10px; border-left: 4px solid #ffc107; }
  </style>
</head>
<body>
  <h1>✅ Monthly Reconciliation Complete - ${month} ${year}</h1>

  <div class="summary">
    <h2>📊 Summary</h2>
    <p><strong>Total Income:</strong> <span class="metric">$${summary.totalIncome.toFixed(2)}</span></p>
    <ul>
      <li>YouTube: $${summary.youtubeIncome.toFixed(2)}</li>
      <li>Affiliates: $${summary.affiliateIncome.toFixed(2)}</li>
    </ul>

    <p><strong>Total Expenses:</strong> <span class="metric">$${summary.totalExpenses.toFixed(2)}</span></p>
    <p><strong>Net Income:</strong> <span class="metric">$${summary.netIncome.toFixed(2)}</span></p>
  </div>

  <div class="section">
    <h2>✅ Results</h2>
    <ul>
      <li>Truist transactions processed: ${summary.totalBankTransactions}</li>
      <li>FreshBooks expenses matched: ${summary.matchedCount}</li>
      <li>New expenses added: ${summary.unmatchedCount}</li>
      <li>Discrepancies: ${needsReview.length}</li>
    </ul>
  </div>

  ${needsReview.length > 0 ? `
    <div class="alert">
      <h3>📋 Actions Required</h3>
      <p>The following transactions need your review:</p>
      <ul>
        ${needsReview.map(item => `
          <li>$${Math.abs(item.bankTransaction.amount).toFixed(2)} - ${item.bankTransaction.name} (${item.bankTransaction.date})</li>
        `).join('')}
      </ul>
    </div>
  ` : ''}

  <div class="section">
    <h3>🔗 Quick Links</h3>
    <ul>
      <li><a href="https://my.freshbooks.com">View in FreshBooks</a></li>
      <li><a href="https://docs.google.com/spreadsheets/d/YOUR_SHEET_ID">View Accountant Sheet</a></li>
    </ul>
  </div>

  <p><small>Reconciliation completed on: ${new Date().toLocaleString()}</small></p>
</body>
</html>
`;

return [{
  json: {
    subject: `✅ Monthly Reconciliation Complete - ${month} ${year}`,
    html,
    to: 'mike@mikemurphy.co'
  }
}];
```

---

### 14. Send Email Node
**Type:** `n8n-nodes-base.emailSend`
**Configuration:**
```json
{
  "fromEmail": "automation@mikemurphy.co",
  "toEmail": "={{$json.to}}",
  "subject": "={{$json.subject}}",
  "emailFormat": "html",
  "text": "={{$json.html}}"
}
```

---

### 15. Error Handling (Error Trigger + Nodes)
**Error Trigger Type:** `n8n-nodes-base.errorTrigger`
**Connects to:** Error notification workflow

**Error Email Node Configuration:**
```json
{
  "subject": "⚠️ Reconciliation Error - {{$now.format('YYYY-MM-DD')}}",
  "text": "An error occurred during the bank reconciliation workflow:\n\n{{$json.error.message}}\n\nWorkflow: {{$workflow.name}}\nExecution ID: {{$execution.id}}",
  "toEmail": "mike@mikemurphy.co"
}
```

---

## 🔑 Authentication Setup Guide

### 1. Plaid (via Community Node)
**Steps:**
1. Sign up at https://dashboard.plaid.com
2. Create an application
3. Get `client_id` and `secret`
4. Generate Link Token for Truist connection
5. Exchange for Access Token
6. Store Access Token in n8n credentials

**n8n Credential Configuration:**
- Credential Type: Plaid OAuth2 API
- Client ID: [from Plaid dashboard]
- Client Secret: [from Plaid dashboard]
- Access Token: [generated via Link flow]

---

### 2. FreshBooks API
**Steps:**
1. Log into FreshBooks
2. Navigate to Settings → Integrations → API
3. Create OAuth app
4. Get `client_id` and `client_secret`
5. Set redirect URI: `https://your-n8n-instance.com/rest/oauth2-credential/callback`

**n8n Credential Configuration:**
- Credential Type: OAuth2 API
- Grant Type: Authorization Code
- Authorization URL: `https://auth.freshbooks.com/oauth/authorize`
- Access Token URL: `https://api.freshbooks.com/auth/oauth/token`
- Client ID: [from FreshBooks]
- Client Secret: [from FreshBooks]
- Scope: `admin:all`

---

### 3. Google APIs (Sheets + AdSense)
**Steps:**
1. Go to https://console.cloud.google.com
2. Create new project
3. Enable APIs:
   - Google Sheets API
   - Google AdSense Management API
4. Create OAuth 2.0 credentials
5. Add authorized redirect URI: `https://your-n8n-instance.com/rest/oauth2-credential/callback`

**n8n Credential Configuration:**
- Credential Type: Google OAuth2 API
- Client ID: [from Google Console]
- Client Secret: [from Google Console]
- Scope:
  - `https://www.googleapis.com/auth/spreadsheets`
  - `https://www.googleapis.com/auth/adsense.readonly`

---

## 📊 Data Flow Summary

### Input Sources
1. **Plaid (Truist):** 30-50 transactions/month
2. **FreshBooks:** Existing expenses + income entries
3. **Google AdSense:** Monthly earnings total
4. **Date Range:** Calculated previous calendar month

### Processing Steps
1. Fetch all data in parallel
2. Merge data streams
3. Run matching algorithm (exact + fuzzy)
4. Route transactions (matched/unmatched/review)
5. Update/create FreshBooks records
6. Calculate summary metrics
7. Update Google Sheet
8. Generate and send report

### Output Destinations
1. **FreshBooks:** Updated/new expense records
2. **Google Sheets:** New row with monthly summary
3. **Email:** HTML report with links and review items

---

## ⚡ Performance Considerations

**Expected Execution Time:** 2-5 minutes
- Plaid fetch: 10-15 seconds
- FreshBooks API calls: 30-60 seconds
- Matching algorithm: 5-10 seconds
- Updates/Creates: 30-60 seconds
- Report generation: 5 seconds

**Optimization Strategies:**
1. Use parallel execution for data fetching
2. Batch FreshBooks API calls (max 10 per request)
3. Cache category mappings
4. Implement request throttling for rate limits

**Resource Usage:**
- Memory: ~100 MB during execution
- CPU: Minimal (<5% of single core)
- Network: ~2-5 MB data transfer

---

## 🚨 Error Scenarios & Handling

| Error Type | Detection | Response |
|------------|-----------|----------|
| Plaid API timeout | HTTP 504 | Retry 3x, then alert |
| FreshBooks rate limit | HTTP 429 | Wait + retry after reset |
| OAuth token expired | HTTP 401 | Auto-refresh, then retry |
| Invalid transaction data | Validation error | Skip transaction, log, continue |
| Google Sheets permission denied | HTTP 403 | Alert immediately, store backup |
| Network failure | Connection error | Retry 3x, then alert |

**Global Error Handler:**
- Catches all unhandled errors
- Logs to error database (separate Google Sheet)
- Sends alert email with error details
- Workflow marked as failed but doesn't crash

---

## 🧪 Testing Checklist

### Unit Tests (Individual Nodes)
- [ ] Date range calculation returns correct previous month
- [ ] Plaid node fetches transactions successfully
- [ ] FreshBooks API authentication works
- [ ] Matching algorithm produces expected confidence scores
- [ ] Google Sheets append works correctly
- [ ] Email generation produces valid HTML

### Integration Tests (Full Workflow)
- [ ] End-to-end execution with test data
- [ ] Parallel data fetching completes without race conditions
- [ ] Transaction matching handles all edge cases
- [ ] FreshBooks updates/creates work correctly
- [ ] Google Sheet updates correctly
- [ ] Email sent successfully with correct content

### Edge Case Tests
- [ ] Empty month (no transactions)
- [ ] Duplicate transactions
- [ ] Large transaction amounts
- [ ] Transactions on month boundaries
- [ ] Missing AdSense data
- [ ] Partial API failures

---

## 📈 Success Metrics

**Automation KPIs:**
- Time saved: 35-45 minutes → 3-5 minutes (88% reduction)
- Error rate: <1% (with human review for edge cases)
- Match accuracy: >90% auto-matched
- Execution success rate: >95%

**Monitoring:**
- Track execution time trends
- Monitor API call counts vs rate limits
- Log confidence score distributions
- Track manual review frequency

---

## 🔧 Maintenance & Updates

**Monthly:**
- Review match accuracy
- Check for new transaction patterns
- Verify API credential expiration dates

**Quarterly:**
- Update matching algorithm based on manual corrections
- Review and update category mappings
- Optimize performance bottlenecks

**Annually:**
- Update n8n and community nodes
- Review API changes from providers
- Update documentation

---

## 📚 References

**n8n Documentation:**
- [Google Sheets Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)
- [HTTP Request Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.httprequest/)
- [Function Node](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.function/)

**Community Nodes:**
- [n8n-nodes-plaid](https://github.com/jcdotio/n8n-nodes-plaid)
- [Plaid Setup Guide](https://github.com/jcdotio/n8n-nodes-plaid/blob/main/docs/PLAID_SETUP_GUIDE.md)

**API Documentation:**
- [FreshBooks API](https://www.freshbooks.com/api/start)
- [Plaid Transactions API](https://plaid.com/docs/api/products/transactions/)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [Google AdSense API](https://developers.google.com/adsense)

---

**Next Step:** Build the actual n8n workflow JSON file implementing this architecture.

---

**Sources:**
Based on n8n integration research from January 2026:
- [Google Sheets n8n Integration](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)
- [n8n-nodes-plaid Community Node](https://www.npmjs.com/package/n8n-nodes-plaid)
- [Plaid Setup Guide](https://github.com/jcdotio/n8n-nodes-plaid/blob/main/docs/PLAID_SETUP_GUIDE.md)
- [FreshBooks Integration Request](https://community.n8n.io/t/freshbooks-integration/12619)
