# 01 — Invoice Intake & Data Extraction

## Goal
Produce a complete, structured data record from any invoice document. Every field
must be accounted for — missing = noted, not silently skipped.

---

## Standard Fields to Extract

### Header
| Field | Notes |
|-------|-------|
| Invoice Number | May be "Invoice #", "Inv No", "Bill No", "Reference" |
| Invoice Date | Normalize to YYYY-MM-DD |
| Due Date | If absent, derive from terms; else "Not specified" |
| Payment Terms | Net 30, 2/10 Net 30, Due on Receipt, etc. |
| PO Number | May be absent — note explicitly |
| Contract / Project Ref | If present |
| Currency | Default to USD if not stated; flag if ambiguous |

### Vendor
| Field | Notes |
|-------|-------|
| Vendor Name | Full legal name |
| Vendor ID / Vendor Code | From invoice if shown |
| Vendor Address | Full address |
| Vendor Tax ID | EIN / VAT / GST |
| Remittance Contact | Email or name |

### Bill-To (Buyer)
| Field | Notes |
|-------|-------|
| Company Name | |
| Address | |
| AP Contact / Dept | If shown |

### Line Items
Extract as a table:
| # | Description | Qty | Unit | Unit Price | Discount | Line Total |
|---|-------------|-----|------|-----------|----------|------------|

If line items are missing (lump sum), note: "No line item detail — lump sum invoice."

### Totals
| Field | Value |
|-------|-------|
| Subtotal | Before tax/discount |
| Discount | If any |
| Tax (type + rate) | e.g., "Sales Tax 8.5%" |
| Shipping / Freight | |
| Other Charges | |
| **Invoice Total** | Grand total |
| Amount Paid | If partial payment noted |
| **Balance Due** | Total − Paid |

### Payment Instructions
- Bank name, account, routing, SWIFT/IBAN
- Check payable to
- Remittance email
- Accepted payment methods

---

## Output Format

```
## [STAGE 1] Extracted Invoice Data

**Invoice #**: [value]
**Invoice Date**: [YYYY-MM-DD]
**Due Date**: [YYYY-MM-DD | derived from terms | Not specified]
**Payment Terms**: [value]
**Currency**: [value]
**PO Reference**: [value | None]

### Vendor
- Name: [value]
- Address: [value]
- Tax ID: [value | Not shown]
- Remittance: [email/contact | Not shown]

### Bill To
- Company: [value]
- Address: [value]

### Line Items
| # | Description | Qty | Unit Price | Total |
|---|-------------|-----|-----------|-------|
| 1 | ...         | ... | ...        | ...   |

### Totals
- Subtotal: [value]
- Discount: [value | None]
- Tax ([type] [rate]%): [value]
- Shipping: [value | None]
- **Invoice Total**: [value]
- Amount Paid: [value | None]
- **Balance Due**: [value]

### Payment Instructions
[Details or "Not provided"]

### Extraction Notes
[Any [?] fields, low-confidence reads, or missing sections]
```

---

## Handling Edge Cases

| Situation | Action |
|-----------|--------|
| Scanned / low-res image | Read L→R, T→B; flag uncertain chars with `[?]` |
| Foreign language invoice | Extract numbers as-is; note language; ask if translation needed |
| Multiple currencies | Extract each as-is; do not convert; flag for review |
| Credit memo | Flag as CREDIT MEMO; show amounts as negative |
| No line items (lump sum) | Note explicitly; extract total only |
| Recurring invoice | Note recurrence frequency if shown |

---

## Batch Extraction (Multiple Invoices)

1. Extract each invoice individually using the format above.
2. After all extractions, produce a **batch summary table**:

| # | Invoice # | Vendor | Invoice Date | Due Date | Total Due | PO # | Flags |
|---|-----------|--------|-------------|----------|-----------|------|-------|

3. Sort by Due Date ascending (soonest first).
4. Highlight past-due invoices with ⚠️.
5. Count: total invoices, total amount, flagged count.
