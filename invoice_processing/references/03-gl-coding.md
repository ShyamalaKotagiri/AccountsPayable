# 03 — GL Coding & Cost Allocation

## Goal
Assign the correct General Ledger (GL) account code, cost center, department, and project
codes to each invoice line item. Produce a coding block ready to paste into any ERP system.

---

## Coding Elements

| Element | Description | Example |
|---------|-------------|---------|
| GL Account | Expense account code | 6200 (Office Supplies) |
| Cost Center | Operational unit | CC-1042 (Engineering) |
| Department | Business division | DEPT-MKTG |
| Project Code | Billable project or initiative | PRJ-2024-009 |
| Entity / Legal Entity | For multi-entity companies | LE-001 |
| Intercompany Flag | If transaction crosses entities | IC: Yes/No |

The user may not need all elements — ask if unclear, or code what's inferable.

---

## GL Account Assignment

### Step 1: Identify expense category from line item descriptions

| Keyword / Category | Typical GL Account Range |
|-------------------|------------------------|
| Software / SaaS / subscriptions | 6100–6199 |
| Hardware / equipment | 6300–6399 (or capitalize if >threshold) |
| Office supplies / materials | 6200–6299 |
| Professional services / consulting | 7100–7199 |
| Legal / audit / accounting | 7200–7299 |
| Marketing / advertising | 8100–8199 |
| Travel & entertainment | 8200–8299 |
| Utilities | 5100–5199 |
| Rent / facilities | 5000–5099 |
| Freight / shipping | 6400–6499 |
| Insurance | 7300–7399 |
| Repairs & maintenance | 6500–6599 |
| Training / education | 8300–8399 |
| Miscellaneous / other | 9000–9099 |

> **Important**: These are default ranges. If the user has provided a chart of accounts, use
> that instead. Always prefer specificity — use the most specific account that fits.

### Step 2: Assess capitalization requirement
If any line item is a capital asset (equipment, software license > 1 year, leasehold improvement):
- Flag as potential **capital expenditure (CapEx)** rather than OpEx
- Threshold is typically $2,500–$5,000 (ask if unknown)
- CapEx goes to an asset account (1500–1999), not an expense account

### Step 3: Allocate to cost center / department
- Infer from: vendor type, project reference, bill-to department, PO data
- If invoice spans multiple departments, split proportionally
- If unable to determine: leave `[REQUIRES INPUT]` and ask the user

### Step 4: Project code (if applicable)
- Assign if a project reference is present in the invoice or PO
- Note if the cost is billable to a client

---

## Split Coding (Multiple Cost Centers)

If an invoice line item must be split across departments:

| Line | Description | Total | GL Account | CC | Dept | % | Amount |
|------|-------------|-------|-----------|-----|------|---|--------|
| 1 | Cloud hosting | $1,200 | 6150 | CC-1042 | ENGR | 60% | $720 |
| 1 | Cloud hosting | $1,200 | 6150 | CC-2010 | OPS | 40% | $480 |

Splits must sum to 100% and match the original line total exactly.

---

## Output Format

```
## [STAGE 3] GL Coding

### Coding — Line by Line
| Line | Description | Amount | GL Account | Account Name | Cost Center | Dept | Project |
|------|-------------|--------|-----------|-------------|------------|------|---------|
| 1    | [desc]      | $X     | [code]    | [name]      | [CC]       | [D]  | [P]     |

### Tax Coding
- Tax amount: $[X] → GL [code] ([Tax Payable/Expense account])
- Tax type: [Sales Tax / Use Tax / VAT / GST]
- Use tax self-assessment required: [Yes / No / Unknown]

### Total Coding Reconciliation
| GL Account | Account Name | Total Coded |
|-----------|-------------|-------------|
| [code]    | [name]      | $[X]        |
| **TOTAL** |             | **$[Y]**    |

Coded total must match Invoice Total / Balance Due: **$[Z]** ✅ / ❌ Mismatch

### Coding Notes
[Any assumptions made, items requiring clarification, CapEx flags, intercompany notes]
```

---

## Common Coding Questions to Ask If Uncertain

- "Is this a recurring SaaS subscription or a one-time purchase?"
- "Which department or project should bear this cost?"
- "Does this exceed your capitalization threshold?"
- "Is this billable to a client project?"
- "Is this an intercompany transaction?"

Always ask before leaving a code as `[REQUIRES INPUT]` rather than guessing.
