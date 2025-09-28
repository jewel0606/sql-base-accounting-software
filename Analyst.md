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

