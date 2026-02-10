---
name: n8n-workflow-builder
description: Build n8n automation workflows from ideation to implementation. Use when the user wants to create n8n workflows, automate business processes with n8n, plan workflow architecture, or turn manual processes into n8n automations. Triggers on phrases like "build n8n workflow", "automate with n8n", "n8n automation for", "create workflow", or any mention of n8n automation planning or implementation.
---

# n8n Workflow Builder

Build production-ready n8n automation workflows from initial concept to deployed JSON.

## Workflow Process

### Phase 1: Ideation & Planning

**Always start here for new workflows.**

1. **Use the Ideation Template**
   - Copy `n8n-workflow-ideation-template.md` from references
   - Fill out systematically with user input
   - Document: problem, current workflow, services, data flow, error handling

2. **Key Questions to Ask User**
   - What manual process are you automating?
   - Which services/APIs need to connect?
   - How often should this run? (schedule, webhook, event)
   - What notifications do you need?
   - What requires human review vs full automation?

3. **Technical Feasibility Check**
   - Verify n8n node availability for each service
   - Check if community nodes needed
   - Identify services requiring HTTP Request node (no official integration)
   - Confirm API access and authentication methods

**Output:** Completed ideation document with full workflow specification.

---

### Phase 2: Architecture Design

**After ideation is complete.**

1. **Create Technical Architecture Document**
   - Map data flow: source → transform → destination
   - Design node sequence and connections
   - Specify matching/processing logic algorithms
   - Plan error handling and retry strategies
   - Document authentication requirements

2. **Node Planning**
   - List required node types (triggers, actions, logic, transforms)
   - Specify n8n built-in vs community nodes
   - Design function node JavaScript for custom logic
   - Plan conditional flows (IF/Switch nodes)

3. **Data Transformation Logic**
   - Define matching criteria and confidence scoring
   - Specify calculations and aggregations
   - Map field transformations
   - Design validation rules

**Output:** Technical architecture document with node diagram and implementation details.

---

### Phase 3: Build Workflow JSON

**After architecture is designed.**

1. **Generate n8n Workflow Structure**
   ```json
   {
     "name": "Workflow Name",
     "nodes": [...],
     "connections": {...},
     "active": false,
     "settings": {...}
   }
   ```

2. **Implement Nodes**
   - **Triggers:** Schedule (cron), Webhook, Manual
   - **Service Integrations:** Use appropriate node types
   - **Data Processing:** Function nodes with JavaScript
   - **Conditionals:** IF, Switch nodes for routing
   - **Output:** HTTP Request, service nodes, notifications

3. **Node Structure Template**
   ```json
   {
     "parameters": {
       // Node-specific configuration
     },
     "id": "unique-node-id",
     "name": "Descriptive Node Name",
     "type": "n8n-nodes-base.nodeType",
     "typeVersion": 1,
     "position": [x, y],
     "notes": "What this node does"
   }
   ```

4. **Connect Nodes**
   ```json
   "connections": {
     "Node Name": {
       "main": [
         [
           {
             "node": "Next Node Name",
             "type": "main",
             "index": 0
           }
         ]
       ]
     }
   }
   ```

5. **JavaScript Function Nodes**
   - Access input: `$input.all()`, `$json`
   - Reference other nodes: `$node['Node Name'].json`
   - Return format: `return [{ json: {...} }]`
   - Handle arrays: map/filter/reduce patterns

**Output:** Complete n8n workflow JSON ready to import.

---

### Phase 4: Documentation

**After workflow is built.**

1. **Create README**
   - Use `README-TEMPLATE.md` from references
   - Document: setup, configuration, testing, troubleshooting
   - Include: authentication guide, common issues, maintenance

2. **Key README Sections**
   - Installation & Setup (step-by-step)
   - Configuration Variables (what to customize)
   - Testing Procedures (validation checklist)
   - Error Handling (common issues + solutions)
   - Monitoring & Maintenance (ongoing tasks)

**Output:** Complete README.md for GitHub repository.

---

## n8n Node Reference

### Built-in Nodes (Always Available)

**Triggers:**
- `n8n-nodes-base.scheduleTrigger` - Cron-based scheduling
- `n8n-nodes-base.webhook` - HTTP webhook endpoints
- `n8n-nodes-base.manualTrigger` - Manual execution

**Actions:**
- `n8n-nodes-base.httpRequest` - Universal API calls
- `n8n-nodes-base.googleSheets` - Google Sheets integration
- `n8n-nodes-base.emailSend` - Send emails
- `n8n-nodes-base.function` - JavaScript code execution
- `n8n-nodes-base.code` - JavaScript/Python code

**Logic:**
- `n8n-nodes-base.if` - Conditional branching
- `n8n-nodes-base.switch` - Multi-way branching
- `n8n-nodes-base.merge` - Combine data streams
- `n8n-nodes-base.set` - Set/transform data

