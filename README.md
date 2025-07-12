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
<img width="616" height="443" alt="image" src="https://github.com/user-attachments/assets/077562e9-e8f4-4e34-8d96-26a83527ed3f" />


### 1.3 Trial Balance result in sql

<img width="722" height="322" alt="image" src="https://github.com/user-attachments/assets/b70dc917-449f-4a66-affe-9ac7c58e1464" />




### Ledger SQL Query

The following SQL view calculates a Ledger aggregating journal entries, grouped by account code and type. It also includes a final row for the grand total.

```sql
WITH ledger AS (
  SELECT
    jnl.transaction_id,
    jnl.transaction_date,
    coa.account_name,
    coa.account_type,
    CASE WHEN jnl.dc = 'debit' THEN jnl.amount ELSE 0 END AS debit,
    CASE WHEN jnl.dc = 'credit' THEN jnl.amount ELSE 0 END AS credit,
    SUM(
      CASE
        WHEN coa.account_type IN ('asset', 'expense') THEN
          CASE WHEN jnl.dc = 'debit' THEN jnl.amount
               WHEN jnl.dc = 'credit' THEN -jnl.amount
               ELSE 0
          END
        ELSE
          CASE WHEN jnl.dc = 'debit' THEN -jnl.amount
               WHEN jnl.dc = 'credit' THEN jnl.amount
               ELSE 0
          END
      END
    ) OVER (ORDER BY jnl.transaction_date, jnl.transaction_id) AS balance
  FROM jnl
  JOIN chart_of_accounts coa ON jnl.account_code = coa.account_code
  WHERE coa.account_name = 'Accounts Payable'
)

-- Main ledger with summary total
SELECT transaction_id, transaction_date, account_name, debit, credit, balance
FROM ledger

UNION ALL

SELECT
  NULL AS transaction_id,
  NULL AS transaction_date,
  'Total' AS account_name,
  SUM(debit),
  SUM(credit),
  CASE
    WHEN MAX(account_type) IN ('asset', 'expense') THEN SUM(debit) - SUM(credit)
    ELSE SUM(credit) - SUM(debit)
  END
FROM ledger
;
```
