# Базы данных

### Реляционная модель данных

<em>Реляционная модель данных</em> - это математическая модель, которая описывает способ организации и хранения данных в базе данных. Она основана на понятии "таблицы" или "реляции", которая состоит из строк и столбцов.

В реляционной модели каждая таблица имеет набор атрибутов (столбцов), которые определяют тип данных, содержащихся в каждой ячейке столбца. Каждая строка таблицы представляет собой конкретную запись или кортеж данных.

Главная идея реляционной модели заключается в том, что связи между таблицами (реляциями) осуществляются через общие атрибуты, так называемые "внешние ключи". Внешний ключ в таблице ссылается на первичный ключ другой таблицы, что позволяет связывать данные между разными таблицами.

Преимущества реляционной модели данных:
1. Простота структуры и понятность модели.
2. Гибкость и возможность создания сложных запросов для извлечения данных.
3. Независимость от физической реализации данных.
4. Высокая надежность и целостность данных.
5. Поддержка множества операций, таких как сортировка, поиск, фильтрация и соединение данных.


### Транзакции и транзакционность

<b>Транзакция</b> в базе данных представляет собой последовательность операций базы данных, которые выполняются как единое целое. Транзакция обладает следующими свойствами, известными как ACID-свойства:

1. <em>Атомарность (Atomicity)</em>: Транзакция либо выполняется полностью, либо не выполняется вообще. Если одна из операций внутри транзакции не может быть выполнена, то вся транзакция откатывается, и все изменения, сделанные до этого, отменяются.

2. <em>Согласованность (Consistency)</em>: Транзакция должна приводить базу данных из одного согласованного состояния в другое согласованное состояние. Это означает, что после завершения транзакции должны быть выполнены все правила целостности БД.

3. <em>Изолированность (Isolation)</em>: Каждая транзакция должна выполняться изолированно от других транзакций. Изменения, внесенные одной транзакцией, должны быть видимы только после успешного завершения этой транзакции.

4. <em>Долговечность (Durability)</em>: После успешного завершения, изменения, сделанные в транзакции, должны быть сохранены и доступны даже в случае сбоя системы или отключения питания.

<b>Транзакционность</b> в базах данных означает, что операции, выполняемые в рамках транзакции, являются неделимыми и отражают только либо полное выполнение, либо отмену всех изменений. Это обеспечивает надежность и целостность данных.

```sql
-- Начало транзакции
BEGIN;

-- Выполнение операций внутри транзакции
UPDATE users
SET balance = balance - 100
WHERE user_id = 1;

UPDATE products
SET quantity = quantity - 1
WHERE product_id = 100;

-- Проверка результатов операций
SELECT * FROM users WHERE user_id = 1;
SELECT * FROM products WHERE product_id = 100;

-- Если все операции выполнены успешно, фиксируем транзакцию
COMMIT;

-- Если произошла ошибка или нужно отменить изменения, откатываем транзакцию
ROLLBACK;
```


### Типы данных

Типы данных [описаны в документации](https://postgrespro.ru/docs/postgresql/14/datatype). 

Некоторые типы данных PostgreSQl:

1. Числовые:
   - `INTEGER` (целочисленный тип)
   - `BIGINT` (большие целые числа)
   - `DECIMAL` или `NUMERIC` (число с фиксированной точностью)
   - `REAL` или `FLOAT4` (число с плавающей запятой с одинарной точностью)
   - `DOUBLE PRECISION` или `FLOAT8` (число с плавающей запятой с двойной точностью)

2. Символьные:
   - `CHAR(n)` или `CHARACTER(n)` (строка фиксированной длины)
   - `VARCHAR(n)` или `CHARACTER VARYING(n)` (строка переменной длины)
   - `TEXT` (строка переменной длины без ограничений)

3. Дата и время:
   - `DATE` (дата)
   - `TIME` (время без часового пояса)
   - `TIMESTAMP` (дата и время без часового пояса)
   - `TIMESTAMPTZ` (дата и время с часовым поясом)

4. Логический:
   - `BOOLEAN` (логическое значение: `TRUE` или `FALSE`)

5. Бинарные:
   - `BYTEA` (переменная длина для бинарных данных)

6. Массивы:
   - `INTEGER[]` (массив целых чисел)
   - `VARCHAR(255)[]` (массив строк переменной длины)


### Очистка дискового пространства

1. <b>VACUUM</b>: Эта команда выполняет процесс автоматического освобождения пространства в таблицах, которое было выделено для удаленных, обновленных или вставленных строк. Она также обновляет статистику таблицы, которая используется оптимизатором запросов для выбора наиболее эффективных планов выполнения запросов.

2. <b>VACUUM FULL</b>: Эта команда выполняет более интенсивный процесс очистки и компактации таблицы. Она перемещает данные из таблицы в новое физическое расположение, освобождая пространство, которое занимали удаленные строки. Однако, <b>VACUUM FULL</b> блокирует таблицу на время выполнения операции и может быть более ресурсоемкой по сравнению с обычным <b>VACUUM</b>.

В целом, <b>VACUUM</b> обычно достаточно для поддержания эффективности работы базы данных, однако, при необходимости выполнить более глубокую очистку и компактацию таблицы, может использоваться <b>VACUUM FULL</b>.