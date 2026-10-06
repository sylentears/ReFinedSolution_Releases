# ReFined Solution — User & Operations Guide

*What is in the app, what you need before it runs, how to install it, how to set it up and how the day-to-day work flows.*

---

## 1. What ReFined Solution is

ReFined Solution is a **Windows desktop business and accounting system** for shops that sell goods and/or
repair devices (mobile / computer / electronics workshops). One program covers four things that normally
need four programs:

| Area | What it does |
|---|---|
| **Sales** | Invoices, invoice returns (credit notes), pending (unpaid) invoices, customer balances and statements, cash receipts |
| **Purchases** | Supplier bills, purchase returns, supplier payments, payables and statements |
| **Inventory** | Categories, products, stock levels and valuation, stock ledger, period-wise inventory report, manual stock adjustments |
| **Service (repair)** | Job cards (service orders) with device details, complaint/diagnosis, technician, priority, parts used, costs, advance and balance, printable job card |
| **Accounting** | Automatic double-entry posting for every document, chart of accounts, posting rules, fiscal periods and period lock, journal day book, expense entry, general ledger, trial balance, profit & loss, balance sheet |
| **Control** | Users, roles, per-module permissions, audit log (every action and every denied action), settings |

At a glance:

* **Everything works offline.** No internet connection is required for the application, the database or the
  licence — activation is done by exchanging a code with your vendor by e-mail or WhatsApp.
* **Six languages** (English, Arabic, Urdu, Dutch, French, Spanish), switchable at any time; Arabic and Urdu
  are laid out right-to-left, with a Nastaliq font for Urdu.
* **Six colour themes** (Modern, Elegant, Midnight, Emerald, Rose, Slate) — default **Rose**.
* **Per-company currency symbol and name.**
* **Audit-ready:** every create/edit/delete/post/print and every refused action is written to the audit log
  with the user name, the screen and the time.
* Your business data lives in a **Microsoft SQL Server** database; the application is the client that reads
  and writes it. ReFined Solution never needs you to write SQL.

---

## 2. What you need before installing

### 2.1 Windows

| Item | Requirement |
|---|---|
| Operating system | Windows 10 (build 1607 or later) or Windows 11 — **64-bit** |
| Rights to install | A local administrator account (**per-machine install**; after that, ordinary users run the app) |
| Free disk space | Roughly 250 MB for the program (the MSI itself is about 79 MB) plus room for the database |
| .NET runtime | **Not needed** — the shipped MSI is self-contained and brings everything it needs |

### 2.2 Microsoft SQL Server

The database is the one component you must have ready:

* **SQL Server 2016 or newer** (Express, Standard, Enterprise or Developer). SQL Server Express is free and
  is enough for a single shop or workshop.
* It can run **on the same computer** as ReFined Solution (the usual setup) or **on another computer in the
  shop network** (LAN).
* The app **creates the database for you** — the SQL login you give it needs the right to create a database
  (`dbcreator`, or an equivalent administrator login). If your IT department prefers to create the database
  itself, that is fine too: point the app at the existing empty database and it builds everything inside it.
* Recommended: a **dedicated SQL login for ReFined Solution** (not the built-in `sa` account) with a strong
  password. Its details are kept in the app's own configuration (see §4).

### 2.3 What ReFined Solution builds by itself

You never run a script or a migration by hand. At **every start** the application checks the database it
opened and quietly adds whatever is missing — so a database created by version 1.1 is brought up to date
when version 1.3 opens it, **without touching the data already inside**:

* the tables of all modules,
* the standard settings of the company,
* the security catalogue (modules, actions, the six roles and their permissions),
* the chart of accounts, the posting rules and the fiscal periods.

---

## 3. Installing the application

### 3.1 Normal install (interactive)

1. Copy `ReFinedSolution-<version>-<MonYyyy>-x64.msi` to the computer (or to a shared folder) and
   **double-click it**. If your vendor published a SHA-256 checksum, compare it before installing.
2. The Windows Installer wizard opens (English):

   | Page | What to do |
   |---|---|
   | **Welcome** | Continue. The wizard looks for an older ReFined Solution on this machine and tells you whether it will be an *upgrade* or a *fresh install*. |
   | **Licence agreement** | Accept. |
   | **Setup type** | *Typical* (recommended) / *Custom* (choose the folder yourself) / *Complete*. |
   | **Destination folder** | Default: `C:\Program Files\ReFined Solution`. |
   | **Progress → Finish** | The Finish page says either "… has been installed" or "Upgrade complete … version X". |

3. The installer also creates:
   * a **Start Menu** shortcut for all users and a **Desktop** shortcut,
   * the folder `C:\ProgramData\ReFinedSolution` with write rights for ordinary users (the licence store —
     it is deliberately kept even if you uninstall the app later),
   * an entry in *Add or remove programs* with the product name, version and vendor.

