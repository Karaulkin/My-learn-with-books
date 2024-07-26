***
## CREATE CAST

*CREATE CAST* — создать приведение типов

#### Синтаксис

``` postgresql
CREATE CAST (_`исходный_тип`_ AS _`целевой_тип`_)
    WITH FUNCTION _`имя_функции`_ (_`тип_аргумента`_ [, ...])
    [ AS ASSIGNMENT | AS IMPLICIT ]

CREATE CAST (_`исходный_тип`_ AS _`целевой_тип`_)
    WITHOUT FUNCTION
    [ AS ASSIGNMENT | AS IMPLICIT ]

CREATE CAST (_`исходный_тип`_ AS _`целевой_тип`_)
    WITH INOUT
    [ AS ASSIGNMENT | AS IMPLICIT ]
```


***
### Источник

- [PostgresPro](https://postgrespro.ru/docs/postgresql/9.6/sql-createcast)



