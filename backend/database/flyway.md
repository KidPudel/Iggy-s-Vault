
there is a flyway, which is convenient and battle tested automation migration tool
- **Validation**: Flyway can verify that the applied migrations match the scripts in your project, catching accidental changes or corruption.
- **Baseline Support**: It allows you to define a baseline version for existing databases without reapplying old migrations.
- **Undo Migrations**: Some Flyway editions support `undo` scripts for rollback scenarios.
- **[[Repeatable Migrations]]**: Useful for non-versioned scripts (e.g., refreshing views).

### Use cases of flyway

- baseline - initiate migration mechanism on existing DB (init)
- clean - full clean up of all objects that are listed in configuration `schemas`
- info - full information of all migrations
- migrate - apply migrations based on migration files and info about applied migrations
- repair - clean up the migration history table (`flyway_shema_history`) - Removes failed migration entries: failed migrations are flagged and repair will remove them to retry. NOTE that repair does not apply migrations
  - Fixes metadata: updates/recalculates checksum if the migration script was modified after it was applied
 - Cleanup history: only successfull and valid migration will remain
- validate - verifies that **applied** migrations matching expected state defined in the migration scripts. **should be used before applying new migrations**
  - checksum of the migrations - whether they were modified, by comaring checksums in the filesystems with ones stored in the migrtaion history table.
  - order of the migrations
  - integrity of the migrations - detects if migrations are missing or additional migrations that shouldn't be there
  - version consistency - checks if database version aligns with the latest migrtaion applied

### example

flyway.conf.local

```
flyway.locations=filesystem:./db/migrations # Директория для поиска файлов миграций (абсолютный или относительный от места запуска путь)
flyway.password='' # Пароль пользователя, пустой пароль также необходимо указывать
flyway.schemas=migrations # Список схем для работы. Первая указанная является дефолтной, в ней будет создана таблица с информацией о миграц
flyway.table = project # Таблица примененных миграций на целевой базе данных
flyway.url=jdbc:postgresql://host:port/db_name # строка подключения к целевой базе данных
flyway.user=postgres # пользователь БД, от которого будет вестись работа с миграциями
```

when launch in flyway it is required to specify location of the config file

```shell
flyway -configFiles=./flyway.conf migrate
```

additionally, we can specify all of those:

```shell
flyway -url=jdbc:postgresql://host:port/db_name -user=postgres -password='' -schemas=migrations -locations=filesystem:./db/migrations migrate
```

# schema
- **db** - каталог базы данных
    - **migrations** - Каталог версионных миграций базы данных
        - **data** - Каталог повторяемых миграций с данными
            - **R__**{краткое_описание_данных}.sql - Повторяемая (R) миграция для пользовательских данных
            - ...
        - **functions** - каталог повторяемых миграций функций
            - **{схема 1}** - Каталог повторяемых миграций с функциями для схемы 1
                - **R__**{имя_функции_включая_имя_схемы}.sql - Повторяемая (R) миграция для функции в схеме 1
                - ...
            - **{схема 2}** - Каталог повторяемых миграций с функциями для схемы 2
                - **R__**{имя_функции_включая_имя_схемы}.sql - Повторяемая (R) миграция для функции в схеме 2
                - ...
            - ...
        - **triggers** - Каталог повторяемых миграций триггерных функций
            - **{схема 1}** - Каталог повторяемых миграций с триггерными функциями для схемы 1
                - **R__**{имя_триггерной_функции_включая_имя_схемы}.sql - Повторяемая (R) миграция для триггерной функции в схеме 1
                - ...
            - **{схема 2}** - Каталог повторяемых миграций с триггерными функциями для схемы 2
                - **R__**{имя_триггерной_функции_включая_имя_схемы}.sql - Повторяемая (R) миграция для триггерной функции в схеме 2
                - ...
            - ...
        - **views** - Каталог повторяемых миграций представлений
            - **{схема 1}** - Каталог повторяемых миграций с представлениями для схемы 1
                - **R__**{имя_представления_включая_имя_схемы}.sql - Повторяемая (R) миграция для представления функции в схеме 1
                - ...
            - **{схема 2}** - Каталог повторяемых миграций с представлениями для схемы 2
                - **R__**{имя_представления_включая_имя_схемы}.sql - Повторяемая (R) миграция для представления функции в схеме 1
                - ...
            - ...
        - **V2__**версионная_миграция.sql
        - **V3__**версионная_миграция.sql
        - **V4_**_версионная_миграция.sql
        - ...
- **01-baseline-globals**.sql - Базовая линия базы данных с globals
- **02-baseline-schema**.sql - Базовая линия структуры базы данных
    
- **03-baseline-sys**.sql - Базовая линия системных данных
    
- **04-baseline-data**.sql - Базовая линия первоначальных данных
    
- **flyway.conf** - Файл конфигурации FlyWay

> Note: it applies migrations in order by default, and to add order we can do something like R__01 and R__02

## Имя версионной (V) миграции должно соответствовать формату

`db/migrations/V{порядковый номер/версия миграции}__{краткое_описание_изменения}.sql`
Версионные (V) миграции используются для ведения физической структуры базы данных:

- Схемы, их создание, изменение, удаление
- Анализаторы FTS
- Домены
- Конфигурации FTS
- Материализованные представления
- Последовательности
- Правила сортировки
- Словари FTS
- Сторонние таблицы
- Таблицы, включая индексы и ограничения к ним
- Типы
- Шаблоны FTS
- Исправление пользовательских данных

## Имя повторяемой (R) миграции должно соответствовать формату

`db/migrations/{тип(data|functions|triggers|view)}/{схема}/R__{изменяемая сущность}.sql`
- data - каталог повторяемых (R) миграций для пользовательских данных
- functions - каталог повторяемых (R) миграций для функций
- triggers - каталог повторяемых (R) миграций для триггерных функций
- views - каталог повторяемых (R) миграций для представлений