### Типы данных PostgreSQL

Типы данных [описаны в документации](https://postgrespro.ru/docs/postgresql/14/datatype). 

Некоторые типы данных PostgreSQL:

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


#### XML

```sql
SELECT $xml$
<computer>                                <!-- открывающий тег -->
  <motherboard>
    <!-- текстовый элемент -->
    <cpu>Intel® Core™ i7-7567U</cpu>
    <ram>
      <!-- тег с атрибутом -->
      <dimm size_gb="32">Crucial DDR4-2400 SODIMM</dimm>
    </ram>
  </motherboard>
  <disks>
    <ssd size_gb="512">Intel 760p Series</ssd>
    <hdd size_gb="3000">Toshiba Canvio</hdd>
  </disks>
</computer>                               <!-- закрывающий тег -->
$xml$ AS xml
```

Вывести все, что заключено в теге `<ram>`:
```sql
SELECT xpath('/computer/motherboard/ram', :'xml');
```
или
```sql
SELECT xpath('/computer//ram', :'xml');
```

Передвижение по дереву не только вниз, но и вверх:
```sql
SELECT xpath('//ram/dimm/..', :'xml');
```

Извлечение текста:
```sql
SELECT xpath('//cpu/text()', :'xml');
```

Извлечение значения атрибутов (с помощью @):
```sql
SELECT xpath('//@size_gb', :'xml');
```

Условия фильтрации в квадратных скобках:
```sql
SELECT xpath('//*[@size_gb >= 1024]', :'xml');
```

Преобразование XML в реляционный вид и вставка данных в подготовленну таблицу:
```sql
SELECT xpath('//disks/*', :'xml');

INSERT INTO disks(drive_type, name, capacity)
SELECT * FROM xmltable(
    '//disks/*'
    PASSING :'xml'
    COLUMNS
        drive_type  text PATH 'name()',
        name        text PATH 'text()',
        capacity integer PATH '@size_gb * 1024'
);
```


#### JSON

```sql
SELECT $js$
{ "motherboard": {
    "cpu": "Intel® Core™ i7-7567U",
    "ram": [
      { "type": "dimm",
        "size_gb": 32,
        "model": "Crucial DDR4-2400 SODIMM"
      }
    ]
  },
  "disks": [
    { "type": "ssd",
      "size_gb": 512,
      "model": "Intel 760p Series"
    },
    { "type": "hdd",
      "size_gb": 3000,
      "model": "Toshiba Canvio"
    }
  ]
}
$js$ AS json
```

Вывести документ в удобочитаемый вид:
```sql
SELECT jsonb_pretty(:'json'::jsonb);
```

Перемещение по дереву:
```sql
SELECT jsonb_pretty(jsonb_path_query(:'json', '$.motherboard.ram'));
```

Выбор элементов массива в квадратных скобках:
```sql
SELECT jsonb_pretty(jsonb_path_query(:'json', '$.disks[0]'));
```

Все элементы сразу:
```sql
SELECT jsonb_pretty(jsonb_path_query(:'json', '$.disks[*]'));
```

Условия фильтрации
```sql
SELECT jsonb_pretty(jsonb_path_query(:'json', '$.disks ? (@.size_gb > 1000)'));
```

Синтаксис стрелочной нотации (одинарная стрелка - перемещение, двойная - текстовое представление):
```sql
SELECT (:'json'::jsonb)->'motherboard'->'ram'->0->>'model';
```

Преобразование JSON в реляционный вид:
```sql
WITH dsk(d) AS (
   SELECT jsonb_path_query(:'json', '$.disks[*]')
)
SELECT d FROM dsk;
```

Преобразование таблицы в JSON:
```sql
SELECT json_agg(disks) FROM disks;
```

#### GIN для слабоструктурированных данных

Идея метода доступа GIN (general inverted index) основана на том, что для сложносоставных значений имеет смысл индексировать элементы значений, а не все значение целиком.

Для хранения элементов в GIN используется обычное B-дерево, поэтому элементы должны принадлежать к сортируемому типу данных. Основные отличия от B-дерева состоят в следующем:
* Когда нужно проиндексировать новое значение, это значение разбивается на элементы и индексируются сразу все элементы. Поэтому в индекс добавляется не один элемент, а сразу несколько (обычно много).
* Каждый элемент индекса ссылается на множество табличных строк.
* Хотя элементы и организованы в B-дерево, классы операторов GIN не поддерживают операции сравнения «больше», «меньше».

GIN-индекс для jsonb на основе B-tree по выражению:
```sql
CREATE INDEX disks_btree_idx ON disks((disk->>'capacity'));
```