4. Nothing else is asked. The installer does **not** install or change SQL Server, and it never overwrites
   the connection settings file (`appsettings.json`), so an upgrade keeps the settings of the site.

### 3.2 Upgrading

There is only one thing to do: **run the new MSI**. The package decides for itself what to do — an older
version was found → upgrade; nothing was found → fresh install; a newer version is already installed → it
refuses with a clear message and leaves the machine alone. Running the *same* version again performs a
repair.

* The old version is removed **inside the same transaction** as the new install, so if the new install fails
  the old version comes back automatically.
* If ReFined Solution is running during an upgrade it is closed and started again afterwards (Restart
  Manager). In a silent deployment this happens without a prompt.

### 3.3 Installing across many computers (silent)

```bat
:: silent install or upgrade, with a log file
msiexec /i ReFinedSolution-1.3.5-Oct2026-x64.msi /qn /l*v "%TEMP%\rs-install.log"

:: silent uninstall
msiexec /x ReFinedSolution-1.3.5-Oct2026-x64.msi /qn /l*v "%TEMP%\rs-uninstall.log"
```

The repository also ships a helper that **decides first and then installs** (handy for GPO / Intune /
scheduled tasks, because it reports what it is about to do):

```powershell
pwsh -File installer\install.ps1 -DryRun                              # only print the decision
pwsh -File installer\install.ps1 -Silent -LogPath C:\Temp\rs-upgrade.log
```

| Decision | Meaning |
|---|---|
| `INSTALL` | No ReFined Solution found — fresh install |
| `UPGRADE` | An older version was found — the same `msiexec /i` upgrades it |
| `REPAIR` | Exactly the same version is already installed |
| `BLOCKED` | A newer version is already installed — nothing is changed |

Exit codes: `0` = success (or dry run), `1` = `msiexec` failed, `3` = blocked. The graphical wizard is always
available by running `msiexec /i <msi>` without `/qn`, or by simply double-clicking the MSI.

### 3.4 Uninstalling

Use *Add or remove programs* or `msiexec /x <msi>`. The program folder is removed.
**Your business data is not touched** (it lives in SQL Server) and the licence folder in `ProgramData` is
kept on purpose, so a re-install keeps that computer's activated state.


---

## 4. First run: from the licence gate to the sign-in window

The first start of a new installation runs in a fixed order. You only ever see the screens that need you.

| Step | What you see | What happens |
|---|---|---|
| **0 · Licence gate** | "Activate ReFined Solution" | A **7-day trial** starts on the first run. You can press **Skip** and keep working; the trial banner shows the days left (§12). Send the request code to your vendor whenever you are ready to buy. |
| **1 · Splash** | A progress window that names each step | Steps: configuration → connection → preparing database → company → ready. If something goes wrong, the reason is shown here in plain language. |
| **2 · Database** | Either nothing, or the **Database** window | See §4.1. |
| **3 · Preparing the database** | Splash steps with module names | Tables, settings, security catalogue, chart of accounts, posting rules and fiscal periods are created/updated. |
| **4 · Company** | The **Company setup** wizard | Shown only while the database has no company yet (§5). |
| **5 · Sign-in** | "Sign in" window | Language, theme and currency of the company are already applied. |

### 4.1 The Database window (only when it is needed)

