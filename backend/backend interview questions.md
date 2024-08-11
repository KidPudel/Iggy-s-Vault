# Top 10 questions in backed python interview

[[python interview questions]]

## Indexes, what is it, and types
[[indexes]]

## transaction
[[transactions]]


## replication
- [[database replication]]
- [[database shard]]


## messaging systems
[[RabbitMQ]]
[[Kafka]]


## containerization
[[Docker basics]]
[[Docker commands]]


# tests
[[unit tests]]

# Вопросы по архитектуре и принципам разработки ПО:

- В чем отличие объектно-ориентированного и функционального программирования?
## Что такое SOLID, DRY, KISS, YAGNI?
- [[SOLID]]
- [[DRY]]
- [[KISS]]
- [[YAGNI]]
- Назовите плюсы и минусы монолитной архитектуры?
- Назовите плюсы и минусы микросервисов?
- Как «распилить» монолит, не прекращая разработку? 
## Authentication vs Authorization
- Identification: that the user exists
- Authentication: process of verifying the user credentials
- Authorization: checking and giving some role with permissions
[[Authentication and authorization]]

# Вопросы по реляционным БД:

## Что такое составной индекс?
[[types of indexes]]

## Что такое внешний ключ?
[[foreign key]]
## Для чего нужен оператор UNION?
[[union]]
- По вашему мнению, всегда ли необходима нормализация БД? Когда целесообразно использовать денормализованные БД?
- Как вы перенесете приложение из одной базы данных в другую, например, из MySQL в PostgreSQL? 

# Вопросы по NoSQL:

- Что вы подразумеваете под NoSQL?
- Какие бывают типы NoSQL СУБД?
- Какие преимущества и недостатки реляционных СУБД в сравнении с NoSQL?
- Что такое MapReduce?
- Есть ли индексы в NoSQL?

# Вопросы по CI/CD:

- Как подключить CI/CD к GitLab/Github?
- Как настроить тестирование на каждый комит, а сборку артефактов на создание тега?
- Как ускорить выполнение pipeline с помощью директивы cache и/или использования docker?
- Как подключить сервисы при тестровании (например ДБ/Redis и т.д.)?
- Как настроить деплой релиза?

# Вопросы по Clouds:

- Как можно развернуть приложение в клаудах? Как делать это регулярно? (Как связать с CICD?)
- Как скалировать приложение в клаудах?
- Как масштабировать и бекапить данные БД в клаудах?
- Как организовать сервис сообщений в клаудах? Аналог RabbitMQ, ZeroMQ, Kafka, etc.?
- Что такое lambda функции? Для каких задач их лучше всего использовать? Какие есть ограничения?

# Вопросы по GIT:

- Что такое index в контексте GIT?
- Отличие tag от branch?
- Какой командой можно отменить последние 2 коммита. *С сохранением изменений в коммите/без сохранения?
- Как удалить файл из индекса Git? (сам файл должен остаться на своем месте)
- Что выполняет команда git rebase -i HEAD~15?
- За что отвечает команда git stash?
- Какие вы знаете модели ветвления в Git?