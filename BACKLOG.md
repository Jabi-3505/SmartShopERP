# Remaining work from the wider ERP brief

This release is a working localhost retail/accounting application. The items
below are not completed features and must not be inferred from its navigation,
database technology or general ERP name. See VERIFICATION.md for executed checks.

## Before accepting live hypermarket operations

1. Add batch numbers, expiry dates, batch stock allocation/returns and expiry
   alerts. Current stock and weighted-average valuation are per product.
2. Validate the actual shop's GST registrations, product classifications,
   exemptions, place of supply, input-credit treatment, statutory document types
   and required invoice particulars. Current GST summaries are bookkeeping
   working reports; no return filing, IRN registration or compliance certification
   is implemented. Applicable online statutory processes need a separately
   reviewed operational procedure alongside this offline application.
3. Add controlled opening balances for existing receivables/payables and other
   balance-sheet accounts, with an accountant-approved migration/reconciliation.
   Current explicit opening workflows cover drawer cash and inventory.
4. Perform operator acceptance with the real barcode scanner, receipt printer,
   stock units, staff workflow and required transaction volume. Software keyboard
   input and saved invoice/PDF output do not certify physical hardware operation.
5. Rehearse recovery using the shop's own protected backup media and reconcile
   reports before entering live opening accounts.

## Functional expansion

- Multiple warehouses/registers, formal unit/conversion masters and transfers.
- Customer due dates, overdue aging and configurable allocation review UI.
- Product images and safe expense/supporting-document storage.
- Purchase orders, richer procurement references and supplier workflows.
- Custom chart-of-accounts maintenance and controlled manual correcting journals.
- Date-range/fiscal-period reporting, larger exports, additional print layouts and
  embedded regional-language PDF fonts/shaping.
- Historical effective-dated tax master administration; posted invoices already
  retain the exact tax/price snapshots originally used.
- Stronger accessibility/large-volume usability testing and independent security
  review before broader deployment.

## Distribution and deployment

An Electron shell, signed Windows installer, automatic updates, cloud execution,
scheduled resumption, LAN hosting and unattended Windows service installation are
not included. The delivered operating model is the documented foreground Waitress
server plus locally installed PostgreSQL, opened in a browser on the same PC.
The project starts empty and has no default owner password or business/demo data.

No card, UPI, bank-transfer, mixed-tender, online-payment, analytics, advertising,
paid API or external tax-filing integration is planned as an alternative payment
workflow. Cash-only settlement remains a fixed requirement.