A fresh installation is shipped with a default connection in the file `appsettings.json` next to the program
(it looks like this — replace the placeholders with your own values):

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=<server>;Database=<database>;User Id=<login>;Password=<password>;TrustServerCertificate=true;"
  }
}
```

* If that default **answers**, the app adopts it silently, saves it and never asks you anything.
* If it does **not** answer (server down, database missing, wrong login), the **Database** window opens with
  the values already filled in. Fill in / correct:

  | Field | Notes |
  |---|---|
  | **Server name / instance** | `localhost`, `.`, `.\SQLEXPRESS`, or `SERVERNAME\INSTANCE` for a server on the network |
  | **Database name** | Type it, or press **Load databases** and pick from the list |
  | **Windows Authentication** | Tick it to sign in to SQL Server with the Windows user — no SQL password needed |
  | **SQL username / password** | Untick the box above and enter the dedicated SQL login |
  | **Test connection** | Confirms the server and login |
  | **Create database** | Creates the database if it does not exist yet, saves the connection and continues to the company setup |

* A successful test saves the profile to `%AppData%\ReFinedSolution\appconfig.json` (the password inside it is
  encrypted with Windows DPAPI for the current Windows user — it is not stored in plain text on disk).
* On later starts there is **no database window at all**. To change the database, use **Change database…** on
  the sign-in window, or **File ▸ Database settings…** inside the app. The app restarts after a change.
* If the saved database disappears (deleted, renamed, server moved), the app does not die: the splash explains
  the problem and the same window opens so you can repair the connection. If the database name is right but
  the database is gone, it asks *"The database … is not on … yet. Create it now?"*.

---

## 5. Company setup wizard

The wizard appears when the database has no company yet — and **the sign-in window is not shown before it is
finished**. Four short steps; everything you enter can be changed later in *Administration ▸ Settings*.

### Step 1 · Company

| Field | Notes |
|---|---|
| **Company name *** | Required. Also used to suggest the company file name and the database name |
| Address / City / Phone / E-mail | Printed in the header of invoices, job cards and reports |
| **Tax / NTN number** | Printed on documents |
| **Financial year starts** | The month the fiscal year begins |
| Notes | Free text |
| **Language / Theme / Currency** | The language the screens open in, the colour theme, and the currency symbol + name used for every amount. These three are saved in this company's settings |

### Step 2 · Database

"Where must the data live?" — the company's database. You do not need to know SQL: if the database does not
exist yet it is created here.

* **Server name / instance *** and **Database name *** — press **Test server** to check the server and
  **Create database** to create it (an existing database is used as it is).
* Tick **Windows Authentication** to use the Windows login instead of a SQL username/password.
* If your SQL Server keeps the database on this same machine, the suggested server name can be left as it is.

### Step 3 · Administrator

The **first and only account created for you**, with every right (all modules, all actions). Later you create
the rest of the users yourself and give each one a role (§6).

| Field | Notes |
|---|---|
| **User name *** | The wizard suggests `admin` |
| Full name / E-mail | Optional, shown in the audit log and the user list |
| **Password *** / **Repeat password *** | The wizard's suggestion is `Admin@123`; choose your own. The policy requires at least 8 characters with one letter and one digit |
| **Ask for a new password at the first sign-in** | Recommended — the account must then change its password the first time it signs in |

### Step 4 · Finish

* **Company file (*.rfc)** — a small file holding the company details and the address of its database. Default
  location: `Documents\ReFined Solution\<company name>.rfc`. **Keep it together with your backups**: on another
  computer, *File ▸ Open company data…* opens the same company with this one file.
* Press **Create company**: the database is created if needed, the tables, roles and settings are built, your
  administrator is created, and the company file is written — then the **sign-in window** opens.

A new installation starts **empty**: no demo products, vendors, customers or invoices. You add master data and
users yourself (§7 and §8).


---

## 6. Signing in, users, roles and permissions

### 6.1 Signing in

Enter the **user name** and **password** of your account and press **Login**. The sign-in window also has a
**Change database…** link (for switching this computer to another company database).

The password policy (minimum length, letters and digits) and the lockout after repeated wrong attempts come
from the company settings — the defaults are at least 8 characters with one letter and one digit, and **five
failed attempts lock the account for 30 minutes**. An administrator can change both in
*Administration ▸ Settings*.

### 6.2 Roles that are shipped ready

| Role | Intended for | Rights in one line |
|---|---|---|
| **Admin** | Owner / IT | Everything: every module, every action |
| **Manager** | Business head | Sees and creates everywhere, approves; **no delete, no posting** |
| **Accountant** | Books and GL | Full posting/unpost/cancel in accounting; view-only on masters |
| **Cashier** | Counter | Dashboard, invoices, invoice returns, customer payments, customers, service orders; read-only products |
| **Technician** | Field engineer | Dashboard, customers (view/add), **service orders**, counter invoices and receipts — and nothing else |
| **Auditor** | Read-only | Can see/print/export everything, cannot change anything |

Roles are permission sets over a simple matrix: **Modules × Actions**, where the actions are view, create,
edit, delete, print, export, post, unpost, cancel and approve. A user's role decides which menu items are
visible *and* which buttons work; the app enforces the same check when an action is carried out, and a refused
attempt is recorded in the audit log.

### 6.3 Administration screens

| Screen | Purpose |
|---|---|
| **Users & Permissions** | Create/edit users, set role, reset password, force a password change, activate/deactivate |
| **Roles** | See and adjust each role's permission set |
| **Modules & Actions** | See the catalogue of screens/actions; modules can be enabled or disabled |
| **Audit Log** | Full trail: who did what, when, in which module, with before/after values, plus refused attempts |
| **Settings** | Searchable list of all company settings (language, theme, currency, password policy, print sizes, database, and so on). Double-click a row to edit its value, then **Save Selected**; **Reset Defaults** brings a value back to the shipped default |

> Company switching, backup, restore and database settings are **administrator tools** — they are meant for the
> owner or IT, not for the counter staff.

---

## 7. Tour of the app — every screen

The shell has a **navigation row** at the top (File, the module menus, Themes, Help and a "More" menu that
collects menus when the window is narrow), a **sidebar** on the left (Dashboard, Service Orders, Reports) and a
**status footer** showing the signed-in user, the module count, the database and the product version.

### 7.1 File menu (always first)

| Item | Purpose |
|---|---|
| **New company…** | Creates another company with its own database (limited by edition, §11) |
| **Open company data…** | Lists the companies of this computer; opens the selected one from its company file (`*.rfc`) |
| **Backup…** | Makes a full backup of the open database (the SQL Server writes the file) |
| **Restore…** | Puts a backup back |
| **Database settings…** | Shows / changes the server, login and database of this machine |
| **Exit** | Closes the application |

### 7.2 Sales — the *Invoice* menu

| Screen | Purpose |
|---|---|
| **Invoice** | Sales invoice register (search, filter, open, print, return) |
| **New Invoice** | Create a new invoice |
| **Pending Invoices** | Invoices with an outstanding balance |
| **Return Invoice** | Sales return / credit note |

### 7.3 Customers — the *Customer* menu

| Screen | Purpose |
|---|---|
| **Customer list** | Customer master (add, edit, balances) |
| **Receive Payment** | Record a customer receipt; the double entry is posted for you and the slip can be printed |
| **Customer Balances** | Outstanding balance per customer |
| **Customer Ledger** | Full statement with opening and closing balance for a period |

### 7.4 Purchases — the *Purchase* menu

| Screen | Purpose |
|---|---|
| **Purchases** | Supplier bills (stock comes in here) |
| **Purchase Returns** | Returns to the supplier |
| **Purchase Report** | Purchase register for a period — paid and outstanding |


### 7.5 Vendors — the *Vendor* menu

| Screen | Purpose |
|---|---|
| **Vendors List** | Supplier master |
| **Pay Bills** | Pay a supplier bill (double entry posted for you) |
| **Vendor Payments** | Supplier payment register |
| **Vendor Balances** | Outstanding payable per supplier |
| **Vendor Ledger** | Full statement with opening and closing balance for a period |

### 7.6 Inventory — the *Inventory* menu

| Screen | Purpose |
|---|---|
| **Categories** | Product categories |
| **Products** | Items, purchase/sale pricing and stock |
| **Stock** | Stock levels and valuation |
| **Inventory Report** | Opening / in / out / closing per item for a period |
| **Stock Ledger** | Movement ledger of one item, period by period |
| **Stock Adjustment** | Manual count / damage / found — quantity change with a reason **and** a GL entry |

### 7.7 Accounting — the *Setup*, *Transactions* and *Reports* menus

| Menu | Screen | Purpose |
|---|---|---|
| Setup | **Chart of Accounts** | Accounts and opening balances |
| Setup | **Posting Rules** | Which document line goes to which account (document × line role → account) |
| Setup | **Fiscal Periods** | Period lock and closing; a locked period refuses new postings |
| Transactions | **Journal Voucher** | Manual journal entry *(screen is present but not yet released — it appears dimmed)* |
| Transactions | **Day Book** | All vouchers of a day/period, draft and posted |
| Transactions | **Customer Payments** | Customer receipts register |
| Transactions | **Expense Entry** | Record a cash or bank expense — the double-entry voucher is posted for you |
| Reports | **General Ledger** | Account ledger with running balance |
| Reports | **Trial Balance** | Trial balance as of a date |
| Reports | **Profit & Loss** | Income, cost of goods sold, expenses and net profit for a period |
| Reports | **Balance Sheet** | Assets, liabilities and equity as of a date |
| Reports | **Expense Report** | Period expenses by account and by paid-from |

### 7.8 The sidebar

| Item | Purpose |
|---|---|
| **Dashboard** | Welcome line, session/company badge, KPI tiles, **sales for the last 12 months** (settled vs unpaid), **service orders by status**, and **recent activity** straight from the audit trail. The footer shows the open database |
| **Service Orders** | The job-card register (§8.5) |
| **Reports** | Sales, stock and control reports (the classic report screen) |

### 7.9 Themes and Help

* **Themes** — one click applies a palette immediately and saves it in the company settings (the palette in use
  is marked *Current*).
* **Help** — *Help / Quick start* (the same short guide as in *About ▸ Help*), *About ReFined Solution* (product
  version, licence state on this machine, machine code, activation folder, "Copy details for support"), and
  *Activate this machine…*.

### 7.10 Administration — the *Admin* menu

**Users & Permissions · Roles · Modules & Actions · Audit Log · Settings** — see §6.3.


---

## 8. Daily operations — how the work flows

### 8.1 Once, at the beginning: master data

1. **Categories** → add your groups (e.g. Accessories, Spare Parts, Devices).
2. **Products** → add each item with its purchase price / cost and its sale price. Items you sell or use as
   spare parts belong here, because stock, cost and profit all come from this screen.
3. **Customer list** → add your customers. An invoice always names a customer, so create a record such as
   "Walk-in customer" once and pick it for over-the-counter sales.
4. **Vendors List** → add your suppliers.
5. **Chart of Accounts** → the 30 standard accounts are already there; adjust them and enter opening balances if
   you are starting in the middle of a year (opening balances are entered here, as accounts, so the books stay
   balanced).
6. **Users & Permissions** → create an account per person and give each one a role (§6).

### 8.2 Buying stock (purchases)

1. **Purchase ▸ Purchases ▸ New**.
2. Choose the supplier, the bill date and the bill number; add one row per item (product, quantity, unit cost,
   discount as needed).
3. **Save.** Stock comes in at the cost you paid, and the double entry is posted automatically
   (inventory increases, the supplier is credited).
4. When you pay the supplier, use **Vendor ▸ Pay Bills** — that posts the payment (bank/cash decreases, the
   supplier is settled) and reduces the outstanding balance.
5. Watch **Vendor Balances** (what you still owe) and **Purchase Report** (what was bought this month).

### 8.3 Selling (invoices)

1. **Invoice ▸ New Invoice** (or **New Invoice** on the *Invoice* menu).
2. Fill in the header: **customer** (required — picked from the customer list), invoice date, due date,
   **payment type** (Cash / Credit / Partial), remarks.
3. Add the lines: pick a **product** — the sale price fills in automatically — then the description, quantity
   and unit price. An empty row is ignored; a fresh row appears as soon as you pick a product.
4. Check the totals: **sub total, line discount, tax, invoice discount, freight, paid, grand total**.
5. **Save & Close.** What happens on save:

   * the goods leave stock at the **weighted average cost** (one stock movement per line),
   * the accounting voucher is written **once**: cash or receivables on one side; sales, tax, freight and
     discount on the other; and the cost side (cost of goods sold / inventory),
   * the customer's balance is updated,
   * **Invoice #** is generated for you, and the whole thing is recorded in the audit log.

6. Print the invoice from the invoice register (there is a print preview; the company name, address, phone,
   e-mail and tax number from the company setup appear in the header).
7. **Statuses:** `Draft` (nothing posted yet), `Posted`, `Partial` (partly paid) and `Paid`.
   * Only a **Draft** can be edited or deleted.
   * A posted invoice is corrected with **Return Invoice** (credit note), which puts the goods back into stock
     and reverses the posting — the original invoice is never silently changed.
8. Use **Pending Invoices** to see everything still unpaid, and **Customer Ledger** to print a statement.

### 8.4 Money in and money out

| Task | Where | What it does |
|---|---|---|
| Customer pays (cash, bank, cheque, online) | **Customer ▸ Receive Payment** | Posts the receipt (cash/bank up, receivables down), reduces the customer's balance, and prints a payment slip |
| You pay a supplier | **Vendor ▸ Pay Bills** | Posts the payment (cash/bank down, payables down) |
| Petty / shop expense | **Accounting ▸ Transactions ▸ Expense Entry** | Records a cash or bank expense and posts the voucher for you |
| Bank charges / discounts on a receipt | same screens | Maps to the bank-charges and discount accounts through the posting rules |


### 8.5 Service work — the job card

This is the part a repair workshop lives in: **Service Orders** (side bar) → **➕ New Order**.

**The job card at a glance**

| Block | Fields |
|---|---|
| Device & customer | **Order #** (generated), customer (pick from the list or type a walk-in name and phone), **priority** (Low / Normal / High / Urgent), **status** |
| Device | **Brand ***, **Model ***, serial number, accessories received, **Under Warranty** |
| Diagnosis & costs | **Complaint (problem reported) ***, **Diagnosis** (the technician's finding), technician, estimated cost, approved cost, final cost, **advance paid**, **balance** (calculated for you), notes |
| Dates | Received date, promised date, delivery date |
| Parts used | **➕ Add Part** — product, quantity, unit price per line |

**How the work flows in real life**

1. The customer hands in the device. Create the job card, tick whether it is under warranty, write down the
   accessories, and give the customer the printed job card (**print** in the register) as their receipt of the
   device. An advance payment can be recorded in *Advance Paid*; the *Balance* line updates itself.
2. Work through the status pipeline: `Received → Diagnosed → In Progress → Waiting Parts → Ready → Delivered`
   (or `Cancelled`). Update the diagnosis, the technician and the approved cost as you go and press
   **Save & Close** again.
3. When the device is handed back, mark it **Delivered** (the delivery date is filled with today) and print the
   job card again for the customer's signature.

**What happens to stock and books**

* Saving a job card **withdraws the listed parts from stock** at their weighted average cost — the stock ledger
  gets one movement per part, labelled with the order number (the withdrawal is written only once per job card,
  so saving the same card twice never doubles it).
* **Editing a job card later does not withdraw more stock.** Parts are taken out when the card is *first*
  saved; a part added afterwards is recorded on the card (and on the printed card) but no further stock
  movement is created. In practice: add the parts to the card before you press Save the first time.
* Because the parts leave stock without an accounting entry of their own, the job card's *balance* is a
  **request / billing amount for the counter**, not a posted sale. Turning a finished job card into a proper
  invoice (revenue + cost + customer balance in the books) is the next planned step — see Appendix B.
* Job cards cannot be deleted unless the status is `Received` or `Cancelled` — that protects the history.

### 8.6 Stock adjustment, period lock and the reports to watch

| Task | Where | Notes |
|---|---|---|
| Physical count, damage, shrinkage or found stock | **Inventory ▸ Stock Adjustment** | Quantity changed with a reason; the GL entry is posted (inventory vs stock adjustment) |
| Close a month/quarter and lock it | **Accounting ▸ Setup ▸ Fiscal Periods** | A locked period refuses new postings; the lock asks for confirmation |
| See whether the books are balanced | **Trial Balance** (debit = credit), **Day Book** (all vouchers) | The quick daily check |
| Profit for the month | **Profit & Loss** (revenue − cost of goods sold − expenses) | Comes from the same postings as the documents |
| Balance sheet / stock value | **Balance Sheet**, **Stock** report, **Inventory Report**, **Stock Ledger** | The inventory account (1040) and the stock ledger should tell the same story |
| Sales analysis | **Reports** (sidebar) and **Dashboard** charts | Settled vs unpaid sales, service orders by status |


---

## 9. The money side — what is posted for you

You do not type journal entries for normal business. Each document posts its own double entry using the
**Posting Rules** (document × line role → account), which you can see and change in
*Accounting ▸ Setup ▸ Posting Rules*.

| Document | Debit | Credit |
|---|---|---|
| Sales invoice (cash) | Cash 1010 | Sales 4010 / Tax 2020 / Freight income 4090 |
| Sales invoice (credit) | Accounts receivable 1030 | Sales 4010 / Tax 2020 |
| Sales invoice — discount | Discount allowed 5070 | (reduces the receivable / cash side) |
| Sales invoice — cost side | Cost of goods sold 5010 | Inventory 1040 |
| Invoice return (credit note) | Sales 4010 / Tax 2020 reversal | Receivable 1030 or Cash 1010 |
| Purchase bill | Inventory 1040 | Accounts payable 2010 |
| Purchase return | Accounts payable 2010 | Inventory 1040 |
| Customer receipt | Cash 1010 / Bank 1020 | Accounts receivable 1030 |
| Supplier payment | Accounts payable 2010 | Cash 1010 / Bank 1020 |
| Expense entry | The expense account you choose | Cash 1010 / Bank 1020 |
| Stock adjustment | Inventory 1040 or Stock adjustment 5095 | the other one of the pair |

Things worth knowing:

* Posting happens when the document is saved with a status that means "real" (`Posted`, `Paid`, `Partial`); a
  **Draft** changes nothing in stock or in the books.
* Posting is **idempotent**: saving or re-saving the same document does not create a second voucher. Every
  voucher carries its source (document type + document id), which is also how the app knows that a document has
  already been posted.
* Every voucher can be traced back to its document, and every document to its voucher, in the **Day Book** and
  the **Audit Log**.
* The bank/cash account used by a receipt depends on the payment method (cash, bank, cheque, online), which is
  mapped in the posting rules / payment methods.

---

## 10. Backups, restore and moving to another computer

| Task | Where | Notes |
|---|---|---|
| Full backup of the open database | **File ▸ Backup…** | The backup runs **inside SQL Server**, so the suggested folder is the folder of the database files; you only pick the path. Do this every day |
| Put a backup back | **File ▸ Restore…** | Administrator tool; the app confirms before overwriting the current data |
| Keep the company file safe | the `*.rfc` file from the setup wizard | It holds the company details and the address of its database — store it with the backup |
| Server-level backup | your SQL Server / your IT | A maintenance plan on SQL Server gives you a second, independent safety net |

**Moving the shop to a new computer**

1. Install ReFined Solution on the new computer (§3).
2. Make sure the SQL Server that holds the database is reachable from it (same machine, or the network).
3. Start the app; if the database window appears, point it at the server and the database name (§4.1).
4. If the company file is not on the new machine, copy the `*.rfc` file over and use
   **File ▸ Open company data…**; the company (and its database) opens with its settings, users and history.

**Where things live on the machine**

| Path | Contents |
|---|---|
| `C:\Program Files\ReFined Solution` | The program (and the shipped `appsettings.json`) |
| `%AppData%\ReFinedSolution\appconfig.json` | This Windows user's saved database connection (password encrypted with Windows DPAPI) |
| `%AppData%\ReFinedSolution\startup-error.log` | The last startup/window errors — the first place to look when the app misbehaves |
| `%ProgramData%\ReFinedSolution` | Licence files of this computer (`license.key`, `license.sig`, `license.log`) |
| `Documents\ReFined Solution\<company>.rfc` | Company file(s) of this user |
| Your SQL Server | **All business data** (companies, documents, stock, vouchers, users, audit log) |


---

## 11. More than one company, and the three editions

**File ▸ New company…** creates another company with **its own database** (its own products, customers and
books). **File ▸ Open company data…** switches between the companies registered on that machine. Each company
has its own company file (`*.rfc`), its own settings, language, theme and currency — and its own users.

How many companies a licence allows depends on the edition:

| Edition | Companies | Accounting (GL) screens | Typical user |
|---|---|---|---|
| **Standard** | 1 | **Hidden** — the accounting menu group is not shown at all; sales, purchases, inventory, reports and security work unchanged | A shop that only needs invoicing, stock and reports |
| **Professional** | 5 | Shown | A growing business that wants full books |
| **Enterprise** | 50 | Shown | Several branches / companies on one licence |
| *Trial (no licence yet)* | 50 | Shown | The trial deliberately shows the **full** product so you can evaluate everything |

If you press **File ▸ New company…** in the Standard edition, the app explains the limit and opens the
activation window, whose **EDITIONS — UPGRADE** section describes what Professional and Enterprise include.
**Buying a higher edition does not need a new installation** — the edition travels inside the activation code
you receive.

---

## 12. Licensing: the trial and activation

* A new installation runs a **7-day trial** (the full product). The remaining days are shown in a banner, and
  inside **About**.
* During the trial you can press **Skip** on the activation window and keep working — Skip never extends the
  trial, it only stops the window from standing in your way while trial days remain.
* After the 7 days the activation window is the only way forward (Skip is then disabled).

**To activate**

1. **Help ▸ Activate this machine…** (or the *Activate…* button on the banner).
2. Step 1 — press **Copy** to copy the **request code**. It is calculated once and never changes, so it is safe
   to send by e-mail or WhatsApp (the editable field shows it; **Copy all details** copies everything the vendor
   may ask for).
3. Step 2 — paste the **activation code** (`RF1-…`) you receive and press **Activate**, or import the `.key`
   file you were sent.
4. The app confirms "Activated until …".

Notes:

* Activation is **offline and bound to that computer** (a machine fingerprint, not a serial number you have to
  look up). Moving the licence to another computer means sending that computer's request code and receiving a
  new code.
* Licence files stay in `%ProgramData%\ReFinedSolution` on that machine and cannot usefully be copied to
  another one.
* Errors you may see and what they mean: *"Issued for another machine"* → a code of a different computer was
  pasted; *"System date is behind"* → correct the Windows date and time (a wrong clock also blocks an active
  licence); *trial expired* → request a code.
* The current state (trial / active / expiring / blocked) is always visible in **About**, and the running state
  is re-checked when you sign in.

---

## 13. Troubleshooting and support

| Symptom | What to do |
|---|---|
| The app does not start / closes immediately | Open `%AppData%\ReFinedSolution\startup-error.log` — the last errors with full details are there. Fix the reason shown (most often a database that is not reachable) and start again |
| "Server not found" in the database window | Check that the SQL Server service is running; try `localhost`, `.` or `MACHINE\SQLEXPRESS`; press **Load databases** to prove the login works; a wrong SQL password is the most common cause |
| The database was deleted or the server changed | Start the app: it explains the problem and reopens the database window; enter the new server / database and, if the database is missing, let the app create it |
| Install fails with error **1603** | Close ReFined Solution and retry; for details run the MSI with `/l*v "<log>"` and search near `Return value 3`. The usual cause is a file in use or a missing administrator right |
| Windows SmartScreen warns about the setup file | The MSI is not code-signed (yet). Compare the SHA-256 with the one your vendor published, then *More info ▸ Run anyway* |
| Cannot see the accounting screens | The licence is **Standard** (accounting is not part of it) or your role does not include those modules — check with an Admin account, or ask your vendor for an upgrade |
| A screen or button is missing for a user | The **role** decides that; adjust it in *Administration ▸ Roles* / *Users & Permissions* |
| Numbers in the books look wrong | Check **Trial Balance** (debits must equal credits) and the **Day Book** for the document in question; the **Audit Log** shows who posted or cancelled what |
| The app opened the wrong company / database | Sign-in window → **Change database…**, or **File ▸ Open company data…** (both are administrator tools). The app restarts after a database change |

**Support:** open **Help ▸ About ReFined Solution** and press **Copy details for support** — it copies the
product version, the licence state, the machine code and the request code, which is exactly what your vendor
needs to issue a licence or investigate a problem. Send that text together with the question.


---

## Appendix A · The chart of accounts you start with

Every new company starts with a small, standard chart of accounts. Rename or extend it in
*Accounting ▸ Setup ▸ Chart of Accounts* — automatic posting follows the **Posting Rules**, so changing an
account name or a mapping never needs a code change.

| Code | Account | Type | Used by |
|---|---|---|---|
| 1000 | Assets | group | — |
| 1010 | Cash in Hand | Asset | Cash sales, receipts, expenses |
| 1020 | Bank Account | Asset | Bank transfers, cheque and online receipts |
| 1025 | Cheques in Hand | Asset | Cheques received but not yet deposited |
| 1030 | Accounts Receivable | Asset (control) | Credit sales and customer balances |
| 1040 | Inventory | Asset | Stock value — purchases in, sales/parts out |
| 1050 | Advances & Deposits | Asset | Advance payments |
| 1500 | Furniture & Equipment | Asset | Fixed assets |
| 2000 | Liabilities | group | — |
| 2010 | Accounts Payable | Liability (control) | Supplier bills and payments |
| 2020 | Tax Payable | Liability | Sales tax on invoices |
| 2030 | Salaries Payable | Liability | Unpaid salaries |
| 4000 | Revenue | group | — |
| 4010 | Sales Revenue | Revenue | Goods sold on invoices |
| 4020 | Service Income | Revenue | Service / repair income |
| 4090 | Other Income | Revenue | Freight and miscellaneous income |
| 4095 | Discount Received | Revenue | Supplier discounts |
| 5000 | Expenses | group | — |
| 5010 | Cost of Goods Sold | Expense | Cost of the goods sold |
| 5020 | Salaries & Wages | Expense | Payroll |
| 5030 | Rent | Expense | Shop / office rent |
| 5040 | Utilities | Expense | Electricity, internet |
| 5050 | Freight & Cartage | Expense | Transport |
| 5060 | Office Expenses | Expense | Stationery and miscellaneous |
| 5070 | Discount Allowed | Expense | Customer discounts |
| 5075 | Bank Charges | Expense | Bank and payment fees |
| 5080 | Bad Debts | Expense | Uncollectible amounts |
| 5090 | Round Off | Expense | Rounding differences |
| 5095 | Stock Adjustment | Expense | Count / damage / found differences |

Twelve fiscal periods are created with the company and are managed in *Fiscal Periods* — lock a period once it
has been closed, so that nothing can be posted into it afterwards.


---

## Appendix B · Planned: creating an invoice from a service order

> **This is not part of the current build.** Today a job card withdraws its parts from stock but posts no
> accounting entry (§8.5), and invoices are created by hand. The following describes the intended end-to-end
> operation once the feature is released, so you know what to expect. It is written from the user's point of
> view and ends with the decisions that still have to be taken.

**Intended flow for the counter / technician**

1. **Service Orders** → open the finished job card (status *Ready* or *Delivered*).
2. Press **Create invoice** on the card. The button is only visible to users whose role allows creating
   invoices.
3. The **New Invoice** screen opens, already filled in from the job card:
   * the customer kept on the card (a card written for a walk-in name needs a customer record before it can be
     invoiced),
   * the parts used, with the quantities and prices from the card,
   * the labour / service charge from the job card's approved or final cost,
   * the advance already paid on the card as the paid amount, so the balance follows.
4. Check the amounts, add anything the customer asked for on the spot (an accessory, a protection plan), then
   **Save & Close** — exactly as with a normal invoice.
5. The invoice is posted like any other invoice: revenue on one side, cash / receivables on the other, and the
   cost of the parts on the cost side. Print it and hand it over; take the balance with
   **Customer ▸ Receive Payment** if it is not paid on the spot.
6. Back on the job card, the status can be set to **Delivered** and the card shows that it has been invoiced.

**What will be true for the staff**

* **A job card is invoiced once.** The app keeps the link between the card and the invoice and refuses a second
  invoice for the same card (a credit note is the way back).
* **Parts are not deducted from stock twice.** They left stock when the job card was saved, so an invoice made
  from a job card is a *billing* document for those lines — one stock movement, one cost entry, no double count.
* **Invoiced job cards are marked** in the register, so nobody wonders which devices still owe an invoice.
* Money already collected as an advance on the card is not collected again — it counts as paid on the invoice.

**Open decisions (to be confirmed before release)**

| # | Question | Effect on the daily work |
|---|---|---|
| 1 | Should the **job card keep deducting parts from stock** (the invoice then bills those lines only), or should stock move on the invoice instead? | Decides whether the invoice touches stock at all — and what a job card that is still open does to your stock value |
| 2 | How is the **advance paid on the card** treated on the invoice? | Whether it becomes the invoice's paid amount (simplest for the counter) or is kept as a separate customer credit |
| 3 | Is there a **labour / service charge line** on the job card, or is the invoice's labour line typed by hand? | Whether the technician can price the labour before the invoice exists |
| 4 | Is the conversion a **button on the job card**, a wizard, or an action in the invoice register? | How many clicks the counter needs, and who is allowed to do it |
| 5 | Do job-card part lines need their own free-text **description** carried onto the invoice? | Whether the printed invoice can show a free-text line description instead of only the product name |
| 6 | Are **warranty jobs** invoiced at zero value, or not invoiced at all? | What the customer receives for a free-of-charge repair, and what the books show |

These answers are the only thing missing before the feature can be built; the technical design is already
prepared.

---

*ReFined Solution — User & Operations Guide. Product version 1.3.x (see the footer inside the app, or
About ▸ About ReFined Solution).*

