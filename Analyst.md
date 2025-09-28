### 1. Data Structure Overview

```sql
SELECT 'Data Structure Analysis' as analysis_type;

SELECT 
    COUNT(*) as total_records,
    COUNT(DISTINCT login) as unique_logins,
    COUNT(DISTINCT symbol) as unique_symbols,
    MIN(open_time) as earliest_trade,
    MAX(close_time) as latest_trade
FROM analyst;
```
### Result

<img width="784" height="78" alt="image" src="https://github.com/user-attachments/assets/d5eb000d-7a62-4631-b69d-eaa6c0161c0f" />

### 2. Data Quality Assessment & Missing Values Check

```sql
SELECT 
    'Missing Values Analysis' as check_type,
    SUM(CASE WHEN login IS NULL THEN 1 ELSE 0 END) as missing_login,
    SUM(CASE WHEN ticket IS NULL THEN 1 ELSE 0 END) as missing_ticket,
    SUM(CASE WHEN symbol IS NULL THEN 1 ELSE 0 END) as missing_symbol,
    SUM(CASE WHEN profit IS NULL THEN 1 ELSE 0 END) as missing_profit,
    SUM(CASE WHEN pips IS NULL THEN 1 ELSE 0 END) as missing_pips
FROM analyst;
```
### Result
<img width="829" height="77" alt="image" src="https://github.com/user-attachments/assets/d24f7e9f-bcd1-4113-8974-9103f6687511" />

### 3. Duplicate Records Check

```sql

SELECT 
    ticket,
    COUNT(*) as occurrence_count
FROM analyst
GROUP BY ticket
HAVING COUNT(*) > 1
```
### Result
<img width="158" height="626" alt="image" src="https://github.com/user-attachments/assets/2cc56ff8-f53b-467a-8b2d-e98ff12076b4" />

### 4. Data Distribution Analysis

```sql
SELECT 
    symbol,
    COUNT(*) as trade_count,
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM analyst), 2) as percentage
FROM analyst
GROUP BY symbol
ORDER BY trade_count DESC;
```
### Result
<img width="238" height="591" alt="image" src="https://github.com/user-attachments/assets/eb5884c9-b7b6-44d9-8d3d-31e8d9c625d1" />   

<img width="206" height="522" alt="image" src="https://github.com/user-attachments/assets/1f2d372f-27e3-4539-8cdf-a561663f5c47" />

### 5.Trade Type Distribution

```sql
SELECT 
    type,
    COUNT(*) as trade_count,
    ROUND(AVG(profit), 2) as avg_profit_per_type,
    ROUND(SUM(profit), 2) as total_profit_per_type
FROM analyst
GROUP BY type;
```
### Result
<img width="655" height="220" alt="image" src="https://github.com/user-attachments/assets/b3f7beed-d1ff-4650-a5cf-716b2af70481" />

### 6. PROFITABILITY ANALYSIS

```sql
SELECT 
    login,
    COUNT(*) as total_trades,
    ROUND(SUM(profit), 2) as total_profit,
    ROUND(AVG(profit), 2) as avg_profit_per_trade,
    ROUND(MIN(profit), 2) as worst_trade,
    ROUND(MAX(profit), 2) as best_trade,
    SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) as winning_trades,
    SUM(CASE WHEN profit <= 0 THEN 1 ELSE 0 END) as losing_trades,
    ROUND(
        SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
    ) as win_rate_percentage
FROM analyst
GROUP BY login
ORDER BY total_profit DESC;
```
### Result - In attached Pdf named - 6. PROFITABILITY ANALYSIS




