# SmartShop ERP on Windows

This guide sets up a local browser application, not a Windows desktop installer.
No script downloads software, installs a database or changes Windows security
policy. Run the commands yourself after reviewing the package. Keep the laptop
on while the application or local development is running.

## 1. Check prerequisites

Use a supported 64-bit Windows version and a current browser. Obtain software
manually from official sources only if it is not already installed:

- [Python for Windows](https://www.python.org/downloads/windows/): Python 3.14 Windows x64
  for the bundled offline wheels. Python 3.13 needs the online installation path below.
- [PostgreSQL Windows downloads](https://www.postgresql.org/download/windows/): PostgreSQL 18,
  including `psql`, `pg_dump`, `pg_restore` and `createdb`. The PostgreSQL project
  links its Windows installer provider; extra StackBuilder add-ons are not needed.

An existing PostgreSQL installation may already be suitable. Do not replace or
reinitialize an existing service, data directory or database. Use a new dedicated
database and role. Check versions in PowerShell:

```powershell
py -3.14 --version
& 'C:\Program Files\PostgreSQL\18\bin\psql.exe' --version
```

Node.js is needed only to rebuild the included frontend during development, not
for normal installation or operation. If Python 3.13 is installed, use `py -3.13`
in place of `py -3.14` and follow the online dependency path. PostgreSQL
tool paths may differ. Use backup tools from the same server major version or an
official compatible newer version. Do not download binaries from unofficial mirrors.

## 2. Unpack and install project dependencies

Extract the ZIP to a folder owned by your Windows account, for example
`C:\SmartShopERP`. Open PowerShell there. Avoid protected system folders and
cloud-synced folders for the working database or secret configuration.

```powershell
Set-Location 'C:\SmartShopERP'
py -3.14 -m venv .venv
.\.venv\Scripts\python.exe -m pip install --no-index --find-links vendor\wheels --require-hashes -r requirements-lock.txt
```

The offline bundle is specifically for **CPython 3.14, Windows x64 (AMD64)**.
Check `VERIFICATION.md` and `vendor/wheels/manifest.json` for the exact delivered artifacts.
It does not include Python or PostgreSQL installers. If using Python 3.13,
another architecture/platform, or a source package without the wheel bundle,
install compatible dependencies from official PyPI instead:

```powershell
.\.venv\Scripts\python.exe -m pip install --index-url https://pypi.org/simple -r requirements.txt
```

These commands install project dependencies only when you choose to run them.
Do not replace pinned versions during initial setup. The production UI assets
are already in `frontend/dist`; Node.js/npm and frontend rebuilding are optional
developer steps. If this delivery lists an unresolved dependency/build issue in
`VERIFICATION.md`, resolve it before continuing. No frontend CDN is used.

If virtual-environment creation reports an `ensurepip` error, repair the official
Python installation's pip/venv components. An existing official pip can also
target the new interpreter with `python -m pip --python .venv\Scripts\python.exe
install --index-url https://pypi.org/simple -r requirements.txt`. Do not fetch or
execute bootstrap scripts from unknown websites.

If Windows policy prevents a PowerShell helper from running, follow your
administrator's policy. Do not disable execution policy, antivirus or firewall.
The equivalent Python management commands are included below.

## 3. Create an empty application database

Open the official PostgreSQL SQL Shell (`psql`) as the local database administrator.
Create a dedicated role and database. These example names match the configure
script defaults; choose different new names if they already exist.

```sql
CREATE ROLE smartshop LOGIN NOSUPERUSER NOCREATEDB NOCREATEROLE;
\password smartshop
CREATE DATABASE smartshop OWNER smartshop ENCODING 'UTF8' TEMPLATE template0;
```

The `\password` command prompts privately. Do not put a real password in SQL
history, source files, screenshots or chat. The normal application role does not
need superuser or database-creation rights. Keep PostgreSQL listening locally,
use password/SCRAM authentication, and retain the installation's firewall rules.
Do not use `trust` authentication to work around a password error.

## 4. Generate local configuration

From the project root:

```powershell
.\scripts\configure.ps1 -Database smartshop -DatabaseUser smartshop -DatabasePort 5432
```

Enter the password set for the application role. The script creates `.env`,
generates unique cryptographic Django/setup secrets and prints the first-owner
setup token. It never overwrites an existing `.env`. The file contains your
database password; restrict the folder to authorized Windows users and keep it
out of the project ZIP, source control, email and public backups. The checked-in
`.env.example` contains no usable secrets.

Existing process environment variables override `.env`. Open a fresh terminal
after changing configuration if old values were exported. The scripts accept
single-quoted `.env` values generated by configure; do not add executable
PowerShell expressions or multiline values. No configuration value is executed.
For an isolated test installation, `SMARTSHOP_ENV_FILE` can explicitly select a
different local environment file for both Django and the helpers. Leave it unset
for normal use of the project-root `.env`.

## 5. Initialize and start

The commands below load `.env` for PostgreSQL tools and Django. They create the
application schema only in the database you configured.

```powershell
. .\scripts\common.ps1
Import-SmartShopEnvironment
Assert-SmartShopSecrets
Assert-LocalPostgres
Set-Location backend
..\.venv\Scripts\python.exe manage.py check
..\.venv\Scripts\python.exe manage.py migrate --noinput
Set-Location ..
.\scripts\start.ps1
```

Check each command succeeds before continuing. Open `http://127.0.0.1:8765` in
your browser and complete first-time owner/shop setup using the private token
printed by configure. Pick a strong, unique owner password. Once initialization
has succeeded, an unauthenticated caller cannot initialize another owner.
There are no built-in staff usernames/passwords. Create staff through the owner UI.

`start.ps1` refuses missing secrets, absent frontend builds or pending migrations.
It serves the production UI/API with Waitress on loopback and prints errors in
the terminal. It installs no service and launches no hidden worker. To start
Waitress manually after the same environment checks:

```powershell
Set-Location backend
..\.venv\Scripts\python.exe -m waitress --listen=127.0.0.1:8765 --threads=4 config.wsgi:application
```

## 6. Normal operation and shutdown

Keep the Waitress terminal open. Open a cash register before cash transactions.
Add opening inventory through the stock-adjustment workflow with an explicit
cost/reason and the opening-stock option (capital treatment), or receive a
purchase. An ordinary positive correction affects adjustment gains. Do not edit
database quantities directly.
The application starts empty; any test records you enter are your own records.

At day end, review cash movements, count the drawer and close/reconcile the cash
register. Complete any posting in progress, then press Ctrl+C in the Waitress
terminal. Closing only the browser does not stop the server. A closed terminal
stops the foreground application; the PostgreSQL Windows service remains managed
by Windows. Do not stop/restart a shared PostgreSQL service just to close SmartShop.

The application requires the laptop and PostgreSQL service to remain running.
After setup, normal billing uses no internet connection. Network sharing and
multiple computers are outside this localhost deployment guide.

## 7. Backup and test recovery

From the project root, while PostgreSQL is running:

```powershell
.\scripts\backup.ps1 -PostgresBin 'C:\Program Files\PostgreSQL\18\bin'
```

The default destination is `backups\smartshop-<timestamp>.dump`. You can choose a
new filename with `-File 'D:\ProtectedBackups\smartshop-20261024.dump'`. The script
does not overwrite files. It runs a consistent PostgreSQL custom-format dump,
validates the archive index with `pg_restore --list` and prints a SHA-256 hash.
A failed `.partial` file is not a valid backup. Do not infer recovery success
from archive listing alone.

Keep protected copies on a separate device. Database archives contain business
data. Store `.env`, application source/version and any uploaded files separately,
with appropriate access controls. PostgreSQL dumps are not automatically encrypted.

Restore first into a **new database**, using a unique lowercase name:

```powershell
.\scripts\restore.ps1 -File 'D:\ProtectedBackups\smartshop-20261024.dump' -TargetDatabase smartshop_recovery_20261024 -MaintenanceUser postgres -PostgresBin 'C:\Program Files\PostgreSQL\18\bin'
```

The administrator password is prompted when creating the fresh database. The
restored database is owned by the configured application role; the restore uses
that role's credentials from your local environment. If your chosen maintenance
role has a different name, change `-MaintenanceUser`. Without that parameter the
configured role must already have `CREATEDB`; do not grant this to the normal
production role just for convenience.

Restore refuses the configured live database and never drops an existing target.
It checks the archive before creation, restores in one transaction with
`--exit-on-error`, and confirms the Django migration table can be read. If a
restore fails, the new target may remain for inspection; the script does not
delete it. No `.env` setting is changed automatically.

To accept a recovered database: stop SmartShop, preserve a current backup, set
`PGDATABASE` in `.env` to the new name, open a fresh terminal, run the setup checks
and migration check, then start. Reconcile invoice counts, last invoice numbers,
cash, customer/supplier balances, stock quantities/values and trial balance
against the backup's expected date. Open a saved invoice and print a test receipt.
Restore separately stored files if applicable. Retain the original database and
archive until recovery has been accepted.

## 8. Developer and PostgreSQL verification

The same project environment is suitable for development. Edit source, rebuild
the frontend, and restart Waitress to test the integrated, same-origin production
assets. For frontend work only, install [Node.js 24 LTS with npm from the official
project](https://nodejs.org/en/download), then run:

```powershell
Set-Location frontend
npm ci --registry=https://registry.npmjs.org
npm run build
Set-Location ..
```

Preserve the dependency lockfile. Django's `runserver` is an optional development
tool only; it is not the packaged production startup. Any split Vite/API setup
requires reviewed development-only origin/proxy/CSRF settings.

Run non-mutating checks with:

```powershell
.\scripts\verify.ps1
```

Integrity tests require a separate PostgreSQL test/development role with
`CREATEDB` and a separate, empty development database. Do not use the production
role or live business database. Django creates/drops a database named
`test_<PGDATABASE>` during tests. Give that test role no access to other business
databases. Use a fresh terminal and override `PGUSER`, `PGDATABASE` and
`PGPASSWORD` for the isolated development connection; avoid placing a password
directly in command history.

```powershell
$env:PGUSER = 'smartshop_test'
$env:PGDATABASE = 'smartshop_dev'
$testPassword = Read-Host 'Test database role password' -AsSecureString
$env:PGPASSWORD = [Net.NetworkCredential]::new('', $testPassword).Password
$testPassword.Dispose()
$env:SMARTSHOP_ALLOW_TEST_DATABASE = '1'
.\scripts\verify.ps1 -RunPostgresTests
```

Close this test terminal afterward so exported credentials/settings do not carry
into normal startup. The tests use real PostgreSQL transactions/connections to
exercise overselling, concurrent retries, unique numbering, return limits,
credit limits, immutable records, rollback, cash/change, historical snapshots and
balanced journals. A passing run is meaningful; merely having these files is not
a passing result. Consult `VERIFICATION.md` for the delivered test results.

## 9. First business-use acceptance

Use a disposable shop/test database to complete these checks before loading live
opening balances:

1. Create owner and restricted staff; check unauthorized actions and repeated setup are rejected.
2. Enter two differently costed purchases, sell part/all stock and reconcile weighted-average COGS.
3. Tender more than the sale total; compare receipt change with net cash and revenue.
4. Retry a saved request/refresh the invoice; verify no duplicate stock or journal effects.
5. Test partial/full returns and rejection of excessive returns.
6. Where enabled, make a credit sale, collect cash partially and confirm revenue is unchanged.
7. Close the drawer with exact/short/over counted cash and inspect its journal/report.
8. Confirm actual printer output, keyboard shortcuts and scanner behavior.
9. Restore a backup into a new database and reconcile reports.
10. Have the shop's accountant verify GST classification, invoice format and opening balances.

## 10. Troubleshooting and updates

- **Database connection refused:** confirm the existing PostgreSQL service is running and
  `PGHOST`/`PGPORT` match it. Do not initialize a replacement data directory.
- **Password authentication failed:** check the application role and password. Existing
  process variables override `.env`; never change authentication to `trust`.
- **`psql`/backup tool not found:** use the official installation's bin directory through
  `-PostgresBin`; no global PATH edit is required for recovery scripts.
- **Missing frontend:** run `npm ci` and `npm run build` in `frontend` during setup.
- **Pending migration:** make a verified backup, read release notes and run migrations
  against the intended database. Never skip or edit applied migration history.
- **Port in use:** stop the previous SmartShop terminal or choose `start.ps1 -Port 8766`
  and open that loopback URL. Do not kill an unidentified process.
- **Setup denied:** use the local token for the first setup. An existing shop means
  initialization is complete; log in with its owner account.
- **Insufficient stock/cash:** inspect the saved stock/cash movements and correct the
  source through an authorized transaction; do not overwrite ledger rows.

For updates, keep the source release/version and database backup, stop SmartShop,
install reviewed pinned dependencies into the project environment, apply
migrations, rebuild local assets, run checks/tests on an isolated copy, and
restart. Do not expose the local application or database to public networks.
