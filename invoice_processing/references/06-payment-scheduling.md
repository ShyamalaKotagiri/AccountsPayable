# 06 — Payment Scheduling

## Goal
Determine the optimal payment date, check for early payment discounts, flag late payment
risk, and produce a payment schedule entry the user can add to their payment run.

---

## Payment Date Calculation

### Step 1: Establish the due date
- If due date is explicit on invoice → use it
- If payment terms given → calculate from invoice date:
  | Terms | Formula |
  |-------|---------|
  | Net 30 | Invoice Date + 30 days |
  | Net 60 | Invoice Date + 60 days |
  | Net 15 | Invoice Date + 15 days |
  | Due on Receipt | Invoice Date (pay immediately) |
  | EOM (End of Month) | Last day of invoice month |
  | Net 30 EOM | Last day of invoice month + 30 days |
  | 2/10 Net 30 | Discount deadline: Invoice Date + 10 days; Final due: Invoice Date + 30 days |

### Step 2: Check early payment discount
Format: `X/Y Net Z` means "X% discount if paid within Y days; otherwise net due in Z days"

| Terms | Discount Rate | Discount Deadline | Regular Due Date |
|-------|-------------|------------------|-----------------|
| 2/10 Net 30 | 2% | Invoice Date + 10 | Invoice Date + 30 |
| 1/15 Net 45 | 1% | Invoice Date + 15 | Invoice Date + 45 |

**Early pay discount value**: Invoice Total × Discount Rate

**Annualized return** (to assess if it's worth taking):
= (Discount % ÷ (1 − Discount %)) × (365 ÷ Days Between Discount Date and Due Date)

> Example: 2/10 Net 30 → (2% ÷ 98%) × (365 ÷ 20) = **37.2% annualized** — almost always worth taking.

Flag if early pay discount is available and calculate the savings.

### Step 3: Late payment risk
- Flag if invoice is already past due → calculate days overdue
- Check if late fee terms are on the invoice (e.g., "1.5% per month after due date")
- Calculate accrued late fee if applicable
- Note any relationship risk for critical vendors

---

## Payment Method Selection

| Payment Method | Use When |
|---------------|---------|
| ACH / EFT | Domestic vendor, bank details on file — preferred for cost |
| Wire Transfer | International vendor, large amount, urgent |
| Check | Vendor requires check; no bank details on file |
| Credit Card | Small amounts, card accepted, float benefit |
| Virtual Card | If AP automation supports it |

---

## Payment Run Integration

If multiple invoices are being scheduled, produce a payment run batch:

| Vendor | Invoice # | Invoice Total | Discount Available | Discount Deadline | Pay By Date | Pay Amount | Method |
|--------|-----------|-------------|------------------|------------------|-------------|-----------|--------|
| [Vendor] | [#] | $[X] | $[D] | [Date] | [Date] | $[X-D] | ACH |

Sort by: (1) Past due first, (2) Discount deadline, (3) Regular due date.

---

## Output Format

```
## [STAGE 6] Payment Schedule

**Invoice Total**: $[X]
**Invoice Date**: [YYYY-MM-DD]
**Payment Terms**: [Terms]
**Due Date**: [YYYY-MM-DD]

### Early Pay Discount
[If available:]
- Discount: [Rate]% = $[Savings]
- Discount deadline: [YYYY-MM-DD]
- Annualized return: [X]% — [RECOMMENDED / MARGINAL / NOT RECOMMENDED]
- Pay $[Discounted Amount] by [discount deadline] to capture discount

[If not available: "No early pay discount available."]

### Recommended Payment
- **Pay Amount**: $[X]
- **Pay By**: [YYYY-MM-DD] [(X days from today)]
- **Payment Method**: [ACH / Wire / Check]
- **Payee**: [Vendor Name]
- **Remittance Reference**: Invoice #[X]

### Late Payment Status
[If overdue:]
⚠️ OVERDUE — [X] days past due as of today
  - Accrued late fee (if applicable): $[X]
  - Recommended action: Pay immediately

[If not overdue: "Payment is current — [X] days until due."]

### Notes
[Any special payment instructions from invoice, vendor-specific notes, or cash flow considerations]
```

---

## Cash Flow Note (Optional)

If the user is managing cash flow across multiple invoices, add:

```
### 30-Day Cash Flow Snapshot
| Due Date | Vendor | Invoice # | Amount | Method |
|----------|--------|-----------|--------|--------|
[Sorted by due date, all invoices in current batch]

Total due in next 7 days:  $[X]
Total due in next 30 days: $[X]
```