### Community Nodes (Require Installation)

**Check availability:** Search npm for `n8n-nodes-[service-name]`

**Installation:**
```bash
# In n8n: Settings → Community Nodes → Install
npm install n8n-nodes-package-name
```

**Known Community Nodes:**
- `n8n-nodes-plaid` - Plaid bank integrations
- Check npm/n8n community for others

### HTTP Request Node (Universal Fallback)

**Use when no official/community node exists.**

**Configuration:**
- Method: GET, POST, PUT, DELETE, PATCH
- URL: Full API endpoint
- Authentication: OAuth2, API Key, Basic Auth
- Headers: Content-Type, Accept, Custom
- Body: JSON, Form Data, Raw

**OAuth2 Setup:**
- Authorization URL: Provider's OAuth authorization endpoint
- Access Token URL: Provider's token endpoint
- Credentials: Client ID + Secret from provider
- Scope: Required permissions

---

## Common Patterns

### Parallel Data Fetching
```
Trigger → Calculate Params → [Split to multiple API nodes] → Merge → Process
```

### Transaction Matching
```javascript
// Fuzzy matching with Levenshtein distance
function levenshtein(a, b) {
  // Implementation in technical architecture examples
}

// Confidence scoring
function calculateConfidence(item1, item2) {
  let score = 0;
  // Amount matching (40 points)
  // Date proximity (30 points)
  // Name similarity (30 points)
  return score;
}
```

### Error Handling
```
Main Flow → [If Error] → Error Trigger → Log + Alert → Optional Retry
```

### Conditional Processing
```
Process Data → Switch Node (by category) → Route 1 / Route 2 / Route 3 → Merge
```

---

## Authentication Setup Guide

### OAuth 2.0 (FreshBooks, Google, etc.)

1. **Provider Setup:**
   - Create OAuth app in service dashboard
   - Get Client ID + Client Secret
   - Set redirect URI: `https://your-n8n.com/rest/oauth2-credential/callback`

2. **n8n Credential:**
   - Credential Type: OAuth2 API
   - Grant Type: Authorization Code
   - Authorization URL: [from provider docs]
   - Access Token URL: [from provider docs]
   - Client ID + Secret: [from step 1]
   - Scope: [required permissions]

3. **Authorize:**
   - Click "Connect" in n8n
   - Complete OAuth flow in browser
   - Token auto-refreshes

### API Key (Simple)

1. **Get API Key:** Service dashboard → API section
2. **n8n Credential:**
   - Type: Header Auth or API Key
   - Name: `Authorization` or `X-API-Key`
   - Value: `Bearer [token]` or `[key]`

---

## Testing Strategy

### Before Production

1. **Use Test Accounts:**
   - Sandbox/dev environments for all services
   - Separate test workflow in n8n
   - Test data that won't affect production

2. **Manual Execution:**
   - Click "Execute Workflow"
   - Watch each node execute
   - Verify data at each step
   - Check output in destination systems

3. **Test Scenarios:**
   - Normal operation (happy path)
   - Missing data
   - Duplicate records
   - API failures (simulate timeout)
   - Edge cases (boundary dates, large amounts)

### Validation Checklist

- [ ] All nodes execute without errors
- [ ] Data correctly fetched from sources
- [ ] Transformations produce expected output
- [ ] Conditionals route correctly
- [ ] Output written to destinations
- [ ] Notifications sent
- [ ] Error handling triggers properly

---

## Common Issues & Solutions

### "OAuth Token Expired"
**Solution:** Refresh credential in n8n, re-authorize

### "Rate Limit Exceeded"
**Solution:** Add Wait nodes, batch requests, implement exponential backoff

### "Data Format Mismatch"
**Solution:** Add validation in Function node, normalize data types

### "Community Node Not Found"
**Solution:** Install via `npm install n8n-nodes-[package]`

### "Workflow Times Out"
**Solution:** Optimize queries, add indexes, split into smaller batches

---

## References

**Ideation Template:** `references/n8n-workflow-ideation-template.md`
Use this to plan ANY n8n workflow from scratch.

**README Template:** `references/README-TEMPLATE.md`
Use this to document completed workflows for GitHub.

**n8n Documentation:**
- Official Docs: https://docs.n8n.io/
- Community Forum: https://community.n8n.io/
- Node Reference: https://docs.n8n.io/integrations/

---

## Workflow Delivery

After building, provide user with:

1. **Workflow JSON file** - Ready to import into n8n
2. **Setup documentation** - Step-by-step configuration guide
3. **Architecture document** - Technical reference
4. **Testing guide** - Validation procedures
5. **README** - Complete documentation for GitHub

All files should be saved to the workspace for easy access.
