# Invoice Agent – Synthetic Sample Dataset

This package contains 18 synthetic invoice test cases and the procurement records needed to evaluate them. No real vendor or university data is intended.

## Files

- `invoice_documents.json`: raw invoice payloads. Field-name variations are deliberate and test normalisation. No expected outcomes are embedded in the invoice documents.
- `reference_data.json`: vendors, purchase orders, goods receipts, service confirmations, goods returns, prior invoices, FX reference fixtures and demo policies.
- `test_manifest.json`: expected recommendation and findings, kept separate from invoice input so the agent cannot read the answer as part of the invoice.
- `sample_dataset.json`: all three datasets combined for convenient inspection.

## Required execution order

1. Seed vendors, purchase orders, goods receipts, service confirmations, goods returns, prior invoices and FX fixture records from `reference_data.json`.
2. Run case 01 before case 07 (duplicate detection).
3. Seed or process `CHAIR-114-01` before case 14 (cumulative PO overbilling).
4. Run the remaining cases in sequence.
5. Compare the result to `test_manifest.json`; do not pass the manifest into Gemini or the extraction prompt.

## Data and testing notes

- These are synthetic fixtures, not actual tax/legal advice or real market FX rates.
- GST split/rate rules are demo assumptions; actual treatment depends on applicable jurisdiction and place of supply.
- Raw invoice fields intentionally vary (for example `invoice_no`, `invoice_number`, `invoice_id`; `vendor`, `supplier`, `vendor_name`). A backend normalisation layer should map aliases to a canonical schema.
- Invoice documents contain vendor claims. PO, GRN, service confirmation, vendor-master, prior-invoice and FX records are separate internal reference data.
- For three-way matching, use the PO for ordered quantities, GRNs for accepted quantities, and invoice for billed quantities. Do not trust invoice-supplied `quantity_ordered` or `quantity_delivered` fields as internal evidence.
- Case 05 is a service invoice: check the service confirmation instead of a physical GRN.
- Case 10 is a credit note: check the original invoice and return record, then route for human decision.
- Case 18 needs a real low-quality PDF/image for an end-to-end OCR test; its JSON contains only partial extracted data and quality metadata.
- Case 02 has an unmapped order-like reference. Do not silently treat it as a confirmed PO.
- Case 07 is a duplicate of case 01. A prior invoice record must exist.
- Case 14 tests cumulative invoiced quantity against one PO, so prior invoice history must be seeded.

## Canonical comparison fields recommended

Vendor: vendor ID / legal name / GSTIN.
Invoice: invoice number, date, PO reference, currency, billed quantities, unit prices, tax and total.
PO: status, vendor, currency, ordered quantities, approved prices, tolerances and tax terms.
GRN: PO number, item, received quantity, accepted quantity and status.
Service confirmation: PO number, service period, accepted hours or milestone.
Credit note: original invoice reference, PO reference, returned items and accepted return quantity.

## Expected recommendations

Only use `READY_FOR_APPROVAL` and `NEEDS_HUMAN_REVIEW` as the final recommendation labels. `expected_findings` contains test assertions, not extra final recommendation states.
