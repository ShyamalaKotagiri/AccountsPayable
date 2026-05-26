# 02 — Invoice Validation & Exception Handling

## Goal
Verify the invoice is accurate, complete, and safe to process. Flag any issue before
it moves to coding or payment. Never suppress a flag — surface everything, even minor items.

---

## Validation Checklist

Run all checks. Mark each: ✅ Pass | ⚠️ Warning | ❌ Fail | N/A

### Completeness Checks
- [ ] Invoice number present
- [ ] Invoice date present and valid (not future-dated beyond tolerance)
- [ ] Due date present or derivable from terms
- [ ] Vendor name present
- [ ] Bill-to company matches our organization
- [ ] At least one line item or a lump sum total
- [ ] Invoice total present

### Math Verification
- [ ] Line item totals = Qty × Unit Price (within rounding tolerance of ±$0.02)
- [ ] Subtotal = sum of line items
- [ ] Tax calculation correct given stated rate
- [ ] Invoice Total = Subtotal + Tax + Shipping + Other Charges − Discount
- [ ] Balance Due = Invoice Total − Amount Paid

**If math fails:** note the expected vs. actual amount and the delta.

### Duplicate Detection
- [ ] Invoice number not already seen in this batch
- [ ] Vendor + Amount + Date combination not duplicated
- **Flag any potential duplicate immediately** — duplicate payment risk is high.

### PO Matching (if PO number present)
- [ ] PO number format looks valid (not obviously incorrect)
- [ ] Invoice total does not exceed PO value by more than tolerance
  - Typical tolerance: ±5% or $500, whichever is lower (adjust per company policy)
- [ ] Line items on invoice correspond to PO line items (if PO data provided)
- [ ] Vendor on invoice matches vendor on PO

### Vendor Validation (if vendor master data provided)
- [ ] Vendor name matches vendor master
- [ ] Vendor address matches records (flag changes — fraud risk)
- [ ] Bank/payment details match records (flag changes — fraud risk)
- [ ] Vendor is not on hold or blocked

### Policy Checks
- [ ] Invoice not older than [company policy — default: 90 days] from invoice date
- [ ] Invoice currency is approved / expected
- [ ] No round-number anomaly (e.g., exactly $10,000.00 with no line items — flag for review)
- [ ] Tax ID present if required by jurisdiction

---

## Exception Severity Levels

| Level | Label | Action Required |
|-------|-------|----------------|
| 🔴 Critical | HOLD | Do not process. Requires human resolution before continuing. |
| 🟡 Warning | REVIEW | Flag and note; can proceed with approval if reviewer signs off. |
| 🟢 Info | NOTE | Minor item; log for awareness but do not block processing. |

### Critical (HOLD) Exceptions
- Math error on invoice total
- Duplicate invoice number (same vendor)
- Vendor payment details changed from master record
- Invoice amount exceeds PO by >tolerance
- Vendor on hold/blocked
- Missing invoice number
- Suspected fraud indicators (round numbers + no line items, new vendor + large amount, etc.)

### Warning Exceptions
- Missing PO reference on invoice over threshold (e.g., >$500)
- Invoice date is older than 90 days
- Tax calculation slightly off (within rounding)
- Missing vendor tax ID
- Line item descriptions vague/incomplete

### Info Exceptions
- Minor rounding difference (≤$0.02)
- No payment instructions provided
- Bill-to address slightly differs from our standard format

---

## Output Format

```
## [STAGE 2] Validation Results

**Overall Status**: ✅ APPROVED TO PROCEED | 🟡 PROCEED WITH REVIEW | 🔴 HOLD — ACTION REQUIRED

### Checks Summary
| Check | Result | Notes |
|-------|--------|-------|
| Completeness | ✅ | |
| Math verification | ✅ | Subtotal: $X, Tax: $Y, Total: $Z — all correct |
| Duplicate check | ✅ | No duplicates detected in this batch |
| PO matching | ⚠️ | Invoice $5,200 vs PO $5,000 — 4% over, within tolerance |
| Vendor validation | ✅ | |
| Policy checks | ✅ | |

### Exceptions
[If none: "No exceptions found."]

🔴 CRITICAL — [Exception type]
  - Detail: [what was found]
  - Required action: [what must happen before processing]

🟡 WARNING — [Exception type]
  - Detail: [what was found]
  - Recommended action: [what should be reviewed]

🟢 INFO — [Exception type]
  - Detail: [what was noted]
```

---

## If Critical Exceptions Found

**STOP** and present the HOLD notice. Do not proceed to GL coding or routing.

Tell the user:
> "⛔ Processing is on hold due to [exception]. Please resolve the following before I continue:
> [list of required actions]. Once resolved, let me know and I'll continue with coding and routing."

Only continue to Stage 3 if the user confirms the exception is resolved or instructs you to proceed anyway (note that they have overridden the hold).
