# URPSA Financial Management System — Build Spec

**Status: CONFIRMED** (user directive: build end to end, no further questions)
**Source:** `URPSA_Financial_Management_System_2026-2027 (1).xlsx` — 20 sheets
**Administration period:** 1 Oct 2026 – 30 Sep 2027 · **Currency:** NGN (₦)

---

## 1. Goal and roles

A treasurer's ledger for the University of Lagos Urban and Regional Planning Students' Association
(URPSA) that replaces the Excel workbook with a real database, server-enforced posting
rules, and one-click statutory reports.

**Roles:** a single **treasurer** login. Every row is scoped to its owner (`ctx.user.id`).
The workbook's three-officer model (Financial Secretary + Treasurer edit, President
view-only) is **out of scope** — the platform serves one authenticated user per account,
so the app ships one login and the officer names are recorded in Settings as metadata.

## 2. Data model (13 tables, all owner-scoped)

| # | Table | Key fields | Source sheet |
|---|---|---|---|
| 1 | `settings` | org_name, period_start, period_end, opening_balance, currency, financial_secretary, treasurer, president, member_id_rule | Control Centre |
| 2 | `accounts` | code, name, class (Income/Expense), typical_use, active | Chart of Accounts |
| 3 | `cash_accounts` | code, name, type (Bank/Cash), details, opening_balance, opening_date, active | Accounts |
| 4 | `transactions` | txn_id, type, date, category_code, cash_account_id, payee, member_id, reference, amount, evidence_id, activity_id, approval_status, approved_by, notes | Master Transactions |
| 5 | `journal_lines` | transaction_id, account_code, cash_account_id, debit, credit, memo | *(derived — double entry)* |
| 6 | `members` | member_id, admission_year, matric_number, full_name, level, phone, email, status | Members |
| 7 | `dues` | dues_ref, member_id, dues_type, due_date, amount_assessed | Dues |
| 8 | `sponsorship` | sponsorship_id, sponsor, purpose, contact, commitment_date, amount_pledged, stage | Sponsorship |
| 9 | `activities` | activity_id, name, start_date, end_date, budget_category, approved_budget, approver, approval_date | Activities-Budgets |
| 10 | `liabilities` | liability_id, payee, description, activity_id, date_incurred, due_date, amount_committed, approval_status | Liabilities & Commitments |
| 11 | `assets` | asset_id, description, category, serial, identifier, custodian, location, purchase_date, purchase_txn_id, cost, condition, last_verified | Assets |
| 12 | `evidence` | evidence_id, transaction_id, doc_type, description, file_path, date_received, submitted_by, reviewed_by, status, exception_reason | Evidence Tracking |
| 13 | `reconciliations` | month, cash_account_id, statement_date, statement_balance, ledger_balance, reviewed_by, status, notes | Reconciliation |

**Double entry.** The workbook is single-entry (Income/Expense + category). The app keeps
that register as the input surface and **derives** `journal_lines` server-side, exactly as
the society app does, so trial balance / financial position / cash flow are real:

- Income → **Dr** cash account, **Cr** income account (4001–4005)
- Expense → **Dr** expense account (5001–5007), **Cr** cash account
- Opening balance → **Dr** cash account, **Cr** 3001 Accumulated Fund

## 3. Pages and routes (12)

| Route | Access | Purpose |
|---|---|---|
| `/login` | public | Sign in |
| `/signup` | public | Create the treasurer account |
| `/` | login | Dashboard — income, expenditure, net, cash, control strip |
| `/transactions` | login | Ledger + double-entry entry form with evidence upload |
| `/transactions/:id` | login | Entry detail, lines, evidence, reverse |
| `/budget` | login | Budget vs actual by activity |
| `/sponsorship` | login | Sponsor pipeline and receipts |
| `/sponsorship/:id` | login | Pledge detail |
| `/members` | login | Members and dues collection |
| `/registers` | login | Assets & liabilities registers |
| `/reports` | login | One-click reports + print/PDF + CSV |
| `/settings` | login | Organisation, accounts, chart of accounts |

## 4. Permission rules

- Every read and write filters by `ctx.user.id`. No cross-account visibility.
- Posting rules enforced **server-side** in one engine every write passes through:
  ≥2 lines; debits = credits to the kobo; closed periods refused; expense above the
  evidence threshold refused without a verified document; opening balances posted once.
- The browser never computes a balance — all figures derive from `journal_lines`.

## 5. Visual direction

Deep teal + amber, dense ledger tables, tabular-numeral alignment — the same design
language as the Student Society Finance app. Teal `hsl(183 62% 22%)` primary, amber
`hsl(38 92% 50%)` accent, warm off-white surface, 6px radius, Inter-style type scale
with a small-caps label treatment for table headers.

## 6. Out of scope

- **No email** — no reminders, no "resend receipt", no scheduled jobs.
- **No bank-feed import** — every row is entered or seeded.
- **No multi-user roles** — one treasurer login; the President's view-only access is not
  enforceable by the platform and is not faked.
- **No server-side PDF** — print/PDF is the browser's print dialog; CSV is generated client-side.
- **No OCR** — evidence is an uploaded file, not a parsed document.

## 7. Seed data

The workbook is a **blank template**: 20 formula-driven sheets, zero transactions, zero
members, zero dues, zero sponsorship rows. The only real figures are the opening balance
(₦104,099.50 on BANK-01) and the structure.

Seeded as **real structural data**: the 12-account chart of accounts, BANK-01 / CASH-01,
the Control Centre settings, the Fresher's Orientation activity (FRESH-ORI-2026-27) and
the Conference activity (CONF-2026).

Seeded as a **representative dataset** (clearly labelled in the app): a full administration
year of transactions, members, dues, sponsorship, assets and liabilities, so the dashboard
and every report show real, reconciling figures rather than an empty shell.
