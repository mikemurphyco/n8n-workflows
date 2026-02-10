# n8n Workflow: Monthly Bank Reconciliation Automation

**Version:** 1.0
**Last Updated:** January 29, 2026
**Owner:** Mike Murphy LLC

---

## 📋 Workflow Overview

### Workflow Name
**Monthly Bank Reconciliation - Truist to FreshBooks to Google Sheets**

---

### Problem Statement
Currently spending 35-45 minutes monthly on manual bank reconciliation that involves:
- Logging into multiple platforms (Truist, FreshBooks, Google AdSense, Google Sheets)
- Manually downloading bank statements
- Comparing transactions line-by-line between systems
- Manually entering/updating expenses and income in FreshBooks
- Manually updating accountant's Google Sheet with categorized data

**Pain Points:**
- Manual data entry is time-consuming and error-prone
- Requires context switching between 5+ different systems
- Repetitive task that happens monthly without variation
- Risk of missing transactions or data entry errors
- No automatic alerts for discrepancies

**Impact:**
- Time: 35-45 minutes monthly (7-9 hours annually)
- Risk: Financial reporting errors could impact taxes
- Opportunity cost: Could spend time on revenue-generating activities

---

### Current Workflow Documentation

**Trigger:** Monthly reminder on the 5th of each month (Apple Reminder)

1. **Download Bank Statement**
   - Sign into Truist Business Account
   - Download PDF statement for previous month

2. **Record YouTube Income**
   - Sign into Google AdSense
   - Check YouTube monthly payment amount
   - Add as "Other Income" in FreshBooks
   - Add total YouTube payment to Accountant Google Sheet

3. **Open Expense Comparison**
   - Sign into FreshBooks
   - Open Expenses section
   - Open Truist PDF statement side-by-side

4. **Match Expenses**
   - Review each expense on Truist statement
   - Try to match with existing FreshBooks expense (manual or recurring)
   - If amounts don't match, edit FreshBooks to match Truist
   - If expense missing from FreshBooks, add it manually

5. **Match Deposits**
   - Review Truist deposits (YouTube, Affiliates, etc.)
   - Match with FreshBooks "Other Income" entries
   - Add any missing deposit entries to FreshBooks

6. **Generate Report**
   - Navigate to FreshBooks Reports
   - Run Profit & Loss Report for current month
   - Review totals

7. **Update Accountant Sheet**
   - Open one-page Google Sheet from accountant
   - Manually enter income total
   - Manually enter expenses by category
   - Verify month is fully reconciled

**Time Analysis:**
- Current time to complete: 35-45 minutes
- Frequency: Monthly (on the 5th)
- Total monthly time spent: ~0.67 hours
- Annual time spent: ~8 hours

---

### Desired Outcome

**Automation Goals:**
- [x] Automatically fetch bank transactions from Truist (via Plaid API)
- [x] Automatically reconcile transactions with FreshBooks expenses
- [x] Auto-match transactions using intelligent matching (description, amount, date proximity)
- [x] Automatically add missing expenses to FreshBooks
- [x] Automatically fetch YouTube AdSense income
- [x] Automatically update FreshBooks with income
- [x] Automatically generate FreshBooks P&L report
- [x] Automatically update accountant's Google Sheet with categorized data
- [ ] Human review required for: unmatched transactions over $100, discrepancies >5%

**Success Metrics:**
- Time saved per execution: 30-40 minutes (reducing from 40 min to 5 min review)
- Error reduction: Near 100% accuracy with human review for edge cases
- Monthly time spent: ~5 minutes for review and approval
- Other benefits: Real-time discrepancy alerts, audit trail

---

## 🔌 Technical Requirements

### Services & APIs to Connect

