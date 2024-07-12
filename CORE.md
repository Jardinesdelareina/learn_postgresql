# Архитектура PostgreSQL

Файлы конфигурации и файлы данных, используемые кластером базы данных, традиционно хранятся вместе в каталоге данных кластера, который обычно называют PGDATA (по имени переменной среды, которую можно использовать для его определения). В ОС `Ubuntu` данный каталог находится по адресу `/var/lib/postgresql/14/main`. 

|Элемент|Описание|
|---|---|
|PG_VERSION|Текстовый файл, который содержит версию PostgreSQL, установленную на сервере. Это нужно для обеспечения совместимости между различными версиями PostgreSQL и обеспечения корректной работы сервера.|
|base|Директория, в которой хранятся данные таблиц, индексы и другие объекты баз данных. Каждая таблица хранится в отдельной поддиректории. Это одно из наиболее важных мест для работы PostgreSQL, так как здесь хранятся все данные пользователей. Для каждой базы данных в кластере существует подкаталог внутри PGDATA/base, названный по OID базы данных в pg_database. Этот подкаталог по умолчанию является местом хранения файлов базы данных; в частности, там хранятся её системные каталоги.|
|current_logfiles|Файл, который используется для хранения информации о текущих журналах транзакций сервера. Он содержит имена файлов журналов транзакций, которые используются для обеспечения целостности данных и восстановления после сбоев.|
|global|Файл, который содержит информацию о глобальных объектах БД, таких как шаблоны баз данных, функции и т.д. Он используется PostgreSQL для управления общими ресурсами между различными базами данных.|
|pg_commit_ts|Файл, который хранит информацию о временных метках выполнения транзакций. Он используется для обеспечения целостности транзакций и управления механизмом фиксации транзакций.|
|pg_dynshmem|Специальный файл, который используется для разделения динамической памяти между различными процессами PostgreSQL. Этот файл хранит информацию о разделенной динамической памяти, которая может быть использована для обмена данными между процессами сервера PostgreSQL.|
|pg_logical|Каталог, который содержит файлы, связанные с репликацией данных в формате логической репликации. В нем хранятся журналы изменений, которые позволяют передавать изменения данных из источника в приемник без пересылки всей базы данных.|
|pg_multixact|Каталог, который содержит файлы, связанные с механизмом множественных транзакций в PostgreSQL. Этот механизм предназначен для корректной работы в условиях параллельной обработки транзакций, где могут возникать конфликты блокировок между разными транзакциями.|
|pg_notify|Специальный канал связи между разными сессиями PostgreSQL, который позволяет отправлять уведомления о событиях в базе данных. С помощью pg_notify можно организовать асинхронное взаимодействие между разными процессами обработки данных.|
|pg_replslot|Каталог, в котором хранятся файлы, связанные с репликацией слотов в PostgreSQL. Репликационные слоты используются для управления потоком WAL (Write-Ahead Logging) и обеспечивают безопасную репликацию изменений данных на удаленные узлы.|
|pg_serial|Файл, который используется для хранения информации о последовательностях (sequences) в базе данных. Последовательности представляют собой объекты, генерирующие уникальные числовые значения, которые часто используются для создания уникальных первичных ключей в таблицах.|
|pg_snapshots|Файл, который содержит информацию о текущих транзакционных снимках базы данных. Снимки используются для изоляции изменений данных, вносимых одной транзакцией, от других транзакций.|
|pg_stat|Файл, который содержит статистическую информацию о работе СУБД.|
|pg_stat_tmp|Файл, который содержит временные данные статистики соответственно. Эти данные, как и данные из `pg_stat`, могут быть использованы для оптимизации запросов и мониторинга производительности базы данных.|
|pg_subtrans|Файл, который используется для отслеживания подтранзакций, которые могут быть созданы внутри основной транзакции в PostgreSQL.|
|pg_tblspc|Файл, который содержит информацию о таблицах, хранящихся внутри различных таблицных пространств (tablespace). Табличные пространства позволяют разделять хранение объектов базы данных на различных физических устройствах.|
|pg_twophase|Каталог, который содержит файлы, связанные с распределенными транзакциями. В нем хранятся данные по двухфазной фиксации транзакций, которая используется для обеспечения надежности при выполнении сложных операций.|
|pg_wal|Каталог, содержащий файлы WAL (журнал предзаписи)|
|pg_xact|Каталог, который содержит состояния активных транзакций в базе данных. Здесь хранятся данные о текущих транзакциях и их состоянии, что позволяет системе отслеживать изменения в данных и обеспечивать их целостность.|
|postgresql.auto.conf|Файл, используемый для хранения параметров конфигурации, которые устанавливаются при помощи ALTER SYSTEM|
|postmaster.opts|Файл, содержащий параметры командной строки, с которыми сервер был запущен в последний раз|
|postmaster.pid|Файл блокировки, содержащий идентификатор (ID) текущего управляющего процесса (PID), путь к каталогу данных кластера, время запуска управляющего процесса, номер порта, путь к каталогу Unix-сокета, первый корректный адрес прослушивания (listen_address), и ID сегмента разделяемой памяти (этот файл отсутствует после остановки сервера).|


