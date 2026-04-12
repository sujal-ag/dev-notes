# SQL Notes — Revision Sheet

---

## NULL
```sql
WHERE col IS NULL
WHERE col IS NOT NULL
IFNULL(col, 0)        -- null hoga toh 0 return karo
```
> `WHERE col != 2` — NULLs nahi pakdega, IS NULL use karo

---

## Aggregates
```sql
SUM(col)
AVG(col)
COUNT(*)              -- sabhi rows
COUNT(col)            -- non-null rows
COUNT(DISTINCT col)   -- unique values
MAX(col) / MIN(col)
ROUND(AVG(col), 2)    -- 2 decimal places
SUM(case when x = 'a' then z else y end)
```

---

## GROUP BY + HAVING
```sql
GROUP BY col
HAVING COUNT(*) > 5   -- groups pe filter
```
> WHERE → grouping se pehle | HAVING → grouping ke baad

---

## JOINs
```sql
INNER JOIN  -- sirf matching rows
LEFT JOIN   -- left table ki saari rows, right mein null agar match nahi
```

### ⚠️ LEFT JOIN trap
```sql
-- LEFT JOIN rehta hai (nulls preserve)
LEFT JOIN t2 ON t1.id = t2.id AND t2.col = 'X'

-- INNER JOIN ban jaata hai (nulls filter ho jaate hain)
LEFT JOIN t2 ON t1.id = t2.id
WHERE t2.col = 'X'
```

---

## DATEDIFF
```sql
DATEDIFF(date1, date2)   -- date1 - date2 in days
DATE_ADD(date, INTERVAL 1 DAY)
DATE_SUB(date, INTERVAL 30 DAY)
DATE_FORMAT(date, '%Y-%m')
```

---

## LIKE
```sql
LIKE 'A%'    -- A se shuru
LIKE '%son'  -- son pe khatam
LIKE '%ar%'  -- ar contain kare
```

---

## UNION
```sql
UNION      -- duplicates remove karta hai
UNION ALL  -- duplicates rakhta hai (faster)
```

---

## Subquery
```sql
WHERE dept_id IN (SELECT id FROM Department WHERE name = 'Sales')
```

---
