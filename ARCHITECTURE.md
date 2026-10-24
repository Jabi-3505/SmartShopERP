# Architecture

SmartShop ERP is one local modular application. A React production build and
versioned Django REST API share the same loopback origin. Waitress hosts both.
PostgreSQL is the only supported application database. Daily operation has no
external service, CDN, API key, payment gateway or internet dependency.

## Boundaries

- `frontend/src`: authentication, operational forms, POS and reports. Amounts
  returned by the server are exact decimal strings. Display/estimation never
  authorizes a posting.
- `backend/erp/api.py`: authenticated HTTP input, role checks, shop scoping,
  validation, CSV import/export and invoice output.
- `backend/erp/services.py`: atomic financial and stock posting, idempotency,
  cash-only policy, valuation, returns and allocations.
- `backend/erp/models.py`: normalized operational records, immutable snapshots,
  accounts, journal lines, stock movements and audit history.
- `backend/erp/reports.py`: read-only calculations from persisted ledgers and
  documents. Profit is based on revenue, COGS and operating accounts.
- `backend/erp/migrations`: schema constraints, deferred reconciliation and
  database guards, including sealing completed documents against line insertion.
- `backend/config`: local deployment, session/CSRF security, exact JSON encoding
  and production asset serving.
- `scripts`: operator-controlled setup, foreground startup, verification and
  backup/recovery helpers.

## Posting transaction

A request identifies the authenticated shop and role. The service locks the
shop row, validates the idempotency key and request hash, locks relevant cash,
party and product rows, and calculates all values with Python Decimal.

It writes an unsealed document, item snapshots, stock movements, cash effects,
party allocations and balanced journal. Headers are sealed before commit.
Deferred PostgreSQL constraints validate ledger/cache reconciliation and completed
postings. Any error rolls back the whole transaction. A successful retry with
identical details returns the existing document; mismatched key reuse fails.
Printing takes place after saving, and a saved receipt can be reprinted.

One shop's writes are serialized deliberately. This is a correctness-first
strategy for a local single-store release; it is not a claim of high-volume,
multi-terminal benchmark performance. PostgreSQL tests exercise independent
connections and overlapping posting requests.

## Accounting policies

Inventory uses perpetual moving weighted-average carrying cost, with explicit
opening inventory against owner capital. Sales transfer carrying cost to COGS.
Sales returns reverse original sold cost; purchase returns remove current
carrying cost and recognize any difference from the original supplier amount.

Cash tender/change are receipt facts; only the settled invoice amount increases
the drawer. Credit sales are wholly unpaid receivables, distinct from cash
settlement. Cash collections reduce balances and never recognize sales twice.
Supplier credit follows the corresponding payable model. Owner cash transfers
affect capital/drawings, not operating profit. Cash-count differences have their
own income/expense accounts.

Money is stored to two decimal places, quantities to three, inventory unit costs
to six. Calculations round half up. Saved price/tax/cost snapshots survive master
record edits. Journal, movement and item records cannot be changed or deleted;
posted headers cannot receive additional lines. Corrections use linked returns
or new authorized adjustments.

## Deployment and trust boundary

The normal application uses a dedicated local PostgreSQL role and a private
installation-specific `.env`. It binds to loopback, checks remote addresses,
uses session cookies and CSRF tokens, and protects first-run setup with a secret
token and a database initialization lock. Staff permissions are enforced by the
API and posting services, not only navigation visibility.

A local Windows/database administrator can modify software or database access;
this is an explicit trust boundary. This package does not set up remote access,
an Electron shell, a Windows service, cloud execution or automatic scheduling.
See SETUP_WINDOWS.md for operating and recovery procedures, and VERIFICATION.md
for observed results and remaining implementation limits.