### Процессы и память

При старте сервера запускается процесс, традиционно называемый `postmaster`. Он запускает все остальные процессы и "присматривает" за ними - если какой-нибудь процесс завершится аварийно, `postmaster` перезапустит его (или перезапустит весь сервер, если сочтет, что процесс мог повредить общие данные). 

Работу сервера обеспечивает ряд фоновых служебных процессов. Чтобы процессы могли обмениваться информацией, `postmaster` выделяет общую память, доступ к которой могут получить все процессы. Кроме общей памяти, каждый процесс имеет и свою локальную память, доступную только ему самому. 

`postmaster` слушает входящие соединения. При появлении клиента `postmaster` порождает обслуживающий процесс (backend) и дальше каждый клиент общается со своим процессом. Место, необходимое для выполнения запроса (разобранные запросы и их планы, состояние курсоров, кэш системного каталога, место для сортировки данных и т. п.), выделяется в локальной памяти обслуживающего процесса.

Когда к серверу подключается много клиентов, для каждого из них порождается собственный обслуживающий процесс. Это не проблема, пока клиентов не очень много, на всех хватает оперативной памяти,а соединения не происходят слишком часто.


### MVCC

<b>Мультиверсионность (multiversion concurrency control)</b> - это техника, позволяющая обеспечить параллельный доступ к данным для множества пользователей без блокировки или конфликтов. Она базируется на механизме снимков данных, который фиксирует состояние данных на определенный момент времени и позволяет транзакциям видеть данные, как если бы они были изменены после начала транзакции. Каждая транзакция в PostgreSQL видит свою "виртуальную" копию базы данных на момент старта, что позволяет избежать конфликтов записи и обеспечить консистентность данных.

<b>Снимок данных</b> - это состояние базы данных на определенный момент времени, которое является версией данных и остается постоянным в течение всей транзакции. Когда транзакция начинается, PostgreSQL создает снимок данных для этой транзакции, чтобы она видела данные, как они выглядели на момент старта транзакции. Это позволяет транзакциям работать с данными в изолированном режиме, не видя изменений других транзакций до их фиксации.

В снимке видны самые последние версии уже зафиксированных данных, а еще незафиксированные данные не видны. Иными словами, от каждой строки в снимок попадает версия, соответствующая моменту создания снимка.

Снимок — это не физическая копия данных, а всего несколько чисел:
* номер последней зафиксированной транзакции на момент создания снимка (он определяет тот самый момент времени);
* список активных транзакций на этот момент.

Преимущества мультиверсионности и снимков данных в PostgreSQL:
1. Повышение производительности: блокировки минимизированы, что позволяет параллельно выполнять множество операций чтения и записи.
2. Изоляция транзакций: каждая транзакция видит свою версию данных, что предотвращает чтение грязных данных.
3. Поддержка скрытия конфликтов: транзакции проходят ряд уровней изоляции, определяющих видимость данных другим транзакциям.

Многоверсионность — тот основной механизм, который обеспечивает первые три свойства транзакций (атомарность, согласованность, изоляция).


