# SmartShop ERP 0.1.0 verification

Verified on 24 October 2026. This is a working Windows localhost source release,
with a prebuilt React UI and offline Python wheel bundle. It is not an Electron
installer, a completed implementation of every advanced ERP item, or a GST
compliance certification. Remaining scope is listed in BACKLOG.md.

## Executed checks

| Check | Observed result |
|---|---|
| Django application/system checks | Passed; 0 issues |
| PostgreSQL migrations 0001–0003 | Applied successfully on an isolated PostgreSQL 18.6 cluster |
| Migration/model consistency | No pending model changes |
| Full backend suite | **49 tests passed**, 76.545 seconds, exit code 0 |
| React production build | Passed with Vite 8.3.5; built assets included in `frontend/dist` |
| npm dependency audit | 0 reported vulnerabilities at build time |
| Official PyPI release metadata | 12 locked runtime packages; 0 reported advisories in the retrieved release metadata |
| Offline standard Python installation | All 12 hash-locked wheels installed into a fresh project test virtual environment with `--no-index`; `pip check` passed |
| Browser checkout workflow | Passed against real Waitress/PostgreSQL; no browser JavaScript errors |
| Saved invoice PDF | Valid PDF generated, rendered with Poppler and visually inspected |
| Windows helper syntax/config handling | All PowerShell helpers parsed; literal password escaping, environment precedence and required-secret checks exercised |
| Backup and restore | Actual shipped scripts passed; fresh restored database reconciled to the source snapshot |

Dependency audit results are observations from the checked registries, not an
independent security certification or a guarantee about future advisories.

## Financial and security coverage

The PostgreSQL suite uses real transactions and separate connections for races.
It covers:

- Cash tender/change, insufficient cash/stock and rejection of unsupported
  payment values.
- Purchase → sale → partial/full return and historical price/tax/cost snapshots.
- Moving-average inventory, final-cent consumption and fractional tax/return
  rounding conservation.
- Concurrent last-unit sales, identical retries, unique invoice numbering,
  conflicting idempotency reuse and concurrent return/credit limits.
- Rollback after a deliberately injected partial-posting failure.
- Balanced journals, stock-cache reconciliation and customer/supplier settlements.
- Posted-record changes/deletions and attempts to append lines to sealed entries.
- Login/setup/session CSRF, private setup token and repeated-initialization rejection.
- Unauthenticated access, role boundaries, cross-shop lookup/relationships,
  staff escalation, inactive accounts and cashier credit-limit restrictions.
- Read-only balances, safe master edits after intervening stock changes, and
  cashier redaction of purchase costs, inventory valuation and invoice COGS.
- Taxable value after discounts/inclusive tax, purchase-return tax reporting and
  supplier statement reconciliation.

Tests ran in a separate task-owned database using `--keepdb`. The temporary
cluster used loopback/SCRAM authentication and test-only autovacuum/io settings;
the package does not change the user's existing PostgreSQL service or prescribe
those test settings for normal operation.

## Actual browser acceptance

An installed Chrome browser, with its normal sandbox enabled and a fresh test
profile, exercised real owner onboarding, cash-register opening, product/supplier
creation, cash purchasing, inventory receipt, immediate sequential scans of two
different barcodes, F8 sale confirmation, saved invoice/PDF and financial reports.
Browser request routing allowed local application resources only.

The synthetic test sale verified:

- Invoice INR 135.00; tender INR 200.00; change INR 65.00.
- Net cash increase INR 135.00, rather than the tendered INR 200.00.
- Cost of Goods Sold INR 90.00; gross/net profit delta INR 45.00.
- Correct remaining product quantities/carrying values and a balanced trial balance.

Loaded setup, dashboard, POS, trial balance, register and invoice screens were
visually checked. The final invoice recipient block uses the saved customer
snapshot. The PDF was rendered separately and checked for legible text, correct
amounts and absence of overlap/clipping. No physical receipt printer or scanner
hardware was connected; barcode keyboard input was exercised.

## Recovery verification

The shipped `backup.ps1` created a PostgreSQL custom archive of the synthetic QA
database. `restore.ps1` restored it into a **new** database without changing the
original database or environment file. Comparison matched all 36 table counts
and the saved stock, cash, invoice, sealed-posting and accounting snapshots.

The restored sample contained four products, two purchases, one sale, six stock
movements and four journals/ten lines. Trial-balance debits and credits both
equaled INR 10,135.00; the difference was zero. Revenue was INR 135.00, COGS INR
90.00, profit INR 45.00 and cash INR 9,315.00. The balance-sheet difference was
also zero. These are explicitly synthetic verification records, not business data.

## Delivered and excluded

Included: application source, migrations/tests, prebuilt local UI, pinned npm
lockfile, hash-locked Windows x64/CPython 3.14 wheels, upstream licenses, setup,
startup, backup/restore and troubleshooting instructions.

Excluded: real/test databases, backup archives, `.env`, usable passwords, setup
tokens, test browser profiles, development virtual environments, `node_modules`
and runtime logs. A normal installation starts empty and creates its own owner.

Batch/expiry tracking, multiwarehouse operations, broader opening balances,
regional-script PDF typography, additional statutory GST workflows, actual
hardware acceptance and a signed desktop installer remain unfinished. Review
BACKLOG.md and complete shop/accountant acceptance before live operation.
