### Определение

Результаты двух запросов могут быть объединены с помощью операций над множествами: объединение (union), пересечение (intersection) и разность (difference).
### Пример

``` postgreSQL
'query1' UNION [ALL] 'query2';
'query1' INTERSECT [ALL] 'query2';
'query1' EXCEPT [ALL] 'query2';
```

где _`query1`_ и _`query2`_ - это запросы, которые могут использовать любые из рассмотренных до этого момента функций.

## UNION, INTERSECT, EXPECT
### Простыми словами

Соединение (*UNION*) эффективно добавляет результат _`query2`_ к результату _`query1`_ (хотя нет гарантии, что это будет порядок, в котором строки фактически возвращаются). Кроме того, оно удаляет дублирующиеся строки из результата, так же, как *DISTINCT*, если не используется *UNION ALL*.

*INTERSECT* возвращает все строки, которые присутствуют как в результате `query1`, так и в результате _`query2`_. Дублирующиеся строки удаляются, если не используется *INTERSECT ALL*.

*EXCEPT* возвращает все строки, которые есть в результате _`query1`_, но отсутствуют в результате _`query2`_. (Иногда это называется _разницей_ между двумя запросами). Опять же, дубликаты удаляются, если не используется *EXCEPT ALL*.
