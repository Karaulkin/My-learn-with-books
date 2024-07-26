***
Предложение *HAVING* используется для фильтрации результатов группировки. *WHERE* используется для применения условий к колонкам, а *HAVING* — к группам, созданным с помощью *GROUP BY*.

  

`HAVING` должно указываться после `GROUP BY`, но перед `ORDER BY` (при наличии).

  

```postgresql
SELECT col1, col2, ...colN
FROM table1, table2, ...tableN
[WHERE condition]
GROUP BY col1, col2, ...colN
HAVING condition
ORDER BY col1, col2, ...colN;
```

***


