---
name: invoice-processing
description: >
  End-to-end invoice processing skill covering the full lifecycle: intake and data extraction,
  validation and exception handling, GL coding and cost allocation, approval routing, ERP/system
  entry guidance, and payment scheduling. Use this skill whenever the user wants to process,
  validate, code, route, or enter an invoice into a system — even if they just say "process this
  invoice", "code this bill", "route for approval", "enter into the system", "check if this invoice
  is valid", or "set up payment". Also triggers for batch invoice processing, invoice exception
  management, duplicate detection, and invoice-to-PO matching. Works with PDFs, images, Excel/CSV,
  and Word documents. Always use this skill when invoice processing is the primary task — don't
  just extract fields, run the full processing workflow.
---

# Invoice Processing Skill

Full lifecycle invoice processing: from raw document intake through validated, coded, approved, 
and payment-scheduled output. Covers single invoices and batch runs.

---

## Workflow Overview

```
INTAKE → EXTRACT → VALIDATE → CODE → ROUTE → ENTRY PREP → SCHEDULE PAYMENT
```

Each stage has a reference file. Read only what you need for the user's task.

---

## Reference Files

| Stage | When to Read |
|-------|-------------|
| `references/01-intake-and-extraction.md` | User uploads an invoice / starts processing |
| `references/02-validation-and-exceptions.md` | Checking invoice correctness, flagging issues |
| `references/03-gl-coding.md` | Assigning GL accounts, cost centers, departments |
| `references/04-approval-routing.md` | Determining who approves and escalation rules |
| `references/05-erp-entry.md` | Preparing data for system entry (SAP, NetSuite, QuickBooks, etc.) |
| `references/06-payment-scheduling.md` | Due dates, early pay discounts, cash flow planning |

Read **all relevant stages** upfront for multi-step requests (e.g., "process and code this invoice" → read 01 + 02 + 03).

---

## Quick Decision Guide

```
User request?
│
├── "Process this invoice" / uploaded a file
│   └── Read 01 (extract) → 02 (validate) → 03 (code) → ask if routing needed
│
├── "Is this invoice valid?" / "Check this invoice"
│   └── Read 01 (if not yet extracted) → 02 (validation focus)
│
├── "Code this invoice" / "What GL account?"
│   └── Read 03; may need 01 first if not yet extracted
│
├── "Who should approve?" / "Route for approval"
│   └── Read 04
│
├── "Enter into [SAP/NetSuite/QuickBooks/etc.]"
│   └── Read 05
│
├── "When should we pay?" / "Schedule payment"
│   └── Read 06
│
└── Batch invoices (multiple files/rows)
    └── Read 01 → 02 → produce batch summary table (see 01 for batch format)
```

---

## Core Principles

**Never guess at amounts.** If a number is unclear, mark `[unclear]` and flag for human review.

**Flag first, process second.** Any validation exception (duplicate, math error, missing PO, 
over-tolerance) gets flagged immediately before proceeding with coding or routing.

**Structured output at every stage.** Each stage produces a clean, copy-pasteable output block 
so the user can act immediately — no reformatting needed.

**Preserve source currency and format.** Never convert currencies unless explicitly asked.

**Audit trail.** Every decision (GL code assigned, approval tier selected, payment date chosen) 
should include a brief rationale so it can be reviewed or overridden.

---

## File Handling

| File Type | Approach |
|-----------|----------|
| PDF | Use `pdf-reading` skill if available; otherwise `pdftotext` via bash |
| Scanned image / photo | Read field by field; mark uncertain text `[?]` |
| Excel / CSV | Read with pandas; preview headers before processing |
| Word (.docx) | Extract with `python-docx`; preserve table structure |
| Email body | Parse as plain text; note any attachments still needed |

Always confirm file type and confirm extraction quality before proceeding to validation.

---

## Output Structure (Full Processing Run)

When running a complete invoice processing workflow, produce sections in this order:

1. **Extracted Invoice Data** (from 01)
2. **Validation Results** (from 02) — STOP and flag if critical exceptions found
3. **GL Coding** (from 03)
4. **Approval Routing** (from 04)
5. **ERP Entry Data** (from 05, if requested)
6. **Payment Schedule** (from 06)
7. **Processing Summary** — one-line status per section

If the user only needs some stages, produce only those sections.
