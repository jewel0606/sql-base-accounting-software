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

### 7. Profit Distribution Analysis

```sql
SELECT 
    login,
    ticket,
    open_time,
    profit,
    SUM(profit) OVER (
        PARTITION BY login 
        ORDER BY open_time 
        ROWS UNBOUNDED PRECEDING
    ) as cumulative_profit,
    ROW_NUMBER() OVER (
        PARTITION BY login 
        ORDER BY open_time
    ) as trade_sequence
FROM analyst
ORDER BY login, open_time;
```
### Result 
<img width="633" height="210" alt="image" src="https://github.com/user-attachments/assets/0c59a70e-2d61-4de9-8397-f37fab105b34" />

### 8. Performance by Trade Direction

```sql
SELECT 
    type as trade_direction,
    COUNT(*) as total_trades,
    ROUND(SUM(profit), 2) as total_profit,
    ROUND(AVG(profit), 2) as avg_profit,
    SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) as winning_trades,
    ROUND(
        SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
    ) as win_rate_percentage,
    ROUND(AVG(pips), 2) as avg_pips_per_trade
FROM analyst
GROUP BY type
ORDER BY total_profit DESC;
```
### Result 
<img width="1011" height="219" alt="image" src="https://github.com/user-attachments/assets/362436bf-8466-4b0f-b53d-65b9eb33dc15" />

### 9. Risk-Reward Analysis
```sql
SELECT 
    login,
    COUNT(*) as total_trades,
    ROUND(AVG(ABS(open_price - stop_loss)), 2) as avg_risk_per_trade,
    ROUND(AVG(ABS(take_profit - open_price)), 2) as avg_reward_per_trade,
    ROUND(
        AVG(ABS(take_profit - open_price)) / NULLIF(AVG(ABS(open_price - stop_loss)), 0), 2
    ) as avg_risk_reward_ratio,
    ROUND(SUM(profit), 2) as total_profit
FROM analyst
WHERE stop_loss IS NOT NULL AND take_profit IS NOT NULL
GROUP BY login;
```
### Result - In attached Pdf named - 9. Risk-Reward Analysis

### 10. Monthly Performance Trend

```sql
SELECT 
    TO_CHAR(open_time, 'YYYY-MM') as trading_month,
    COUNT(*) as trades_count,
    ROUND(SUM(profit), 2) as monthly_profit,
    ROUND(AVG(profit), 2) as avg_trade_profit,
    SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) as winning_trades,
    ROUND(
        SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
    ) as monthly_win_rate
FROM analyst
GROUP BY TO_CHAR(open_time, 'YYYY-MM')
ORDER BY trading_month;

```
### Result 
<img width="850" height="365" alt="image" src="https://github.com/user-attachments/assets/1fc74b5f-f583-4f9e-a27d-8194d16ab795" />+

### 11. Volume vs Profit Analysis

```sql

SELECT 
    CASE 
        WHEN volume >= 200 THEN 'High Volume (200+)'
        WHEN volume >= 180 THEN 'Medium Volume (180-199)'
        ELSE 'Low Volume (<180)'
    END as volume_category,
    COUNT(*) as trade_count,
    ROUND(AVG(profit), 2) as avg_profit,
    ROUND(SUM(profit), 2) as total_profit,
    ROUND(AVG(pips), 2) as avg_pips
FROM analyst
GROUP BY 
    CASE 
        WHEN volume >= 200 THEN 'High Volume (200+)'
        WHEN volume >= 180 THEN 'Medium Volume (180-199)'
        ELSE 'Low Volume (<180)'
    END
ORDER BY avg_profit DESC;
```
### Result 
<img width="702" height="176" alt="image" src="https://github.com/user-attachments/assets/c9685785-291a-4ba7-aab5-d474cb9a4e7c" />

### 12. Top Performing and Worst Performing Trades

```sql

-- Top Performing Trades

SELECT 
    login,
    ticket,
    symbol,
    type,
    open_time,
    profit,
    pips,
    volume
FROM analyst
ORDER BY profit DESC
LIMIT 5;

--'Top 5 Worst Performing Trades' as analysis_type;

SELECT 
    login,
    ticket,
    symbol,
    type,
    open_time,
    profit,
    pips,
    volume
FROM analyst
ORDER BY profit ASC
LIMIT 5;
```
### Result 
Top Performing Trades
<img width="1036" height="260" alt="image" src="https://github.com/user-attachments/assets/62e339df-064a-48a5-bf2f-a312131576d2" />

Top 5 Worst Performing Trades
<img width="1017" height="260" alt="image" src="https://github.com/user-attachments/assets/54569d64-d05b-48de-9102-40cffcec469e" />

### 13. Overall Portfolio Performance Summary

```sql
SELECT 
    COUNT(*) as total_trades,
    COUNT(DISTINCT login) as total_traders,
    ROUND(SUM(profit), 2) as total_profit,
    ROUND(AVG(profit), 2) as avg_profit_per_trade,
    SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) as total_winning_trades,
    SUM(CASE WHEN profit <= 0 THEN 1 ELSE 0 END) as total_losing_trades,
    ROUND(
        SUM(CASE WHEN profit > 0 THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2
    ) as overall_win_rate,
    ROUND(MAX(profit), 2) as best_single_trade,
    ROUND(MIN(profit), 2) as worst_single_trade,
    ROUND(SUM(pips), 2) as total_pips_earned
FROM analyst;
```
### Result 

<img width="1447" height="78" alt="image" src="https://github.com/user-attachments/assets/33829b82-a074-4c61-863f-5692cb216699" />

### 14. Performance Consistency Analysis

```sql
SELECT 
    login,
    COUNT(*) as total_trades,
    ROUND(STDDEV(profit), 2) as profit_volatility,
    ROUND(AVG(profit), 2) as avg_profit,
    CASE 
        WHEN STDDEV(profit) < 2000 THEN 'Consistent Trader'
        WHEN STDDEV(profit) < 4000 THEN 'Moderate Volatility'
        ELSE 'High Volatility Trader'
    END as consistency_rating
FROM analyst
GROUP BY login;
```
### Result - In attached Pdf named - 14. Performance Consistency Analysis

