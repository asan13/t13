Ответы
======

* * *
**1**

Я понял задачку так, что надо вернуть  _десять последних постов_, 
а не _по десять последних_ от каждого друга, но на всякий случай 
сделал оба варианта. Первый запрос возвращает 10 последних постов,
второй - по десять последних каждого друга.


```sql
SELECT u.name, p.content, p.added FROM friend f JOIN LATERAL (
    SELECT * FROM post WHERE usr_id = f.friend_usr_id 
    ORDER BY added DESC LIMIT 10
) p ON p.usr_id = f.friend_usr_id 
JOIN usr u ON u.id = f.friend_usr_id
WHERE f.usr_id = 313;
```

```sql
SELECT u.name, b.* FROM (
    SELECT f.friend_usr_id, p.content, p.added 
    FROM friend f JOIN post p ON p.usr_id = f.friend_usr_id
    WHERE f.usr_id = 313 ORDER BY p.added DESC LIMIT 10
) b JOIN usr u ON u.id = b.friend_usr_id;
```

Прверял на таких таблицах
```sql
CREATE TABLE usr (
    id      serial PRIMARY KEY,
    name    text
);

CREATE TABLE friend (
    usr_id          int NOT NULL REFERENCES usr(id),
    friend_usr_id   int NOT NULL REFERENCES usr(id),
    PRIMARY KEY(usr_id, friend_usr_id)
);
CREATE INDEX friend_usr_idx ON friend(friend_usr_id);

CREATE TABLE post (
    id      serial PRIMARY KEY,
    usr_id  int NOT NULL REFERENCES usr(id),
    content text NOT NULL,
    added   timestamp(0) NOT NULL DEFAULT now()
);
CREATE INDEX post_usr_id_idx ON post(usr_id);
CREATE INDEX post_added_idx ON post(added);
```

* * *
**2**

```plpgsql
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

***
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
SELECT 'found' WHERE exists (
    SELECT * FROM employee 
    WHERE occupation = $1 AND sex = $2
);
EXPLAIN ANALYZE
EXECUTE foo('buldozerist', 'woman');
```



