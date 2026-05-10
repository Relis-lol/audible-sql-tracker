# SQL Query Examples – Audible SQL Tracker

Purpose: Small copy-paste ready SQL examples for exploring the audiobook database in SQLiteStudio or any SQLite tool.

---

## 1. Total number of titles by source

Shows how many entries exist in Audible, BookBeat and physical books.

```sql
SELECT source, COUNT(*) AS total_titles
FROM (
    SELECT source FROM audible_audiobooks
    UNION ALL
    SELECT source FROM bookbeat_audiobooks
    UNION ALL
    SELECT source FROM physical_books
)
GROUP BY source
ORDER BY total_titles DESC;
```

---

## 2. Total number of all titles

Counts every title across all available title tables.

```sql
SELECT COUNT(*) AS total_titles
FROM (
    SELECT titel AS title FROM audible_audiobooks
    UNION ALL
    SELECT title FROM bookbeat_audiobooks
    UNION ALL
    SELECT title FROM physical_books
);
```

---

## 3. Audible titles by payment type

Shows how many Audible titles were bought with credits or cash.

```sql
SELECT typ, COUNT(*) AS total_titles
FROM audible_audiobooks
GROUP BY typ
ORDER BY total_titles DESC;
```

---

## 4. Most purchased / listened authors

Combines Audible, BookBeat and physical books.

```sql
SELECT author, COUNT(*) AS total_titles
FROM (
    SELECT author FROM audible_audiobooks
    UNION ALL
    SELECT author FROM bookbeat_audiobooks
    UNION ALL
    SELECT author FROM physical_books
)
WHERE author IS NOT NULL
  AND TRIM(author) <> ''
GROUP BY author
ORDER BY total_titles DESC, author ASC
LIMIT 20;
```

---

## 5. Audible purchases per year

```sql
SELECT strftime('%Y', purchase_date) AS year,
       COUNT(*) AS audible_titles
FROM audible_audiobooks
GROUP BY year
ORDER BY year;
```

---

## 6. BookBeat listens per year

```sql
SELECT strftime('%Y', listen_date) AS year,
       COUNT(*) AS bookbeat_titles
FROM bookbeat_audiobooks
GROUP BY year
ORDER BY year;
```

---

## 7. Audible purchases per month

```sql
SELECT strftime('%Y-%m', purchase_date) AS month,
       COUNT(*) AS audible_titles
FROM audible_audiobooks
GROUP BY month
ORDER BY month;
```

---

## 8. BookBeat listens per month

```sql
SELECT strftime('%Y-%m', listen_date) AS month,
       COUNT(*) AS bookbeat_titles
FROM bookbeat_audiobooks
GROUP BY month
ORDER BY month;
```

---

## 9. Combined activity per month

Combines Audible purchases and BookBeat listens into one monthly overview.

```sql
SELECT month, source, COUNT(*) AS total_titles
FROM (
    SELECT strftime('%Y-%m', purchase_date) AS month, 'Audible' AS source
    FROM audible_audiobooks

    UNION ALL

    SELECT strftime('%Y-%m', listen_date) AS month, 'BookBeat' AS source
    FROM bookbeat_audiobooks
)
GROUP BY month, source
ORDER BY month, source;
```

---

## 10. Authors with multiple titles

Useful for finding repeated authors or possible series clusters.

```sql
SELECT author, COUNT(*) AS total_titles
FROM (
    SELECT author FROM audible_audiobooks
    UNION ALL
    SELECT author FROM bookbeat_audiobooks
    UNION ALL
    SELECT author FROM physical_books
)
WHERE author IS NOT NULL
  AND TRIM(author) <> ''
GROUP BY author
HAVING COUNT(*) > 1
ORDER BY total_titles DESC, author ASC;
```

---

## 11. Search titles by author

Replace `Jack Campbell` with any author name.

```sql
SELECT source, title, author, date
FROM (
    SELECT source, titel AS title, author, purchase_date AS date
    FROM audible_audiobooks

    UNION ALL

    SELECT source, title, author, listen_date AS date
    FROM bookbeat_audiobooks

    UNION ALL

    SELECT source, title, author, NULL AS date
    FROM physical_books
)
WHERE author LIKE '%Jack Campbell%'
ORDER BY date, title;
```