### Изоляция

Уровни изоляции в PostgreSQL обеспечивают контроль над тем, как одна транзакция взаимодействует с данными, когда другие транзакции также работают с теми же данными. Каждый уровень предоставляет разный уровень изоляции и тем самым контролирует то, как изменения видны другим транзакциям. К выбору уровня изоляции следует подходить внимательно, в зависимости от требований к целостности данных и производительности приложения. Реализация уровней изоляции в PostgreSQL строже, чем в стандарте SQL.

Основные уровни изоляции и аномалии, которые предотвращаются на разных уровнях:

1. <b>Read Uncommitted (Чтение незафиксированных данных)</b>:

   Транзакция на уровне Read Uncommitted может видеть изменения, сделанные другими транзакциями, до их фиксации.

   <em>Может привести к чтению неподтвержденных данных и неконсистентным результатам.</em>

   <em>Не поддерживается PostgreSQL, работает как <b>Read Committed</b>.</em>


2. <b>Read Committed (Чтение зафиксированных данных)</b>:

   Транзакция видит только данные, фиксированные другими транзакциями - не видит изменения до фиксации.

   Аномалии: `Dirty Read` - так называемое Грязное чтение, когда читаемые данные могут быть изменены и сделать результаты недействительными. Иными словами, когда одна транзакция может читать измененные, но не зафиксированные строки другой транзакции. И если произойдет ROLLBACK другой транзакции, то первая транзакция прочитает данные, которых никогда не существовало.

   <em>Работает в PostgreSQL по-умолчанию.</em>


3. <b>Repeatable Read (Повторяемое чтение)</b>:

   Гарантирует, что при повторном выполнении один и тот же запрос получит те же данные. Повторное чтение измененной строки вернет первоначальное значение, если оно было изменено и зафиксировано другой транзакцией. От уровня Read Committed этот уровень изоляции отличается и тем, что на нем транзакция может быть оборвана, чтобы не допустить аномалию (такую транзакцию надо повторять).

   Аномалии: `Non-Repeatable Read` (неповторяющееся чтение) - после того, как первая транзакция прочитала строку, а вторая транзакция ее изменила или удалила и зафиксировала изменения, при повторном чтении первой строки это будет замечено. `Phantom Read`(фантомное чтение) - если прочитанный первой транзакцией набор строк претерпел изменения от второй транзакции, то повтор первой транзакции вернет обновленный набор строк.


4. <b>Serializable (Сериализуемость)</b>:

   Предотвращает любые аномалии. Обеспечивает максимальный уровень изоляции, предотвращая конфликты между параллельными транзакциями и гарантируя их выполнение как будто бы последовательно. Команды, выполняемые в конкурентно работающих транзакциях, приводят к такому же результату, какой получился бы в случае последовательного — одна транзакция завершилась, следующая началась. Это самый высокий уровень изоляции с наивысшей степенью защиты от аномалий, но может привести к блокировкам и ухудшению производительности, транзакции могут обрываться чаще, чем это действительно необходимо.

   Аномалии: Аномалии конкурентного доступа - это ситуация, когда две или более транзакции пытаются получить доступ к общим данным одновременно, что может привести к конфликту или ошибке.

   <em>Не работает на репликах.</em>


Проверить текущий уровень изоляции:
```sql
SHOW transaction_isolation;
```

Установить уровень изоляции:
```sql
BEGIN;

SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

COMMIT;
```

или

```sql
BEGIN ISOLATION LEVEL READ COMMITTED;

COMMIT;
```


### Блокировки

Задача блокировок заключается в упорядочивании конкурентного доступа к разделяемым ресурсам. Под конкурентным доступом подразумевается одновременный доступ нескольких процессов.

Основные блокировки устанавливаются на уровне строк. При этом чтение никогда не блокирует ни читающие, ни пишущие транзакции. Изменение строки не блокирует ее чтение. Единственный случай, когда транзакция будет ждать освобождения блокировки — если она пытается менять строку, которая уже изменена другой, еще не зафиксированной, транзакцией.

