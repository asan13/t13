Ответы
======

**1**

**2**

```sql
CREATE OR REPLACE FUNCTION brackets_balance(str text) RETURNS bool
LANGUAGE plpgsql
AS $$
BEGIN

    str = regexp_replace(str, '[^()]', '', 'g');

    IF str IS NULL OR length(str) = 0 THEN
        RETURN true;
    END IF;

    RETURN CASE WHEN EXISTS (
        WITH RECURSIVE a AS (
            SELECT regexp_replace(str, '\(\)', '') rep
            UNION ALL
            SELECT regexp_replace(rep, '\(\)', '') FROM a WHERE rep ~ '\(\)'
        )
        SELECT * FROM a WHERE rep = ''
    ) THEN true ELSE false END;

END;
$$;
```


**3**
На такой таблице приведенный запрос будет работать медленно, например
для _тракторист - женщина_ или _нянечка - мужчина_. 
Поскольку в таблице фактически два варианта `occupation` - `sex`, 
и множество зап исей по ним, планировщик выберет использование
seqscan. В случае тракториста-женщины будут просканированы все записи,
чтобы убедиться, что условие не выполняется.
Можно отключить seqscan (если есть индексы по jccupation и sex).

```sql
set enable_seqscan = false;
PREPARE foo (text, text) AS 
SELECT 'found' where exists (
    SELECT * FROM employee 
    WHERE occupation = $1 AND sex = $2
);
EXPLAIN ANALYZE
EXECUTE foo('buldozerist', 'woman');
```



