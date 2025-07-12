# sql-base-accounting-software
SQL-powered accounting backend built with Supabase. Supports real-time generation of Trial Balance, P&amp;L, Balance Sheet, Cash Flow, Ledgers, Tax Reports. Great for SaaS devs, accountants &amp; learners exploring GAAP-compliant reporting.


### 1. Trial Balance 

### 1.1 Trial Balance SQL Query


The following SQL view calculates a Trial Balance by aggregating journal entries, grouped by account code and type. It also includes a final row for the grand total.

```sql
WITH account_summary AS (
  SELECT
    coa.account_type,
    coa.account_sub1,
    coa.account_code,
    coa.account_name,
    SUM(CASE WHEN jnl.dc = 'debit' THEN jnl.amount ELSE 0 END) AS total_debit,
    SUM(CASE WHEN jnl.dc = 'credit' THEN jnl.amount ELSE 0 END) AS total_credit,
    CASE
      WHEN coa.account_type IN ('asset', 'expense') THEN
        SUM(CASE WHEN jnl.dc = 'debit' THEN jnl.amount ELSE -jnl.amount END)
      ELSE
        SUM(CASE WHEN jnl.dc = 'credit' THEN jnl.amount ELSE -jnl.amount END)
    END AS final_balance
  FROM jnl
  JOIN chart_of_accounts coa ON jnl.account_code = coa.account_code
  GROUP BY coa.account_code, coa.account_name, coa.account_type
)
 
SELECT 
  account_type,
  account_name,
  total_debit,
  total_credit,
  final_balance FROM account_summary
union all
 
SELECT
  NULL AS account_type,
  'Grand Total' AS account_name,
  SUM(total_debit),
  SUM(total_credit),
  SUM(final_balance)
FROM account_summary
 
order by account_type;
```


### 1.2 Trial Balance result in sql

<img width="722" height="322" alt="image" src="https://github.com/user-attachments/assets/b70dc917-449f-4a66-affe-9ac7c58e1464" />

### 1.3 Trial Balance full result
<img width="543" height="446" alt="image" src="https://github.com/user-attachments/assets/090eabf7-71ca-47e8-8d4a-e5dc904016a7" />



### 2. financial statement summery

This SQL view generates the full financial statement summary directly from the journal table (jnl). It serves as an integrity check to determine whether the SQL queries are producing accurate financial reports or if the results are potentially misleading.

In accounting terms:

Asset = Liability - Equity - Income + Expense 
Therefore: Asset - Liability - Equity - Income + Expense = 0

This equation should yield zero if all postings are accurate, confirming the structural correctness of the report.