Блокировки также устанавливаются на более высоком уровне,в частности на таблицах. Они нужны для того, чтобы никто не смог удалить таблицу, пока другие транзакции читают из нее данные, или чтобы запретить доступ к перестраиваемой таблице. 

PostgreSQL имеет много различных типов блокировок, которые могут быть использованы для предотвращения конфликтов в многопользовательском окружении и обеспечения безопасной работы с данными.

* Эксклюзивная блокировка (Exclusive Lock): Эта блокировка блокирует ресурс так, что он становится недоступным для других пользователей. Только один пользователь может иметь эксклюзивную блокировку на ресурс в любой момент.

* Чтение блокировки (Share Lock): Эта блокировка позволяет пользователям делиться ресурсом для чтения, но блокирует его от изменений другими пользователями.

* Обновление блокировки (Update Lock): Эта блокировка используется перед выполнением операции обновления данных. Она блокирует запись от изменений другими пользователями, чтобы гарантировать целостность данных.

* Ограничение блокировки (Row Share Lock): Эта блокировка ограничивает доступ к конкретной строке данных. Другие пользователи могут читать данные из этой строки, но они не могут её изменить.

* Эксклюзивная блокировка для доступа к таблице (Access Exclusive Lock): Эта блокировка блокирует всю таблицу, предотвращая доступ другим пользователям к данным в ней.

* Интентивные блокировки (Intent Locks): Эти блокировки используются для обозначения намерения выполнения блокировки на определенном уровне (например, на таблице или на строке). Они помогают предотвратить конфликты между операциями блокировки на разных уровнях.

PostgreSQL также поддерживает блокировки уровней изоляции транзакций, такие как Read Committed, Repeatable Read, Serializable, которые определяют уровень изоляции данных и могут влиять на взаимодействие с блокировками.

Есть также возможность использования непрямых блокировок через функции и процедуры сервера. Это позволяет программистам реализовать свою логику блокировок в зависимости от специфики их приложения.

Просмотр блокировок, доступных в представлении pg_locks:
```sql
SELECT locktype, virtualxid AS virtxid, transactionid AS xid, mode, granted
FROM pg_locks
```

* locktype — тип ресурса,
* mode — режим блокировки,
* granted — удалось ли получить блокировку.


Функция `pg_current_xact_id()` возвращает текущий идентификатор транзакции в формате UUID, который не изменяется в пределах одной транзакции. Этот идентификатор может быть использован для отслеживания изменений, выполненных в рамках данной транзакции.

Функция `txid_current()` возвращает текущий идентификатор транзакции в виде целого числа (txid). Этот идентификатор также не изменяется в пределах одной транзакции и может быть использован для отслеживания изменений.

Таким образом, основное отличие между этими функциями заключается в формате идентификатора транзакции, который они возвращают.

Получить номер обслуживающего процесса:
```sql
SELECT pg_backend_pid();
```

`SELECT FOR UPDATE` - это конструкция в SQL, которая используется для блокировки выбранных строк в режиме обновления. Она применяется в контексте многопользовательской среды базы данных, чтобы гарантировать, что выбранные строки не будут изменены другими транзакциями до того, как текущая транзакция завершит свою работу.

Когда выполняется запрос `SELECT ... FOR UPDATE`, строки, выбранные этим запросом, блокируются в режиме обновления. Это означает, что другие транзакции не смогут изменять данные в этих строках до тех пор, пока блокировка не будет снята. Обычно такая блокировка сохраняется до завершения текущей транзакции.

```sql
BEGIN;

-- Выбираем строки для обновления и блокируем их
SELECT * FROM users WHERE id = 1 FOR UPDATE;

-- Теперь другие транзакции не смогут изменить данные строки с id=1,

...
-- Выполнение других операций или обновлений

-- Когда все операции закончены, фиксируем транзакцию
COMMIT;
```

Важно использовать `SELECT FOR UPDATE` аккуратно, чтобы избежать длительных блокировок и конфликтов между транзакциями. Эта конструкция полезна в случаях, когда нужно обеспечить целостность данных при выполнении операций чтения и записи.

