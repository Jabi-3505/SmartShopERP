# SmartShop ERP

SmartShop ERP is a local Windows retail application built with React, Vite,
Bootstrap, Django REST Framework and PostgreSQL. The browser opens the application
at `http://127.0.0.1:8765`; Waitress serves the Django API and locally built UI.
Daily operations use the local database and local assets. No payment gateway,
paid API, external tax filing API, cloud account or runtime API key is required.

Start with [SETUP_WINDOWS.md](SETUP_WINDOWS.md). This is a source ZIP and a
localhost application, not a signed Windows installer or an Electron package.
The production frontend is included in `frontend/dist`; normal use needs no Node.js
installation. The `vendor/wheels` bundle and hash-pinned `requirements-lock.txt`
provide the Python 3.14 Windows x64 dependencies for offline installation.
Other supported platforms/Python versions require installation from the official
registries. Ordinary operation after setup does not require internet access.

## Working model

- Create the first owner and shop with a private, installation-specific setup token.
- Manage products/categories, customers, suppliers and staff permissions.
- Open the cash register, receive inventory through purchases or opening stock,
  then bill products through POS and view/print saved invoices.
- Record expenses, authorized stock adjustments and returns linked to original sales.
- Return purchased goods against their original purchase and record owner cash
  capital/drawings without counting those transfers as sales or expenses.
- Where enabled by the owner, record wholly unpaid customer credit and collect it
  later in cash. Supplier credit is a liability, with subsequent cash settlement.
- Review persisted sales, cash, stock movements, ledgers, trial balance and profit reports.

Cash is the sole settlement medium. Customer credit is an unpaid receivable,
not another payment method. No mixed tender, card, UPI, QR payment or online
settlement is offered. A sale of INR 850 tendered with INR 1,000 records INR 850
of cash, with INR 150 change saved for the receipt; change is not an expense.

See [VERIFICATION.md](VERIFICATION.md) for the actual build, test and acceptance
results for the delivered package. The presence of a test or screen alone is
not evidence that it passed verification.

## Financial and inventory rules

All posting goes through transaction-scoped backend services. Posting locks the
shop first, then relevant rows in a consistent order; one shop's postings are
serialized to favor correctness on a local installation. Different shops remain
independent. The browser cannot directly set product balances or journal totals.

Each posted sale/purchase creates its operational document, stock movements,
balanced journal and cash/party effect in one PostgreSQL transaction. A failed
posting rolls everything back. Shop-scoped idempotency keys make an identical
retry return the saved document. Reusing a key with different data is rejected.
Posted ledger records are append-only; corrections use linked returns or new
adjustments. PostgreSQL guards supplement application checks.

Inventory uses perpetual moving weighted-average cost. Receipts add their net
inventory value, issues consume the current average cost, and the final issue
consumes the exact remaining value so that zero stock has no stranded cents.
Sales returns use the original sale's cost rather than today's purchase price.
Stock quantity and stock value are cached balances reconciled to the movement
ledger. Inventory purchases are capitalized, not treated as operating expenses.
An explicitly designated opening-stock adjustment credits owner capital; an
ordinary positive stock correction credits inventory adjustment gains. Purchase
returns refund the original purchase amount and remove the current average
carrying cost; a difference is posted as an inventory adjustment gain/loss.

Money and tax amounts are Decimal/NUMERIC with two decimal places; quantities
allow three decimal places and average/unit inventory costs allow six. Computed
money rounds half up to a paisa. The server rejects invalid, non-finite, negative
or excessive-precision inputs rather than silently accepting them. Discounts are
line amounts before tax; inclusive prices extract tax from the discounted total.
CGST receives the rounded half of intra-state tax and SGST the residual paisa;
inter-state mode uses IGST. Totals are sums of the saved line snapshots.

Cash register reconciliation and profit are separate. Revenue, COGS and operating
expenses feed profit reports. Credit collections reduce receivables without
recording revenue again. Initial owner cash is capital; cash shortages/overages
are recorded when a counted register is closed.

## Security and operations

The server binds to loopback only. Use Django session authentication and CSRF
protection. Shop memberships and roles restrict API operations. First-owner
initialization is one-time and requires `SETUP_TOKEN`. There are no default user
credentials. Keep `.env` private, use a dedicated PostgreSQL application role,
and run the application from a Windows account restricted to authorized staff.

Application immutability protects normal posting and database operations; a
machine/database administrator can change the software or database and remains
part of the trust boundary. This release is for a single trusted local machine.
Do not expose Waitress, PostgreSQL or development servers to the LAN/internet
without a separate deployment/security review.

The startup script runs Waitress in the foreground so the operator can see errors
and shut it down with Ctrl+C. It does not install a service, modify the firewall,
start hidden background jobs or manage another PostgreSQL instance. PostgreSQL
is managed through its normal Windows service.

Back up with `scripts/backup.ps1`; test recovery with `scripts/restore.ps1` into
a new database. The scripts install nothing and never overwrite an existing
database. A database archive does not contain source code, `.env` or separately
stored files. Keep protected offline copies and test restoration regularly.

## GST scope and remaining work

GST rates and HSN classifications are configurable. Invoice lines retain the
selected name, price, tax rate, tax mode and amounts. The application does not
submit GST returns or obtain IRNs. Its PDF and tax summaries are working records;
they must not be described as an IRP e-invoice or a filed return.

Product classification, rate applicability, input-tax-credit eligibility,
place-of-supply treatment, exempt-supply documents, statutory invoice particulars
and any applicable e-invoicing/e-way-bill obligations require verification for the
actual shop before production compliance can be claimed. The app's purchase-tax
accounts are bookkeeping records, not confirmation that tax credit is legally available.

The wider ERP brief also calls for batch/expiry tracking, multiple warehouses,
document/image handling and an optional desktop installer.
These are backlog items unless the delivery's verification record explicitly
marks them implemented and tested. Hardware printers/barcode scanners, real
opening balances and the shop's full accounting workflow require operator
acceptance. Do not load live accounts until the acceptance checks in the Windows
guide are complete.

Official GST references checked during development:

- [CBIC invoice particulars, Rule 46](https://taxinformation.cbic.gov.in/content-page/explore-rules/1000136/1000001)
- [CBIC stock records and electronic edit logs, Rule 56](https://taxinformation.cbic.gov.in/content-page/explore-rules/1000149/1000001)
- [CGST record retention, section 36](https://taxinformation.cbic.gov.in/content-page/explore-act/1000306/1000001)
- [Government explanation of September 2025 goods-rate notifications](https://www.pib.gov.in/Pressreleaseshare.aspx?PRID=2168426&lang=2&reg=3)

Rules and portal formats can change. Keep effective dates and past transaction
snapshots intact when updating shop tax configuration.

## Project layout and license

`backend/config` contains deployment settings; `backend/erp` contains the API,
models, business services, migrations and PostgreSQL integrity tests.
`frontend` contains the React source and local dependency manifest/lockfile.
`scripts` contains reviewed Windows setup/start/verification/recovery helpers.

Original SmartShop ERP source is provided under the [MIT license](LICENSE).
Dependencies retain their own open-source licenses; preserve their notices when
redistributing a built package. No cloud runtime, overnight continuation,
automatic scheduling or unattended future work is configured by this package.
