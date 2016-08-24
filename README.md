Ответы
======

**1**

**3**
На такой таблице приведенный запрос будет работать медленно, например
для __тракторист - женщина__ или __нянечка - мужчина__. 
Поскольку в таблице фактически два варианта `occupation` - `sex`, 
и множество зап исей по ним, планировщик выберет использование
seqscan. В случае тракториста-женщины будут просканированы все записи,
чтобы убедиться, что условие не выполняется.
Можно отключить seqscan (если есть индексы по jccupation и sex).

set enable_seqscan = false;
PREPARE foo (text, text) AS 
SELECT 'found' where exists (
    SELECT * FROM employee 
    WHERE occupation = $1 AND sex = $2
);
EXPLAIN ANALYZE
EXECUTE foo('buldozerist', 'woman');