`SKIP LOCKED` - это опция запроса, которая позволяет игнорировать заблокированные строки, то есть строки, которые уже заблокированы другими транзакциями. При использовании этой опции, запрос будет пропускать заблокированные строки и обрабатывать только свободные строки. Это может быть полезно, если нужно избежать ожидания блокировки и продолжить выполнение запроса без прерывания.


### Буферный кэш

Буферный кэш (buffer cache) - это механизм, который используется для временного хранения данных из дискового хранилища в оперативной памяти. Буферный кэш используется для сглаживания скорости работы оперативной памяти и дисков. Он состоит из массива буферов (его размер задается конфигурационным параметром `shared_buffers`), которые содержат страницы данных и дополнительную информацию (например, имя файла и положение страницы внутри этого файла). Размер страницы обычно составляет 8 Кбайт.

Когда PostgreSQL загружает данные таблицы или индекса с диска, он помещает их в буферный кэш. При обращении к данным, PostgreSQL в первую очередь проверяет их наличие в кэше. Если данные уже присутствуют в кэше, то запрос может быть выполнен непосредственно из оперативной памяти, ускоряя его выполнение. В случае отсутствия данных в кэше, PostgreSQL считывает их с диска.

Использование буферного кэша помогает уменьшить количество обращений к диску, что повышает производительность системы и снижает задержки в выполнении запросов. Однако необходимо учитывать ограниченный объем оперативной памяти, который может быть выделен под буферный кэш, и настраивать его размер оптимально для конкретной системы.

Если процесс изменил данные в странице, соответствующий буфер называется "грязным". Измененная страница подлежит записи на диск, но по соображениям производительности запись происходит асинхронно и может откладываться.

Буферный кэш PostgreSQL располагается в общей памяти, чтобы все процессы имели к нему доступ. PostgreSQL работает с дисками, на которых находятся данные, не напрямую, а через операционную систему. У операционной системы тоже имеется собственный кэш данных. Поэтому, если страница не будет найдена в буферном кэше, остается шанс, что она есть в кэше ОС и обращения к диску удастся избежать.

Буферный кэш, как и другие структуры общей памяти, защищен блокировками для управления одновременным доступом. Хотя блокировки и реализованы эффективно, доступ к буферному кэшу далеко не так быстр, как простое обращение к оперативной памяти. Поэтому в общем случае чем меньше данных читает и изменяет запрос, тем быстрее он будет работать.

Расширение, которое позволяет посмотреть, что происходит в буферном кэше - `pg_buffercache`.

Узнать размер буферного кэша:
```sql
SHOW shared_buffers;
```

Узнать количество грязных буферов в буферном кэше:
```sql
SELECT count(*)
FROM pg_buffercache
WHERE isdirty;
```


#### Влияние буферного кэша на выполнение запросов

Создадим базу данных
```sql
CREATE DATABASE test_data_db;
```

Подключимся к ней
`\c test_data_db`

Создадим таблицу
```sql
CREATE TABLE t (n INTEGER);
```

Наполним ее данными (100 тысяч записей)
```sql
INSERT INTO t SELECT id FROM generate_series(1, 100000) AS id;
```

Выполним очистку
```sql
VACUUM ANALYZE t;
```

Выйдем из psql
`\q`

Выполним рестарт сервера PostgreSQL чтобы сбросить содержимое буферного кэша
`sudo service postgresql restart`

Подключимся к базе данных занова
`psql test_data_db`

Выполним запрос
```sql
EXPLAIN (analyze, buffers, costs off, timing off)
SELECT * FROM t;
```

И если ввести этот запрос еще раз, то показатели Planning Time и Execution Time будут гораздо меньше, так как во время выполнения второго запроса данные брались уже из буферного кэша.


### Журнал предзаписи WAL (write-ahead log)

Журнал записей транзакций - это механизм, который используется для обеспечения надежности и восстановления данных в случае сбоев. Когда происходит изменение данных в БД, PostgreSQL сначала записывает это изменение в журнал WAL, прежде чем фактически изменить страницу данных. Иными словами, сначала происходит запись о транзакции в журнал WAL, затем происходит сама транзакция. Это обеспечивает атомарность и целостность транзакций. Основная причина существования журнала — необходимость восстановления согласованности данных в случае сбоя, при котором теряется содержимое оперативной памяти, в частности, буферный кэш.