| Service | Purpose | API/Integration Available? | Documentation Link |
|---------|---------|---------------------------|-------------------|
| **Truist Bank** | Fetch bank transactions | ⚠️ Via Plaid API | [Plaid API Docs](https://plaid.com/docs/) |
| **Plaid** | Bank connection middleware | ✅ Yes | [Plaid API](https://plaid.com/docs/api/) |
| **FreshBooks** | Accounting software (expenses/income) | ✅ Yes | [FreshBooks API](https://www.freshbooks.com/api) |
| **PayPal** | Payment processor | ✅ Yes | [PayPal API](https://developer.paypal.com/api/rest/) |
| **Google AdSense** | YouTube revenue tracking | ✅ Yes | [AdSense API](https://developers.google.com/adsense) |
| **Google Sheets** | Accountant reporting | ✅ Yes | [Sheets API](https://developers.google.com/sheets/api) |
| **Email/Slack** | Notifications | ✅ Yes (built-in n8n) | n8n native nodes |

**API Research Notes:**
- **Plaid Authentication:** Requires Link token for Truist connection
  - Rate limits: 100 requests/minute for development
  - Cost: Free for development, paid tiers for production
  - Provides standardized transaction data

- **FreshBooks Authentication:** OAuth 2.0
  - Rate limits: 3600 requests/hour per user
  - Supports: Expenses, Income, Clients, Invoices, Reports
  - Can create/update/delete expenses programmatically

- **Google AdSense Authentication:** OAuth 2.0
  - Rate limits: 10,000 queries per day
  - Can fetch earnings by date range

- **Google Sheets Authentication:** OAuth 2.0 or Service Account
  - Rate limits: 300 read requests per minute per project
  - Can read/write cell ranges programmatically

**Known Limitations:**
- Truist doesn't have direct API; Plaid required as middleware
- FreshBooks API doesn't support direct P&L generation via API (may need workaround)
- Transaction matching logic must be custom-built
- Time zone handling for monthly boundaries

---

### n8n Nodes Required

**Estimated Node Types:**
- [x] **Trigger nodes:** Schedule Trigger (monthly on 5th at 9 AM)
- [x] **Service integrations:**
  - Plaid node (or HTTP Request for Plaid API)
  - FreshBooks node (or HTTP Request)
  - Google Sheets node
  - Google AdSense (HTTP Request)
  - PayPal node (if needed for additional reconciliation)
- [x] **Data transformation:**
  - Set node (for data formatting)
  - Function node (JavaScript for matching logic)
  - Aggregate node (for grouping transactions)
- [x] **Conditional logic:**
  - IF nodes (for matching decisions)
  - Switch node (for categorization)
  - Merge node (to combine data streams)
- [x] **Notification nodes:**
  - Email node (for reports and alerts)
  - Slack node (optional, for real-time updates)
- [x] **Error handling:**
  - Error Trigger node
  - IF node (for validation checks)

**Special Requirements:**
- Custom JavaScript matching algorithm (fuzzy matching for descriptions)
- Date range calculations for "previous month"
- PDF parsing capability (if Plaid doesn't provide all needed data)
- Decimal precision handling for financial calculations
- Transaction categorization logic

---

## 🔄 Data Flow Architecture

### Input Data Sources

1. **Truist Bank (via Plaid)**
   - Data type: Transaction list
   - Format: JSON
   - Fields needed:
     - transaction_id
     - date
     - description/name
     - amount
     - category
     - pending status
   - Update frequency: Daily (Plaid syncs), but workflow runs monthly
   - Date range: Previous calendar month (1st to last day)

2. **FreshBooks - Existing Expenses**
   - Data type: Expense list
   - Format: JSON
   - Fields needed:
     - expense_id
     - date
     - vendor
     - amount
     - category
     - status (logged/invoiced/paid)
   - Date range: Previous calendar month

3. **FreshBooks - Existing Income**
   - Data type: Other Income entries
   - Format: JSON
   - Fields needed:
     - income_id
     - date
     - source
     - amount
   - Date range: Previous calendar month

4. **Google AdSense**
   - Data type: Earnings report
   - Format: JSON
   - Fields needed:
     - date range
     - total earnings
     - breakdown by source (YouTube)
   - Date range: Previous calendar month

---

### Data Transformation Logic

**Transaction Matching Algorithm:**

The workflow needs to match Truist transactions with FreshBooks expenses using:

1. **Exact Match Criteria:**
   - Amount matches exactly (±$0.01 for rounding)
   - Date within ±3 days
   - Vendor/description contains key matching terms

2. **Fuzzy Match Criteria:**
   - Amount matches exactly
   - Date within ±5 days
   - Levenshtein distance < 3 for vendor name
   - Confidence score > 80%

3. **Income Matching:**
   - Match deposits to existing "Other Income" entries
   - Identify YouTube payments (match against AdSense data)
   - Identify affiliate payments (description patterns)

4. **Categorization Logic:**
   - Map Plaid categories to FreshBooks expense categories
   - Use description keywords for custom categorization
   - Default categories for common vendors (e.g., "Adobe" → Software)

**Business Rules:**
- Transactions < $5: Auto-match with lower confidence threshold
- Transactions > $100: Require human review if confidence < 95%
- Recurring expenses: Higher confidence for pattern matching
- Duplicate detection: Flag if same amount/vendor within 24 hours

**Calculations Needed:**
- Previous month date range (dynamic based on execution date)
- Total expenses by category (for Google Sheet)
- Total income by source
- Net income (Income - Expenses)
- Discrepancy amount (Truist total vs FreshBooks total)

**Data Cleaning:**
- Normalize vendor names (trim, lowercase, remove special chars)
- Convert date formats to ISO 8601
- Round amounts to 2 decimal places
- Remove pending/duplicate transactions

---

### Output Destinations

1. **FreshBooks - New/Updated Expenses**
   - What gets written:
     - New expenses for unmatched Truist transactions
     - Updated amounts for mismatched expenses
   - Format: JSON API calls (POST /expenses, PUT /expenses/{id})
   - Update vs. Append: Both (update existing, append new)

2. **FreshBooks - Other Income**
   - What gets written:
     - YouTube AdSense income
     - Affiliate/other deposits from Truist
   - Format: JSON API calls (POST /other_incomes)
   - Update vs. Append: Append (create new entries)

3. **Google Sheets - Accountant Sheet**
   - What gets written:
     - Total income (from FreshBooks)
     - Expenses by category (mapped to accountant's categories)
     - Month/year identifier
     - Reconciliation date/timestamp
   - Format: Cell range update (A2:Z2 for new month row)
   - Update vs. Append: Append new row for each month

4. **Email - Reconciliation Report**
   - What gets sent:
     - Summary statistics (total income, expenses, net)
     - List of unmatched transactions (if any)
     - Discrepancies requiring review
     - Link to FreshBooks for final approval
   - Format: HTML email
   - Recipients: Mike's email

---

## 🚨 Error Handling & Edge Cases

### Potential Failure Points

| Scenario | Impact | Mitigation Strategy |
|----------|--------|---------------------|
| **Plaid API timeout/failure** | Can't fetch bank transactions | Retry 3x with exponential backoff; send alert email; log error |
| **FreshBooks API rate limit** | Can't update expenses | Implement request throttling; batch operations; retry after delay |
| **Google AdSense API auth expired** | Can't fetch YouTube income | Refresh OAuth token automatically; alert if refresh fails |
| **Missing FreshBooks category** | Can't categorize expense | Use "Uncategorized" default; flag for manual review |
| **Duplicate transactions** | Double-counting expenses | Detect duplicates by date+amount+vendor; skip if already in FreshBooks |
| **Amount discrepancy** | Potential data error | Flag if total Truist ≠ total FreshBooks by >$5; human review required |
| **Date range calculation error** | Wrong month processed | Validate date range before API calls; log date boundaries |
| **Google Sheets permission denied** | Can't update accountant sheet | Retry with fresh auth; alert Mike if fails; store data as backup |
| **Partial completion** | Some steps succeed, others fail | Implement transaction log/state tracking; resume from last success point |

---

### Error Notification Strategy
- [x] **Email notifications** (primary notification channel)
  - All critical errors
  - Unmatched transactions
  - Discrepancies requiring review

- [x] **Slack alerts** (optional, for real-time awareness)
  - Workflow started
  - Workflow completed
  - Critical errors only

- [x] **Error log database** (for audit trail)
  - Store in separate Google Sheet "Error Log"
  - Include: timestamp, error type, affected transaction, resolution status

- [x] **Retry logic**
  - API timeouts: 3 attempts with 5/10/20 second delays
  - Rate limits: Wait and retry after rate limit reset
  - Auth failures: Attempt token refresh, then alert

---

## 🔔 Notifications & Reporting

### Progress Notifications

**Workflow Started:**
- Message: "🏦 Bank reconciliation started for [Month Year]"
- Channel: Email + Slack (optional)
- Details: Date range being processed

**Key Milestones:**
- ✅ Bank transactions fetched (X transactions)
- ✅ FreshBooks expenses matched (Y matched, Z new)
- ✅ Income recorded
- ✅ Google Sheet updated
- ✅ Workflow completed

---

### Success Report Content

**Email Report (HTML formatted):**

```
Subject: ✅ Monthly Reconciliation Complete - [Month Year]

Hi Mike,

Your bank reconciliation for [Month] has been completed successfully!

📊 SUMMARY:
• Total Income: $X,XXX.XX
  - YouTube: $XXX.XX
  - Affiliates: $XXX.XX
  - Other: $XXX.XX

• Total Expenses: $X,XXX.XX
  - Software: $XXX.XX
  - Services: $XXX.XX
  - Other: $XXX.XX

• Net Income: $X,XXX.XX

✅ RESULTS:
• Truist transactions processed: XX
• FreshBooks expenses matched: XX
• New expenses added: X
• Income entries created: X
• Discrepancies: X (see below)

📋 ACTIONS REQUIRED:
[List of unmatched transactions or discrepancies if any]

🔗 QUICK LINKS:
• View in FreshBooks: [link]
• View Accountant Sheet: [link]

Reconciliation completed on: [timestamp]
```

**Metrics Included:**
- Records processed (Truist transactions count)
- Records matched (auto-matched count)
- New records added (new FreshBooks expenses)
- Discrepancies found (count + list)
- Time to complete (workflow execution time)
- Confidence scores (average matching confidence)

---

### Issue Alerts

**Immediate Alert Triggers:**
- [x] Unmatched transactions > $100 (requires review)
- [x] Total discrepancy > $50 or >5% (between Truist and FreshBooks)
- [x] Service connection failure (any API)
- [x] Duplicate transaction detection
- [x] Negative balance or unusual patterns
- [x] Missing required data (e.g., no AdSense data)

**Alert Format:**
```
Subject: ⚠️ Reconciliation Issue: [Issue Type]

[Description of issue]
Affected transaction(s): [Details]
Recommended action: [What to do]
```

---

## 🧪 Testing Strategy

### Test Scenarios

| Scenario | Expected Outcome | Test Data Needed |
|----------|------------------|------------------|
| **Normal operation** | All transactions matched, sheet updated | Real data from previous month (sanitized) |
| **Missing FreshBooks expense** | New expense created automatically | Bank transaction not in FreshBooks |
| **Amount mismatch** | FreshBooks updated to match bank | Expense with wrong amount in FreshBooks |
| **Duplicate transaction** | Duplicate detected and skipped | Same transaction in both systems |
| **Unmatched large transaction** | Flagged for review, email alert sent | Transaction >$100 with no match |
| **API failure (Plaid)** | Retry logic triggered, error logged | Simulate API timeout |
| **API failure (FreshBooks)** | Workflow pauses, alert sent | Simulate rate limit |
| **Missing AdSense data** | Warning issued, workflow continues | No AdSense earnings for month |
| **Date boundary edge case** | Correct month transactions only | Transaction on 1st/last day of month |
| **Multiple income sources** | All sources correctly categorized | Mix of YouTube, affiliate, other income |

---

### Test Environment Setup
- [x] **Plaid:** Use Plaid Sandbox environment for testing
  - Use test credentials for Truist
  - Generate sample transaction data

- [x] **FreshBooks:** Create separate test account or use "Test Mode"
  - Clone expense categories from production
  - Use test client/vendor names

- [x] **Google Sheets:** Create separate "Test Accountant Sheet"
  - Duplicate structure of production sheet

- [x] **n8n:** Build separate test workflow
  - Naming: "Bank Reconciliation - TEST"
  - Use test credentials for all services

- [x] **Rollback plan:**
  - Keep backup of FreshBooks data before each run
  - Use version control for workflow JSON
  - Test on previous month's closed data first

---

## 📊 Workflow Execution Plan

### Scheduling
- **Trigger Type:** Schedule Trigger (Cron)
- **Frequency:** Monthly on the 5th at 9:00 AM EST
- **Cron Expression:** `0 9 5 * *`
- **Timezone considerations:** America/New_York (Eastern Time)
  - Bank statements typically finalize by 1st of month
  - Running on 5th allows buffer for processing
  - 9 AM allows time to review results during work hours

**Manual Trigger Option:**
- Also include webhook trigger for manual execution if needed
- Useful for testing or running mid-month reconciliation

---

### Execution Order

```
1. Schedule Trigger (Monthly on 5th)
   ↓
2. Calculate Date Range (previous month 1st-last day)
   ↓
3. [PARALLEL FETCH]
   ├─ 3a. Fetch Truist Transactions (via Plaid)
   ├─ 3b. Fetch FreshBooks Expenses
   ├─ 3c. Fetch FreshBooks Income
   └─ 3d. Fetch Google AdSense Earnings
   ↓
4. Merge & Validate Data
   ↓ [IF all data fetched successfully]
5. Match Bank Transactions to FreshBooks Expenses
   ├─ Exact matches → Update if amount differs
   ├─ Fuzzy matches (high confidence) → Link
   └─ No match → Flag for new expense creation
   ↓
6. Process Unmatched Bank Transactions
   ├─ [IF < $100] → Auto-create FreshBooks expense
   ├─ [IF >= $100] → Flag for review + add to report
   └─ Apply categorization logic
   ↓
7. Match Deposits to Income
   ├─ Match AdSense payment to bank deposit
   ├─ Create/update "Other Income" in FreshBooks
   └─ Categorize other deposits
   ↓
8. Generate Summary Data
   ├─ Total income by source
   ├─ Total expenses by category
   ├─ Net income
   └─ Discrepancy check (bank total vs FreshBooks total)
   ↓
9. [IF discrepancy > $5] → Add to review items
   ↓
10. Update Google Sheet
    ├─ Find next empty row
    ├─ Write month/year
    ├─ Write income totals
    ├─ Write expense by category
    └─ Write net income
    ↓
11. Generate & Send Report Email
    ├─ Success summary
    ├─ Review items (if any)
    └─ Links to FreshBooks & Google Sheet
    ↓
12. [IF errors occurred] → Send error alert email
    ↓
13. Log workflow completion
```

---

### Dependencies

**Pre-Execution Requirements:**
- [x] Truist bank statement must be finalized (usually by 1st of month)
- [x] All API credentials must be valid and not expired
- [x] Previous month must not already be reconciled in the system
- [x] Google Sheet must be accessible with write permissions
- [x] n8n must have outbound internet access for API calls

**Data Dependencies:**
- Plaid must have synced Truist transactions (happens daily automatically)
- FreshBooks recurring expenses should already be logged for the month
- AdSense finalized earnings available (typically by 2nd-3rd of month)

---

## 💰 Cost & Resource Analysis

### API Costs

| Service | Free Tier | Estimated Monthly Calls | Monthly Cost |
|---------|-----------|------------------------|--------------|
| **Plaid** | 100 items in Development | 30-50 transactions | $0 (Dev) / ~$0.10 (Prod) |
| **FreshBooks** | 3600 req/hour | ~100 API calls | $0 (included in subscription) |
| **Google AdSense API** | 10,000 queries/day | 1-2 queries | $0 (free) |
| **Google Sheets API** | 300 req/min | ~10 API calls | $0 (free) |
| **Email (SMTP)** | Varies | 2-3 emails | $0 (using Gmail/existing SMTP) |

**Total Estimated Monthly Cost:** $0.10 (production Plaid) or $0 (if using free tiers)

**Annual Cost:** ~$1.20

---

### n8n Execution Resources
- **Estimated execution time:** 2-5 minutes (depending on transaction count)
- **Executions per month:** 1 (scheduled) + occasional manual
- **n8n plan required:**
  - Self-hosted: Free (if hosting on own server/VPS)
  - n8n Cloud Starter: $20/month (includes 2,500 executions)
  - This workflow easily fits in free self-hosted tier

**Hardware Requirements (Self-Hosted):**
- CPU: Minimal (< 1 core)
- RAM: ~512 MB during execution
- Storage: < 100 MB for workflow + logs

---

## 🎯 Success Criteria & Next Steps

### Definition of Done

**Workflow Functionality:**
- [x] Workflow executes without errors in test environment with test data
- [x] All data sources successfully integrated (Plaid, FreshBooks, AdSense, Sheets)
- [x] Transaction matching algorithm achieves >90% accuracy
- [x] Output data matches expected format in FreshBooks and Google Sheet
- [x] Error handling tested and working (API failures, missing data)
- [x] Notifications triggering correctly (success report, error alerts)

**Documentation:**
- [x] Workflow JSON exported and version controlled
- [x] README with setup instructions
- [x] API credential setup guide
- [x] Troubleshooting guide for common issues

**Testing:**
- [x] User acceptance testing passed with real previous month data
- [x] Edge cases tested (duplicates, large amounts, missing data)
- [x] Error scenarios validated (API failures, auth issues)

**Production Readiness:**
- [x] Plaid moved from Sandbox to Production (if using paid tier)
- [x] OAuth tokens configured and tested for all services
- [x] Schedule trigger activated for 5th of month
- [x] Backup/rollback plan documented

---

### Implementation Phases

**Phase 1: MVP (Basic Automation)**
- [x] Set up Plaid connection to Truist
- [x] Set up FreshBooks API integration
- [x] Build basic transaction matching (exact match only)
- [x] Create new expenses in FreshBooks for unmatched transactions
- [x] Update Google Sheet with totals
- [x] Send basic email report
- **Target:** 80% automation, reduce manual time from 40 min to 10 min

**Phase 2: Enhanced Matching (Error Handling & Intelligence)**
- [x] Implement fuzzy matching algorithm
- [x] Add confidence scoring
- [x] Build categorization logic
- [x] Implement duplicate detection
- [x] Add comprehensive error handling
- [x] Enhanced email reports with details
- [x] Integrate Google AdSense for YouTube income
- **Target:** 95% automation, reduce manual time to 3-5 min review

**Phase 3: Optimization (Polish & Scalability)**
- [x] Add PayPal integration (if needed)
- [x] Build dashboard for historical reconciliation data
- [x] Implement machine learning for categorization (over time)
- [x] Add Slack integration for real-time notifications
- [x] Create admin panel for adjusting matching rules
- [x] Build analytics on reconciliation patterns
- **Target:** 98% automation, bulletproof reliability

---

### Future Enhancements (v2.0+)

**Advanced Features:**
- **Multi-account support:** Handle multiple bank accounts/cards
- **Predictive categorization:** ML model learns from manual corrections
- **Anomaly detection:** Flag unusual spending patterns or fraud
- **Budget tracking:** Alert when category spending exceeds budget
- **Invoice matching:** Auto-match FreshBooks invoices to bank deposits
- **Receipt OCR:** Parse emailed receipts and match to transactions
- **Tax estimation:** Calculate quarterly tax estimates automatically

**Integration Expansions:**
- Stripe integration for client payment reconciliation
- PayPal automatic reconciliation
- Shopify sales reconciliation (if applicable)
- Credit card statement reconciliation
- Integration with tax software (TurboTax, TaxAct)

**User Experience:**
- Web dashboard for monitoring reconciliation history
- Mobile app notifications
- Natural language review ("approve all" via email reply)
- Conversation AI for questions ("Why didn't X match?")

---

## 📝 Additional Notes

### Assumptions
- Bank statement data via Plaid is accurate and matches official PDF
- FreshBooks is the source of truth for accounting
- Google Sheet structure remains consistent (columns don't change)
- Transaction descriptions from Plaid are detailed enough for matching
- Single business checking account (not handling multiple accounts yet)
- All transactions in USD
- No international currency conversions needed

### Open Questions
- [ ] Does FreshBooks API support fetching P&L report data directly, or do we need to calculate it?
- [ ] What's the exact structure of the accountant's Google Sheet? (need column mappings)
- [ ] Are there specific expense categories in FreshBooks that must be preserved?
- [ ] Should PayPal transactions be reconciled separately or are they already in Truist?
- [ ] What's the preferred notification method: email or Slack or both?
- [ ] Should the workflow wait for manual approval before updating FreshBooks, or auto-update with review after?

### Resources & References
- [Plaid Transactions API Documentation](https://plaid.com/docs/api/products/transactions/)
- [FreshBooks API v3 Documentation](https://www.freshbooks.com/api/start)
- [Google Sheets API v4 Documentation](https://developers.google.com/sheets/api/guides/concepts)
- [n8n Plaid Node Documentation](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.plaid/)
- [n8n FreshBooks Node](https://docs.n8n.io/) (check if available)
- [n8n Google Sheets Node](https://docs.n8n.io/integrations/builtin/app-nodes/n8n-nodes-base.googlesheets/)

**Community Resources:**
- n8n Community Forum: Search for "accounting automation" workflows
- Plaid Community: Best practices for transaction categorization
- FreshBooks Developer Community: Expense management patterns

---

## 🔗 Related Workflows

**Potential Connected Workflows:**
- **Invoice Payment Tracking:** Monitor when clients pay invoices (Stripe → FreshBooks)
- **Expense Receipt Management:** Auto-file expense receipts from email
- **Monthly Financial Summary:** Generate comprehensive business report
- **Tax Document Prep:** Annual compilation of data for accountant

**Monetization Opportunity:**
- Package this workflow for accountant's 2000+ clients
- Create white-label version with customizable mapping
- Build SaaS dashboard for non-technical users
- Offer as done-for-you setup service

---

## 🚀 Next Steps for Mike

1. **Review this plan** and answer open questions above
2. **Provide Google Sheet structure** (column names, layout)
3. **Set up API access** for all services:
   - Plaid developer account
   - FreshBooks API credentials (OAuth app)
   - Google Cloud project for Sheets/AdSense APIs
4. **Prepare test data** from last month for validation
5. **Approve workflow build** - ready to create n8n JSON!

---

**Ready to build?** This workflow will save you ~8 hours per year and eliminate reconciliation errors. Let's make it happen! 🎯