---

## 12. Search titles by possible series name

There is no separate `series` column in the database, so this searches inside the title text.
Replace `Lumera` with any series keyword.

```sql
SELECT source, title, author, date
FROM (
    SELECT source, titel AS title, author, purchase_date AS date
    FROM audible_audiobooks

    UNION ALL

    SELECT source, title, author, listen_date AS date
    FROM bookbeat_audiobooks

    UNION ALL

    SELECT source, title, author, NULL AS date
    FROM physical_books
)
WHERE title LIKE '%Lumera%'
ORDER BY date, title;
```

---

## 13. Total Audible cash spending in Euro

Only counts Audible rows where a Euro price is entered.

```sql
SELECT ROUND(SUM(CAST(NULLIF(TRIM(price_euro), '') AS REAL)), 2) AS audible_cash_spent_euro
FROM audible_audiobooks
WHERE price_euro IS NOT NULL
  AND TRIM(price_euro) <> '';
```

---

## 14. Total Audible credit purchases in Euro

Shows how much was spent buying Audible credits.

```sql
SELECT ROUND(SUM(price_euro), 2) AS audible_credit_spent_euro,
       SUM(credit_number) AS total_credits_bought,
       ROUND(SUM(price_euro) / SUM(credit_number), 2) AS average_price_per_credit
FROM audible_credit_log;
```

---

## 15. Audible credits by type

```sql
SELECT typ,
       SUM(credit_number) AS total_credits,
       ROUND(SUM(price_euro), 2) AS total_spent_euro,
       ROUND(SUM(price_euro) / SUM(credit_number), 2) AS average_price_per_credit
FROM audible_credit_log
GROUP BY typ
ORDER BY total_spent_euro DESC;
```

---

## 16. Total BookBeat subscription spending in Euro

```sql
SELECT ROUND(SUM(price_euro), 2) AS bookbeat_spent_euro
FROM bookbeat_abo_log;
```

---

## 17. Total physical book spending in Euro

The column name contains a space and symbol, so it must be wrapped in double quotes.

```sql
SELECT ROUND(SUM(CAST(NULLIF(TRIM("price - €"), '') AS REAL)), 2) AS physical_books_spent_euro
FROM physical_books
WHERE "price - €" IS NOT NULL
  AND TRIM("price - €") <> '';
```

---

## 18. Combined Euro spending overview

```sql
SELECT 'Audible cash purchases' AS category,
       ROUND(SUM(CAST(NULLIF(TRIM(price_euro), '') AS REAL)), 2) AS total_euro
FROM audible_audiobooks
WHERE price_euro IS NOT NULL
  AND TRIM(price_euro) <> ''

UNION ALL

SELECT 'Audible credit purchases' AS category,
       ROUND(SUM(price_euro), 2) AS total_euro
FROM audible_credit_log

UNION ALL

SELECT 'BookBeat subscription' AS category,
       ROUND(SUM(price_euro), 2) AS total_euro
FROM bookbeat_abo_log

UNION ALL

SELECT 'Physical books' AS category,
       ROUND(SUM(CAST(NULLIF(TRIM("price - €"), '') AS REAL)), 2) AS total_euro
FROM physical_books
WHERE "price - €" IS NOT NULL
  AND TRIM("price - €") <> '';
```

---

## 19. Most expensive Audible cash titles

```sql
SELECT titel AS title,
       author,
       purchase_date,
       CAST(NULLIF(TRIM(price_euro), '') AS REAL) AS price_euro
FROM audible_audiobooks
WHERE price_euro IS NOT NULL
  AND TRIM(price_euro) <> ''
ORDER BY price_euro DESC
LIMIT 20;
```

---

## 20. Most recent entries

```sql
SELECT source, title, author, date
FROM (
    SELECT source, titel AS title, author, purchase_date AS date
    FROM audible_audiobooks

    UNION ALL

    SELECT source, title, author, listen_date AS date
    FROM bookbeat_audiobooks
)
ORDER BY date DESC
LIMIT 25;
```