В журнал WAL не попадают записи только о временных и нежурналируемых (unlogged) таблицах. Операции над данными в нежурналируемых таблицах производятся гораздо быстрее, но не попадают в WAL. Такие таблицы создаются в первую очередь для таких данных, потеря которых не критична.

WAL содержит последовательность записей, представляющих изменения данных, произведенные транзакциями. Этот журнал можно использовать для восстановления данных после сбоя. В случае сбоя PostgreSQL может восстановить данные, применив изменения из WAL к последнему снимку базы данных.

WAL также используется для обеспечения согласованности данных в репликации, позволяя повторить все изменения данных на других серверах в той же последовательности, что и на основном сервере.

Текущая позиция в журнале предзаписи
```sql
SELECT pg_current_wal_insert_lsn();
```

Логически журнал можно представить в виде непрерывного потока записей. Каждая запись имеет номер, называемый LSN (Log Sequence Number). Это 64-разрядное число — смещение записи в байтах относительно начала журнала. Физически журнал хранится в файлах в отдельном каталоге (PGDATA/pg_wal). По умолчанию файлы имеют размер 16 Мбайт, задать другой размер можно при инициализации кластера баз данных.

На файлы можно взглянуть не только в файловой системе, но и с помощью функции:
```sql
SELECT * FROM pg_ls_waldir() ORDER BY name;
```

Когда сервер PostgreSQL запускается после сбоя, он входит в режим восстановления. Информация на диске в это время несогласованна: изменения "горячих" страниц пропали, поскольку эти страницы все еще находились в кэше, хотя другие, более поздние изменения уже были сброшены на диск. Чтобы восстановить согласованность, PostgreSQL читает журнал WAL и последовательно проигрывает каждую журнальную запись, если соответствующее изменение не попало на диск. Таким образом восстанавливается работа всех транзакций. Затем те транзакции, запись о фиксации которых не успела попасть в журнал, обрываются.

Восстановление после сбоя начинается с ближайшей контрольной точки, что позволяет хранить только файлы журнала, записанныес момента последней пройденной контрольной точки.


#### Режимы журналирования

* <b>Синхронный</b> - при фиксации транзакции продолжение работы невозможно до тех пор, пока все журнальные записи, относящиеся к этой транзакции, не окажутся на диске. При синхронной записи гарантируется долговечность - если транзакция зафиксирована, то все ее журнальные записи уже на диске и не будут потеряны. Обратная сторона состоит в том, что синхронная запись увеличивает время отклика (команда COMMIT не возвращает управление до окончания синхронизации) и поэтому уменьшает производительность системы.

* <b>Асинхронный</b> - журнал записывается частями в фоновом режиме. Асинхронная запись эффективнее синхронной. Во-первых, фиксация изменений не должна ничего ждать. Во-вторых, при каждой записи на диск обрабатываются все накопившиеся журнальные записи и, таким образом, уменьшается число избыточных обращений к диску. Однако надежность уменьшается: зафиксированные данные могут пропасть в случае сбоя, если между фиксацией и сбоем прошло не очень много времени.

Настройка режимов журналирования (можно устанавливать не только глобально, но и на уровне отдельной транзакции): `synchronous_commit`

Объем данных, который попадает в журнал, регулируется параметром `wal_level`.
* На уровне <b>minimal</b> журнал обеспечивает только восстановление после сбоя.
* На уровне <b>replica</b> в журнал добавляется информация, позволяющая использовать его для создания резервных копий и репликации. При репликации журнальные записи передаются на другой сервер и применяются там; таким образом создается и поддерживается точная копия (реплика) основного сервера.
* На уровне <b>logical</b> в журнал добавляется информация, позволяющая декодировать "физические" журнальные записи и сформировать из них "логические" записи о добавлении, изменении и удалении табличных строк. Это позволяет организовать логическую репликацию.