```sql
WITH account_summary AS (
  SELECT
    coa.account_code,
    coa.account_type,
    coa.account_category,
    CASE
      WHEN coa.account_sub1 IS NULL AND coa.account_sub2 IS NULL THEN coa.account_name
      WHEN coa.account_sub1 IS NULL AND coa.account_sub2 IS NOT NULL THEN coa.account_sub2
      ELSE coa.account_sub1
    END AS main_ledger,
    CASE
      WHEN coa.account_sub2 IS NULL THEN coa.account_name
      ELSE coa.account_sub2
    END AS sub_ledger,
    coa.account_name,
    coa.account_sub1,
    coa.account_sub2,
    SUM(CASE WHEN jnl.dc = 'debit' THEN jnl.amount ELSE 0 END) AS total_debit,
    SUM(CASE WHEN jnl.dc = 'credit' THEN jnl.amount ELSE 0 END) AS total_credit,
    CASE
      WHEN coa.account_type IN ('asset', 'expense') THEN
        SUM(CASE WHEN jnl.dc = 'debit' THEN jnl.amount ELSE -jnl.amount END)
      ELSE
        SUM(CASE WHEN jnl.dc = 'credit' THEN jnl.amount ELSE -jnl.amount END)
    END AS final_balance
  FROM jnl
  JOIN chart_of_accounts coa ON jnl.account_code = coa.account_code
  GROUP BY
    coa.account_code,
    coa.account_type,
    coa.account_category,
    coa.account_name,
    coa.account_sub1,
    CASE
      WHEN coa.account_sub1 IS NULL THEN coa.account_name
      ELSE coa.account_sub1
    END,
    CASE
      WHEN coa.account_sub2 IS NULL THEN coa.account_sub1
      ELSE coa.account_sub2
    END
),

fs AS (
  -- Level 0: Totals per account_type
  SELECT
    account_type,
    NULL AS account_category,
    NULL AS main_ledger,
    NULL AS sub_ledger,
    NULL AS account_name,
    SUM(total_debit) AS total_debit,
    SUM(total_credit) AS total_credit,
    SUM(final_balance) AS final_balance,
    0 AS sort_order
  FROM account_summary
  GROUP BY account_type

  UNION ALL

  -- Level 1: Totals per account_category within account_type
  SELECT
    account_type,
    account_category,
    NULL AS main_ledger,
    NULL AS sub_ledger,
    NULL AS account_name,
    SUM(total_debit) AS total_debit,
    SUM(total_credit) AS total_credit,
    SUM(final_balance) AS final_balance,
    1 AS sort_order
  FROM account_summary
  GROUP BY account_type, account_category

  UNION ALL

  -- Level 2: Totals per main_ledger within category
  SELECT
    account_type,
    account_category,
    main_ledger,
    NULL AS sub_ledger,
    NULL AS account_name,
    SUM(total_debit) AS total_debit,
    SUM(total_credit) AS total_credit,
    SUM(final_balance) AS final_balance,
    2 AS sort_order
  FROM account_summary
  GROUP BY account_type, account_category, main_ledger

  UNION ALL

  -- Level 3: Totals per sub_ledger within category
  SELECT
    account_type,
    account_category,
    main_ledger,
    sub_ledger,
    NULL AS account_name,
    SUM(total_debit) AS total_debit,
    SUM(total_credit) AS total_credit,
    SUM(final_balance) AS final_balance,
    3 AS sort_order
  FROM account_summary
  GROUP BY account_type, account_category, main_ledger, sub_ledger

  UNION ALL

  -- Level 4: Full detail level
  SELECT
    account_type,
    account_category,
    main_ledger,
    sub_ledger,
    account_name,
    total_debit,
    total_credit,
    final_balance,
    4 AS sort_order
  FROM account_summary

  ORDER BY
    account_type,
    account_category DESC,
    main_ledger DESC,
    sub_ledger DESC,
    sort_order,
    account_name
)

-- Output Summary and Calculations
SELECT * FROM fs
WHERE account_category IS NULL

UNION ALL

-- Income Statement
SELECT
  'income statement = income - expense' AS account_type,
  NULL AS account_category,
  NULL AS main_ledger,
  NULL AS sub_ledger,
  NULL AS account_name,
  NULL AS total_debit,
  NULL AS total_credit,
  (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'income' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'expense' AND account_category IS NULL
  ) AS final_balance,
  NULL AS sort_order

UNION ALL

-- Balance Sheet
SELECT
  'balance sheet = asset - liability - equity' AS account_type,
  NULL AS account_category,
  NULL AS main_ledger,
  NULL AS sub_ledger,
  NULL AS account_name,
  NULL AS total_debit,
  NULL AS total_credit,
  (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'asset' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'liability' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'equity' AND account_category IS NULL
  ) AS final_balance,
  NULL AS sort_order

UNION ALL

-- Verification: Net Balance
SELECT
  'asset - liability - equity - income + expense = 0' AS account_type,
  NULL AS account_category,
  NULL AS main_ledger,
  NULL AS sub_ledger,
  NULL AS account_name,
  NULL AS total_debit,
  NULL AS total_credit,
  (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'asset' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'liability' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'equity' AND account_category IS NULL
  ) - (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'income' AND account_category IS NULL
  ) + (
    SELECT final_balance
    FROM fs
    WHERE account_type = 'expense' AND account_category IS NULL
  ) AS final_balance,
  NULL AS sort_order;

```
