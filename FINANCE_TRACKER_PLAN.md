# Notion Finance Tracker — Build Plan

## Goal
Every month, answer three questions from one screen:
1. **How much is coming in?** (Income)
2. **How much is already spoken for?** (Recurring expenses)
3. **How much can I still spend?** (Personal / flexible spending left)

And every week: "Is this a tight week or a flexible week?"

---

## Decision: ONE database, not two

Keep a single **Transactions** database with a **Type** column. Reasons:

- **Notion dashboards are built on one database.** A dashboard view's widgets all read from
  that same database, so one database is what makes the single, clean dashboard possible.
  Two databases would force separate linked charts, which is the layout you don't want.
- "Left to spend" is just **Income − Expenses**. With one database that's one sum.
  With two, Notion can't subtract across databases without rollups and workarounds.
- One place to enter data, one weekly routine, one place to reconcile against the bank.

### Sign convention
You're right that the math should treat income as + and expenses as −. **But don't type the minus sign.**
- Always enter **Amount** as a positive number (what's printed on the receipt).
- A formula, **Net Amount**, flips the sign based on Type (Income = +, expenses = −).
- Why: typing negatives by hand is the #1 source of errors, and spending charts read better with positive bars.
- Refunds/returns: enter as a **negative** Amount on the expense row type. This is the only time you type a minus.

---

## Database schema: 💳 Transactions

| Property | Type | Purpose |
|---|---|---|
| Name | Title | Short label: "Trader Joe's – groceries" |
| Date | Date | Purchase date (not the date you entered it) |
| Amount | Number ($) | Always positive |
| Type | Select | **Income · Recurring Expense · Personal Expense** (your 3 buckets) |
| Category | Select | What it was for (fixes current "Catgory" typo) |
| Merchant | Select | Trader Joe's, Shell, etc. |
| Status | Select | **Planned** · **Cleared** — planned bills/paychecks vs. ones that hit the bank |
| Account | Select | Checking · Credit Card · Debit Card |
| Receipt ID | Text | Groups split lines from one receipt (e.g. `TJ-0926`) |
| Receipt | Files | Photo of the receipt (on the first line of a split) |
| Reconciled | Checkbox | Ticked once matched to the bank statement |
| Notes | Text | Optional |
| Net Amount | Formula | `if(Type == "Income", Amount, -Amount)` |
| Week | Formula | Monday of the purchase week, for weekly charts |
| Month | Formula | `formatDate(Date, "YYYY-MM")` for monthly grouping |

### Categories (grouped by Type)
- **Income:** Paycheck, Side Income, Refund/Reimbursement, Other Income
- **Recurring Expense:** Rent/Housing, Utilities, Phone & Internet, Insurance, Subscriptions,
  Car Payment, Debt Payment, Savings (pay yourself first)
- **Personal Expense:** Groceries, Household & Cleaning, Personal Care & Hygiene, Dining Out,
  Gas & Transportation, Health, Entertainment, Shopping, Gifts, Misc

Keep the list short. More than ~15 categories and entry gets slow and charts get noisy.

---

## Split receipts (the Trader Joe's problem)

One $50 trip = **one row per category**, all with the same Date, Merchant, and Receipt ID:

| Name | Amount | Category | Receipt ID |
|---|---|---|---|
| Trader Joe's – groceries | 32.40 | Groceries | TJ-0926 |
| Trader Joe's – hygiene | 10.85 | Personal Care & Hygiene | TJ-0926 |
| Trader Joe's – cleaning | 6.75 | Household & Cleaning | TJ-0926 |

- Notion AI reads the photo and creates the lines (as you do now).
- **Accuracy check:** the lines must add up to the bank charge ($50.00). Tax goes on the biggest line
  (or spread proportionally). Group the "By Receipt" view by Receipt ID to see each total next to the bank.
- Tick **Reconciled** once the total matches the bank.

---

## Recurring expenses & income: plan the month in advance

Use **Notion repeating database templates**:
- One template per bill (Rent, Phone, Netflix…) set to **repeat monthly** on its due date, pre-filled
  with Type = Recurring Expense, Status = Planned, Amount, Category, Merchant.
- One template for your paycheck, repeating on your pay schedule, Status = Planned.
- On the 1st of each month the month's bills and paychecks already exist as **Planned** rows.
  When one clears the bank, flip it to **Cleared** (and fix the amount if it changed).

This is what makes the "tight week / flexible week" call possible: you see in advance which weeks
the big bills land and when the paychecks arrive.

---

## The dashboard (ONE Notion dashboard view on Transactions)

Dashboard-level filter: **Date is This month** (switch to last month when reviewing).

**Row 1 – The month in four numbers**
- **Income** — sum of Amount, Type = Income
- **Recurring Bills** — sum of Amount, Type = Recurring Expense (planned + cleared)
- **Personal Spent** — sum of Amount, Type = Personal Expense
- **Left to Spend** — sum of Net Amount (green ≥ 0, red < 0)

**Row 2 – Weekly view**
- **Spending by week** — stacked column: x = Week, sum of Amount, stacked by Type
  (Recurring vs. Personal). Tall recurring bars = tight week.

**Row 3 – Where the money goes**
- **Personal spend by category** — donut, **sum of Amount** (not row count), Type = Personal Expense
- **Recurring bills by category** — bar, sum of Amount

**Row 4 – What's coming**
- **Upcoming (Planned)** — table: Status = Planned, sorted by Date. Bills still to come this month.

### Fixes to the current dashboard
- "Spend By Category" and "Spending by Merchant" **count rows** instead of summing dollars, and include income.
- "Spending by Merchant" filter is a hard-coded merchant list; new merchants won't show up.
- "Safe to Spend" sums **all time**, not this month.
- "Weekly Spending" groups by **day**, not week.
- Sample rows (Whole Foods, Netflix, Local Bistro…) should be cleared before real data.

---

## Supporting views (on the same database)
- **📥 Inbox** — Reconciled is unchecked, newest first. Your weekly work queue.
- **🧾 By Receipt** — grouped by Receipt ID, totals shown.
- **🔁 Bills** — Type = Recurring Expense, grouped by Month.
- **💰 Income** — Type = Income.
- **📋 All** — everything, newest first.

---

## Weekly routine (~20 min, same day each week)
1. Upload receipt photos → Notion AI creates split lines.
2. Open the bank app; add anything without a receipt (gas, online, subscriptions).
3. In **Inbox**, match each bank charge to its row(s); tick Reconciled.
4. Flip any Planned bills/paychecks that cleared to Cleared.
5. Glance at the dashboard: Left to Spend ÷ weeks left in the month = this week's personal budget.

---

## Best practices
- **Never log credit-card payments as expenses.** The purchases were already logged; the payment is a
  transfer. Logging both double-counts. Same for moving money between your own accounts.
- **Pay yourself first.** Put savings in as a recurring item on payday so "Left to Spend" is honest.
- **Sinking funds.** For yearly/irregular costs (car registration, Amazon Prime, gifts), divide by 12
  and add a monthly recurring "set-aside" so they don't ambush a single month.
- **Keep a buffer.** Don't plan "Left to Spend" to $0; leave ~5–10% of income as cushion.
- **Date = purchase date**, not entry date, so weekly charts stay correct even with weekly entry.
- **Review monthly.** On the 1st, check last month: which category went over? Adjust next month.

---

## Execution phases
1. **Schema cleanup** — rename Catgory → Category, rename Transaction Type → Type with the 3 options,
   add Status / Receipt ID / Receipt / Reconciled / Week / Month, update Net Amount formula, refresh categories.
2. **Clear sample data** (after confirmation).
3. **Recurring templates** — one per bill + paycheck, set to repeat.
4. **Rebuild the dashboard view** with the widgets above (replace the current one).
5. **Supporting views** — Inbox, By Receipt, Bills, Income, All.
6. **Home page layout** — dashboard front and center; "How to enter data" toggle with the weekly routine.
7. **Test** with one real week of data and one split receipt; verify totals against the bank.

---

## Build log (2026-10-01)

Built on the existing 💳 Transactions database (no new database).

**Schema**
- Renamed `Catgory` → `Category`, `Transaction Type` → `Type` (Income · Recurring Expense · Personal Expense).
- Added: Status (Planned/Cleared), Receipt ID, Receipt (files), Reconciled (checkbox).
- Formulas: Net Amount (sign flip), Week Of, Month, This Month, plus month-aware helpers
  Income (Month), Recurring (Month), Personal (Month), Left to Spend (Month).
  The helpers exist because the Notion API can't set a relative "this month" date filter on dashboard widgets.
- Sample rows kept and re-mapped to the new Types/Categories. Added a 3-line Trader Joe's split example (TJ-1001).
- Recurring templates skipped: bills are entered manually (user's choice).

**Dashboard (same single dashboard view)**
- Row 1: 💵 Income · 🔁 Recurring Bills (this month)
- Row 2: ✅ Left to Spend (this month)
- Row 3: 🛒 Personal Spending by Category (donut; center total = personal spent) · 📅 Spending Over Time (stacked Recurring vs Personal)
- The old "Weekly Spending" widget was dropped by Notion during an API update and can't be re-added via API.
  Its replacement is the Spending Over Time widget in row 3.

**Tabs:** 📥 Inbox (to reconcile) · 💵 Income · 📋 All Transactions · 🔁 Bills · 🧾 By Receipt

**Manual finishing touches (Notion UI only):**
1. Spending Over Time → chart settings → X-axis Date → group by **Week**.
2. Number tiles → show as **$** if they display plain numbers.
3. Optional: + Add widget → number tile on "Personal (Month)" next to Left to Spend.
4. Optional: Left to Spend tile → conditional color (green ≥ 0, red < 0).
