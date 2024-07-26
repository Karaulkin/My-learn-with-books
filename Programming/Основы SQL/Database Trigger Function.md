***
 - [Триггеры при изменении данных](#Триггеры%20при%20изменении%20данных)
 - [Триггеры событий](#Триггеры%20событий)
***
## Триггерные процедуры

В *PL/pgSQL* можно создавать триггерные процедуры, которые будут вызываться при изменениях данных или событиях в базе данных. Триггерная процедура создаётся командой `CREATE FUNCTION`, при этом у функции не должно быть аргументов, а типом возвращаемого значения должен быть `trigger` (для триггеров, срабатывающих при изменениях данных) или `event_trigger` (для триггеров, срабатывающих при событиях в базе). Для триггеров автоматически определяются специальные локальные переменные с именами вида ``TG__`имя`_``, описывающие условие, повлёкшее вызов триггера.
***
#### Триггеры при изменении данных


[Триггер при изменении данных](https://postgrespro.ru/docs/postgrespro/9.6/triggers "Глава 35. Триггеры") объявляется как функция без аргументов и с типом результата `trigger`. Заметьте, что эта функция должна объявляться без аргументов, даже если ожидается, что она будет получать аргументы, заданные в команде `CREATE TRIGGER` — такие аргументы передаются через `TG_ARGV`, как описано ниже.

Когда функция на PL/pgSQL срабатывает как триггер, в блоке верхнего уровня автоматически создаются несколько специальных переменных:

`NEW`

Тип данных `RECORD`. Переменная содержит новую строку базы данных для команд `INSERT`/`UPDATE` в триггерах уровня строки. В триггерах уровня оператора и для команды `DELETE` этой переменной значение не присваивается.

`OLD`

Тип данных `RECORD`. Переменная содержит старую строку базы данных для команд `UPDATE`/`DELETE` в триггерах уровня строки. В триггерах уровня оператора и для команды `INSERT` этой переменной значение не присваивается.

`TG_NAME`

Тип данных `name`. Переменная содержит имя сработавшего триггера.

`TG_WHEN`

Тип данных `text`. Строка, содержащая `BEFORE`, `AFTER` или `INSTEAD OF`, в зависимости от определения триггера.

`TG_LEVEL`

Тип данных `text`. Строка, содержащая `ROW` или `STATEMENT`, в зависимости от определения триггера.

`TG_OP`

Тип данных `text`. Строка, содержащая `INSERT`, `UPDATE`, `DELETE` или `TRUNCATE`, в зависимости от того, для какой операции сработал триггер.

`TG_RELID`

Тип данных `oid`. OID таблицы, для которой сработал триггер.

`TG_RELNAME`

Тип данных `name`. Имя таблицы, для которой сработал триггер. Эта переменная устарела и может стать недоступной в будущих релизах. Вместо неё нужно использовать `TG_TABLE_NAME`.

`TG_TABLE_NAME`

Тип данных `name`. Имя таблицы, для которой сработал триггер.

`TG_TABLE_SCHEMA`

Тип данных `name`. Имя схемы, содержащей таблицу, для которой сработал триггер.

`TG_NARGS`

Тип данных `integer`. Число аргументов в команде `CREATE TRIGGER`, которые передаются в триггерную процедуру.

`TG_ARGV[]`

Тип данных массив `text`. Аргументы от оператора `CREATE TRIGGER`. Индекс массива начинается с 0. Для недопустимых значений индекса ( < 0 или >= `tg_nargs`) возвращается NULL.

Триггерная функция должна вернуть либо `NULL`, либо запись/строку, соответствующую структуре таблице, для которой сработал триггер.

Если `BEFORE` триггер уровня строки возвращает `NULL`, то все дальнейшие действия с этой строкой прекращаются (т. е. не срабатывают последующие триггеры, команда `INSERT`/`UPDATE`/`DELETE` для этой строки не выполняется). Если возвращается не `NULL`, то дальнейшая обработка продолжается именно с этой строкой. Возвращение строки отличной от начальной `NEW`, изменяет строку, которая будет вставлена или изменена. Поэтому, если в триггерной функции нужно выполнить некоторые действия и не менять саму строку, то нужно возвратить переменную `NEW` (или её эквивалент). Для того чтобы изменить сохраняемую строку, можно поменять отдельные значения в переменной `NEW` и затем её вернуть. Либо создать и вернуть полностью новую переменную. В случае строчного триггера `BEFORE` для команды `DELETE` само возвращаемое значение не имеет прямого эффекта, но оно должно быть отличным от `NULL`, чтобы не прерывать обработку строки. Обратите внимание, что переменная `NEW` всегда `NULL` в триггерах на `DELETE`, поэтому возвращать её не имеет смысла. Традиционной идиомой для триггеров `DELETE` является возврат переменной `OLD`.

Триггеры `INSTEAD OF` (это всегда триггеры уровня строк и они могут применяться только с представлениями) могут возвращать NULL, чтобы показать, что они не выполняли никаких изменений, так что обработку этой строки можно не продолжать (то есть, не вызывать последующие триггеры и не считать строку в числе обработанных строк для окружающих команд `INSERT`/`UPDATE`/`DELETE`). В противном случае должно быть возвращено значение, отличное от NULL, показывающее, что триггер выполнил запрошенную операцию. Для операций `INSERT` и `UPDATE` возвращаемым значением должно быть `NEW`, которое триггерная функция может модифицировать для поддержки предложений `INSERT RETURNING` и `UPDATE RETURNING` (это также повлияет на значение строки, передаваемое последующим триггерам, или доступное под специальным псевдонимом `EXCLUDED` в операторе `INSERT` с предложением `ON CONFLICT DO UPDATE`). Для операций `DELETE` возвращаемым значением должно быть `OLD`.

Возвращаемое значение для строчного триггера AFTER и триггеров уровня оператора (BEFORE или AFTER) всегда игнорируется. Это может быть и NULL. Однако в этих триггерах по-прежнему можно прервать вызвавшую их команду, для этого нужно явно вызвать ошибку.

[Пример 39.3](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger#plpgsql-trigger-example "Пример 39.3. Триггерная процедура PL/pgSQL") показывает пример триггерной процедуры в PL/pgSQL.

**Пример 39.3. Триггерная процедура PL/pgSQL**

Триггер, показанный в этом примере, при любом добавлении или изменении строки в таблице сохраняет в этой строке информацию о текущем пользователе и отметку времени. Кроме того, он требует, чтобы было указано имя сотрудника и зарплата задавалась положительным числом.

CREATE TABLE emp (
    empname text,
    salary integer,
    last_date timestamp,
    last_user text
);

CREATE FUNCTION emp_stamp() RETURNS trigger AS $emp_stamp$
    BEGIN
        -- Проверить, что указаны имя сотрудника и зарплата
        IF NEW.empname IS NULL THEN
            RAISE EXCEPTION 'empname cannot be null';
        END IF;
        IF NEW.salary IS NULL THEN
            RAISE EXCEPTION '% cannot have null salary', NEW.empname;
        END IF;

        -- Кто будет работать, если за это надо будет платить?
        IF NEW.salary < 0 THEN
            RAISE EXCEPTION '% cannot have a negative salary', NEW.empname;
        END IF;

        -- Запомнить, кто и когда изменил запись
        NEW.last_date := current_timestamp;
        NEW.last_user := current_user;
        RETURN NEW;
    END;
$emp_stamp$ LANGUAGE plpgsql;

CREATE TRIGGER emp_stamp BEFORE INSERT OR UPDATE ON emp
    FOR EACH ROW EXECUTE PROCEDURE emp_stamp();

  

Другой вариант вести журнал изменений для таблицы предполагает создание новой таблицы, которая будет содержать отдельную запись для каждой выполненной команды INSERT, UPDATE, DELETE. Этот подход можно рассматривать как протоколирование изменений таблицы для аудита. [Пример 39.4](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger#plpgsql-trigger-audit-example "Пример 39.4. Триггерная процедура для аудита в PL/pgSQL") показывает реализацию соответствующей триггерной процедуры в PL/pgSQL.

**Пример 39.4. Триггерная процедура для аудита в PL/pgSQL**

Показанный в этом примере триггер гарантирует, что любое добавление, изменение или удаление строки в таблице `emp` будет зафиксировано в таблице `emp_audit` (для аудита). Также он фиксирует текущее время, имя пользователя и тип выполняемой операции.

CREATE TABLE emp (
    empname           text NOT NULL,
    salary            integer
);

CREATE TABLE emp_audit(
    operation         char(1)   NOT NULL,
    stamp             timestamp NOT NULL,
    userid            text      NOT NULL,
    empname           text      NOT NULL,
    salary integer
);

CREATE OR REPLACE FUNCTION process_emp_audit() RETURNS TRIGGER AS $emp_audit$
    BEGIN
        --
        -- Добавление строки в emp_audit, которая отражает операцию, выполняемую в emp;
        -- для определения типа операции применяется специальная переменная TG_OP.
        --
        IF (TG_OP = 'DELETE') THEN
            INSERT INTO emp_audit SELECT 'D', now(), user, OLD.*;
            RETURN OLD;
        ELSIF (TG_OP = 'UPDATE') THEN
            INSERT INTO emp_audit SELECT 'U', now(), user, NEW.*;
            RETURN NEW;
        ELSIF (TG_OP = 'INSERT') THEN
            INSERT INTO emp_audit SELECT 'I', now(), user, NEW.*;
            RETURN NEW;
        END IF;
        RETURN NULL; -- возвращаемое значение для триггера AFTER игнорируется
    END;
$emp_audit$ LANGUAGE plpgsql;

CREATE TRIGGER emp_audit
AFTER INSERT OR UPDATE OR DELETE ON emp
    FOR EACH ROW EXECUTE PROCEDURE process_emp_audit();

  

У предыдущего примера есть разновидность, которая использует представление, соединяющее основную таблицу и таблицу аудита, для отображения даты последнего изменения каждой строки. При этом подходе по-прежнему ведётся полный журнал аудита в отдельной таблице, но также имеется представление с упрощенным аудиторским следом. Это представление содержит временную метку, которая вычисляется для каждой строки из данных аудиторской таблицы. [Пример 39.5](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger#plpgsql-view-trigger-audit-example "Пример 39.5. Триггер на PL/pgSQL для аудита в представлении") показывает пример триггера на представление для аудита в PL/pgSQL.

**Пример 39.5. Триггер на PL/pgSQL для аудита в представлении**

В этом примере триггер, связанный с представлением, делает это представление изменяемым и гарантирует, что любая команда на добавление, изменение или удаление строки в представлении будет записана для аудита в таблицу `emp_audit`. Также записываются временная метка, имя пользователя и тип выполняемой операции. Представление показывает дату последнего изменения для каждой строки.

CREATE TABLE emp (
    empname           text PRIMARY KEY,
    salary            integer
);

CREATE TABLE emp_audit(
    operation         char(1)   NOT NULL,
    userid            text      NOT NULL,
    empname           text      NOT NULL,
    salary            integer,
    stamp             timestamp NOT NULL
);

CREATE VIEW emp_view AS
    SELECT e.empname,
           e.salary,
           max(ea.stamp) AS last_updated
      FROM emp e
      LEFT JOIN emp_audit ea ON ea.empname = e.empname
     GROUP BY 1, 2;

CREATE OR REPLACE FUNCTION update_emp_view() RETURNS TRIGGER AS $$
    BEGIN
        --
        -- Выполнить требуемую операцию в emp и добавить в emp_audit строку,
        -- отражающую эту операцию.
        --
        IF (TG_OP = 'DELETE') THEN
            DELETE FROM emp WHERE empname = OLD.empname;
            IF NOT FOUND THEN RETURN NULL; END IF;

            OLD.last_updated = now();
            INSERT INTO emp_audit VALUES('D', user, OLD.*);
            RETURN OLD;
        ELSIF (TG_OP = 'UPDATE') THEN
            UPDATE emp SET salary = NEW.salary WHERE empname = OLD.empname;
            IF NOT FOUND THEN RETURN NULL; END IF;

            NEW.last_updated = now();
            INSERT INTO emp_audit VALUES('U', user, NEW.*);
            RETURN NEW;
        ELSIF (TG_OP = 'INSERT') THEN
            INSERT INTO emp VALUES(NEW.empname, NEW.salary);

            NEW.last_updated = now();
            INSERT INTO emp_audit VALUES('I', user, NEW.*);
            RETURN NEW;
        END IF;
    END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER emp_audit
INSTEAD OF INSERT OR UPDATE OR DELETE ON emp_view
    FOR EACH ROW EXECUTE PROCEDURE update_emp_view();

  

Один из вариантов использования триггеров это поддержание в актуальном состоянии отдельной таблицы итогов для некоторой таблицы. В некоторых случаях отдельная таблица с итогами может использоваться в запросах вместо основной таблицы. При этом зачастую время выполнения запросов значительно сокращается. Эта техника широко используется в хранилищах данных, где таблицы фактов могут быть очень большими. [Пример 39.6](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger#plpgsql-trigger-summary-example "Пример 39.6. Триггерная процедура на PL/pgSQL для ведения таблицы итогов") показывает триггерную процедуру в PL/pgSQL, которая поддерживает таблицу итогов для таблицы фактов в хранилище данных.

**Пример 39.6. Триггерная процедура на PL/pgSQL для ведения таблицы итогов**

Представленная здесь схема данных частично основана на примере _Grocery Store_ из книги _The Data Warehouse Toolkit_ Ральфа Кимбалла (Ralph Kimball).

--
-- Основные таблицы: таблица временных периодов и таблица фактов продаж
--
CREATE TABLE time_dimension (
    time_key                    integer NOT NULL,
    day_of_week                 integer NOT NULL,
    day_of_month                integer NOT NULL,
    month                       integer NOT NULL,
    quarter                     integer NOT NULL,
    year                        integer NOT NULL
);
CREATE UNIQUE INDEX time_dimension_key ON time_dimension(time_key);

CREATE TABLE sales_fact (
    time_key                    integer NOT NULL,
    product_key                 integer NOT NULL,
    store_key                   integer NOT NULL,
    amount_sold                 numeric(12,2) NOT NULL,
    units_sold                  integer NOT NULL,
    amount_cost                 numeric(12,2) NOT NULL
);
CREATE INDEX sales_fact_time ON sales_fact(time_key);

--
-- Таблица с итогами продаж по периодам
--
CREATE TABLE sales_summary_bytime (
    time_key                    integer NOT NULL,
    amount_sold                 numeric(15,2) NOT NULL,
    units_sold                  numeric(12) NOT NULL,
    amount_cost                 numeric(15,2) NOT NULL
);
CREATE UNIQUE INDEX sales_summary_bytime_key ON sales_summary_bytime(time_key);

--
-- Функция и триггер для пересчёта столбцов итогов при выполнении
-- команд INSERT, UPDATE, DELETE
--
CREATE OR REPLACE FUNCTION maint_sales_summary_bytime() RETURNS TRIGGER
AS $maint_sales_summary_bytime$
    DECLARE
        delta_time_key          integer;
        delta_amount_sold       numeric(15,2);
        delta_units_sold        numeric(12);
        delta_amount_cost       numeric(15,2);
    BEGIN

        -- Вычислить изменение количества/суммы.
        IF (TG_OP = 'DELETE') THEN

            delta_time_key = OLD.time_key;
            delta_amount_sold = -1 * OLD.amount_sold;
            delta_units_sold = -1 * OLD.units_sold;
            delta_amount_cost = -1 * OLD.amount_cost;

        ELSIF (TG_OP = 'UPDATE') THEN

            -- Запретить изменение time_key -
            -- (это ограничение не должно вызвать неудобств, так как
            -- в основном изменения будут выполняться по схеме DELETE + INSERT).
            IF ( OLD.time_key != NEW.time_key) THEN
                RAISE EXCEPTION 'Запрещено изменение time_key : % -> %',
                                                      OLD.time_key, NEW.time_key;
            END IF;

            delta_time_key = OLD.time_key;
            delta_amount_sold = NEW.amount_sold - OLD.amount_sold;
            delta_units_sold = NEW.units_sold - OLD.units_sold;
            delta_amount_cost = NEW.amount_cost - OLD.amount_cost;

        ELSIF (TG_OP = 'INSERT') THEN

            delta_time_key = NEW.time_key;
            delta_amount_sold = NEW.amount_sold;
            delta_units_sold = NEW.units_sold;
            delta_amount_cost = NEW.amount_cost;

        END IF;


        -- Внести новые значения в существующую строку итогов или
        -- добавить новую.
        <<insert_update>>
        LOOP
            UPDATE sales_summary_bytime
                SET amount_sold = amount_sold + delta_amount_sold,
                    units_sold = units_sold + delta_units_sold,
                    amount_cost = amount_cost + delta_amount_cost
                WHERE time_key = delta_time_key;

            EXIT insert_update WHEN found;

            BEGIN
                INSERT INTO sales_summary_bytime (
                            time_key,
                            amount_sold,
                            units_sold,
                            amount_cost)
                    VALUES (
                            delta_time_key,
                            delta_amount_sold,
                            delta_units_sold,
                            delta_amount_cost
                           );

                EXIT insert_update;

            EXCEPTION
                WHEN UNIQUE_VIOLATION THEN
                    -- ничего не делать
            END;
        END LOOP insert_update;

        RETURN NULL;

    END;
$maint_sales_summary_bytime$ LANGUAGE plpgsql;

CREATE TRIGGER maint_sales_summary_bytime
AFTER INSERT OR UPDATE OR DELETE ON sales_fact
    FOR EACH ROW EXECUTE PROCEDURE maint_sales_summary_bytime();

INSERT INTO sales_fact VALUES(1,1,1,10,3,15);
INSERT INTO sales_fact VALUES(1,2,1,20,5,35);
INSERT INTO sales_fact VALUES(2,2,1,40,15,135);
INSERT INTO sales_fact VALUES(2,3,1,10,1,13);
SELECT * FROM sales_summary_bytime;
DELETE FROM sales_fact WHERE product_key = 1;
SELECT * FROM sales_summary_bytime;
UPDATE sales_fact SET units_sold = units_sold * 2;
SELECT * FROM sales_summary_bytime;

  

[](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger#plpgsql-event-trigger)

***
#### Триггеры событий

***
### Источник

- [postgrespro](https://postgrespro.ru/docs/postgrespro/9.6/plpgsql-trigger)

