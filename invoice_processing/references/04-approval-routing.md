# 04 — Approval Routing

## Goal
Determine who needs to approve the invoice, in what order, and flag any escalation
requirements. Produce a clear routing instruction the user can act on immediately.

---

## Standard Approval Tiers (Default — Override with Company Policy)

| Invoice Amount | Approval Required |
|---------------|------------------|
| < $500 | AP Clerk (self-approve) |
| $500 – $2,499 | Department Manager |
| $2,500 – $9,999 | Director / VP |
| $10,000 – $49,999 | CFO or Finance Director |
| $50,000+ | CFO + CEO (dual approval) |
| Any amount — new vendor | Department Manager + Finance |
| Any amount — CapEx | Finance + relevant VP + CFO |
| Over PO amount | Original PO approver + Finance |

> If the user provides their actual approval matrix, use that instead.

---

## Routing Factors

Always consider these factors beyond dollar amount:

| Factor | Routing Implication |
|--------|-------------------|
| New vendor (first invoice) | Require vendor setup approval + standard tier |
| Vendor bank details changed | Flag as HIGH RISK; require Finance director sign-off |
| Invoice over PO amount | Route back to PO requestor + procurement |
| Unbudgeted expense | Add budget owner to approval chain |
| Intercompany transaction | Route to entity finance leads on both sides |
| CapEx item | Add asset management / IT approval |
| Contract-governed invoice | Confirm against contract terms before routing |
| Rush / urgent payment | Note urgency; may need priority queue |

---

## Escalation Rules

Escalate immediately if:
- Invoice on HOLD from validation (Stage 2)
- Vendor is new AND amount > $5,000
- Payment details differ from vendor master
- Invoice is past due (late fee risk)
- Invoice is older than 90 days

---

## Output Format

```
## [STAGE 4] Approval Routing

**Invoice Total**: $[X]
**Vendor Status**: [Existing / New]
**Special Flags**: [CapEx / Over PO / New Vendor / None]

### Approval Chain
| Step | Approver Role | Name (if known) | Action Required |
|------|--------------|----------------|----------------|
| 1 | [Role] | [Name or TBD] | Review and approve invoice |
| 2 | [Role] | [Name or TBD] | Final approval |

**Routing Method**: [Email / ERP workflow / DocuSign / Manual sign-off]

### Escalation Flags
[Any conditions requiring escalation, or "None"]

### Suggested Email / Routing Note
> "Please review and approve Invoice #[X] from [Vendor] for $[Amount], due [date].
> [Any context: PO reference, project, urgency note.]
> Attached: [invoice file]."
```

---

## If Company Approval Policy Is Unknown

Ask: "What are your approval thresholds? I'll route accordingly — or I can apply a standard
tiered model and you can adjust."

Provide the routing based on the default tiers above, and note clearly: "Using default approval
tiers — please confirm this matches your policy."
