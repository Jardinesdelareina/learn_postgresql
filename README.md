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


### Установка и использование PostgreSQL в Linux


<div>
    <img src="https://github.com/devicons/devicon/blob/master/icons/linux/linux-original.svg" width="40" height="40"/>&nbsp;
    <img src="https://github.com/devicons/devicon/blob/master/icons/postgresql/postgresql-original.svg" width="40" height="40"/>&nbsp;
</div>


`sudo apt install postgresql`   установка

`sudo service postgresql status`    проверка, запущен ли сервис

`sudo service postgresql start`     запуск сервера если он не запущен

`sudo service postgresql restart`   перезапуск сервера postgresql

`sudo pg_isready`       проверка. готов ли сервер postgresql принимать подключение от клиентов

`sudo -u postgres psql`     подключение к серверу, активация оболочки <b>psql</b>

`CREATE USER your_username WITH PASSWORD 'your_password';`      создание пользователя и пароля

`GRANT ALL PRIVILEGES ON DATABASE your_database TO your_username;`      предоставление привелегий новому пользователю

`\q`    выход из <b>psql</b>


### Установка pgAdmin4

1. Установка из репозитория <b>pgAdmin4 APT</b>:

`curl https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo apt-key add`

`sudo sh -c 'echo "deb https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" > /etc/apt/sources.list.d/pgadmin4.list && apt update'`

2. Запуск установки <b>pgAdmin4</b>:

`sudo apt install pgadmin4`

3. Запуск скрипта, устанавливающего веб-компоненты. Скрипт находится по адресу `/usr/pgadmin4/bin/setup-web.sh`.

4. В процессе установки будет перезапущена служба Apache2. После завершения работы скрипта необходимо добавить разрешение для Apache2 на доступ через брэндмауэр:

`sudo ufw allow 'Apache'`

5. Запуск брэндмауэра:

`sudo ufw enable`

6. Убедиться, что Apache2 включена в список разрешенных в брэндмауэре:

`sudo ufw status`

7. Для доступа к веб-версии pgAdmin4 ввести в браузере:

`http://<ip-адрес:порт>/pgadmin4`


### Основные команды PSQL

`\l`    список баз данных

`\c <db_name>`  подключение к базе данных

`\dt`   список таблиц базы данных

`\du`   список пользователей


### Типы данных

Типы данных [описаны в документации](https://postgrespro.ru/docs/postgresql/14/datatype). PostgreSQL поддерживает символьные, целочисленные, вещественные типы данных, типы времяисчисления, логический тип, массивы, JSON, XML, геометрические и кастомные типы, а также NULL - отсутствие данных.