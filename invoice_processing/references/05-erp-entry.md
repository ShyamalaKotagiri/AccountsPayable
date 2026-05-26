# 05 — ERP / System Entry Preparation

## Goal
Produce a system-ready data block formatted for the user's ERP or accounting system,
so they can enter or import the invoice with minimal manual work.

---

## Supported Systems

| System | Entry Format |
|--------|-------------|
| QuickBooks Online / Desktop | Field-by-field entry guide |
| NetSuite | Journal entry / AP bill format |
| SAP (S/4HANA / ECC) | MIRO / FB60 field mapping |
| Xero | Bill entry format |
| Sage Intacct | AP bill format |
| Microsoft Dynamics 365 | Vendor invoice format |
| Generic / Unknown | Universal AP bill format |

Ask the user which system they use if not specified. Default to Universal format.

---

## Universal AP Bill Format

```
VENDOR INVOICE ENTRY
====================
Vendor:           [Vendor Name]
Vendor ID:        [Vendor ID if known]
Invoice #:        [Invoice Number]
Invoice Date:     [YYYY-MM-DD]
Due Date:         [YYYY-MM-DD]
Payment Terms:    [Terms]
Currency:         [Currency Code]
Reference / Memo: [PO # or project ref]

LINE ITEMS
----------
Line 1:
  Description:    [Line item description]
  GL Account:     [Account Code] — [Account Name]
  Cost Center:    [CC Code]
  Department:     [Dept]
  Project:        [Project Code or blank]
  Amount:         $[X.XX]

[Repeat for each line]

TAX
---
  Tax Account:    [GL Code]
  Tax Type:       [Sales Tax / Use Tax / VAT]
  Tax Amount:     $[X.XX]

TOTALS
------
  Subtotal:       $[X.XX]
  Tax:            $[X.XX]
  Shipping:       $[X.XX]
  Invoice Total:  $[X.XX]

ATTACHMENTS
-----------
  [ ] Invoice PDF attached
  [ ] PO attached (if applicable)
  [ ] Approval documentation attached
```

---

## System-Specific Field Mappings

### QuickBooks Online
| QBO Field | Value |
|-----------|-------|
| Vendor | [Vendor Name] |
| Invoice no. | [Invoice #] |
| Invoice date | [Date] |
| Due date | [Date] |
| Category | [GL Account Name] |
| Description | [Line description] |
| Amount | [Amount] |
| Tax | [Tax amount] |

> In QBO: Expenses → Vendors → Create Bill. Enter line by line.

### SAP MIRO / FB60
| SAP Field | Value |
|-----------|-------|
| Company Code | [Entity] |
| Document Date | [Invoice Date] |
| Posting Date | [Today or period-end] |
| Vendor | [Vendor Master #] |
| Reference | [Invoice #] |
| Amount (Gross) | [Invoice Total] |
| Tax Code | [Tax code per jurisdiction] |
| GL Account | [GL Code] |
| Cost Center | [CC Code] |
| Internal Order | [Project Code if applicable] |

### NetSuite (Vendor Bill)
| NetSuite Field | Value |
|---------------|-------|
| Vendor | [Vendor Name] |
| Ref. No. | [Invoice #] |
| Date | [Invoice Date] |
| Due Date | [Due Date] |
| Account | [GL Account] |
| Amount | [Line Total] |
| Department | [Dept] |
| Location | [Entity] |
| Class | [Cost Center] |
| Memo | [Description] |

---

## Import / Upload Option

If the user wants to batch-import invoices, produce a CSV with these columns:
```
Vendor,InvoiceNumber,InvoiceDate,DueDate,Currency,GLAccount,CostCenter,Department,
ProjectCode,Description,Quantity,UnitPrice,LineTotal,TaxType,TaxRate,TaxAmount,
ShippingAmount,InvoiceTotal
```

Populate one row per line item.

---

## Output Format

```
## [STAGE 5] ERP Entry Data — [System Name]

[Use the appropriate system format above, fully populated]

### Entry Checklist
- [ ] Invoice PDF attached to record
- [ ] GL coding verified (total coded = $[X])
- [ ] Approval documentation attached / approval obtained
- [ ] Duplicate check cleared
- [ ] Vendor terms confirmed (Net [X])

### Entry Notes
[Any special instructions, manual steps, or fields requiring lookup]
```
