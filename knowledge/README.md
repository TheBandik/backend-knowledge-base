# Знания

## Java Core

* [ООП](java/oop.md)
* [Полиморфизм, типы ссылок и абстракции](java/polymorphism-reference-types-and-abstraction.md)
* [equals и hashCode](java/equals-and-hashcode.md)
* [String и неизменяемость](java/string-and-immutability.md)
* [Records](java/records.md)
* [Generics](java/generics.md)
* [Stream API](java/stream-api.md)
* [Коллекции (List, Set, Map, HashMap)]
* [Исключения (checked vs unchecked)]

### JVM

* [Память JVM (heap, stack, metaspace)]
* [Java Memory Model и happens-before]
* [Garbage Collector]
* [ClassLoader]

### Дополнительно

* [Reflection]

---

## Многопоточность

* [Thread, Runnable, Callable]
* [synchronized и volatile]
* [Happens-before и visibility]
* [Race condition и deadlock]

### Concurrency API

* [ExecutorService и пулы потоков]
* [Future и CompletableFuture]
* [Locks (ReentrantLock)]
* [Concurrent коллекции]

---

## Spring

### Core

* [IoC и DI](spring/ioc-and-di.md)
* [Жизненный цикл бина]
* [Конфигурация (annotations vs Java config)]

### Spring Boot

* [Автоконфигурация]
* [Стартеры и структура проекта]

### Web

* [REST API (контроллеры, request/response)]
* [Валидация (Bean Validation)]
* [Обработка ошибок (@ControllerAdvice)]

### Data

* [JPA и Hibernate (основы)]
* [Состояния entity]
* [Транзакции (@Transactional)]
* [Lazy vs Eager loading]

---

## Базы данных

### SQL

* [JOIN (виды и применение)]
* [GROUP BY, HAVING]

### Производительность

* [Индексы](db/indexes.md)
* [План выполнения (EXPLAIN)]

### Транзакции

* [ACID]
* [Уровни изоляции]
* [Аномалии (dirty, non-repeatable, phantom)]

### ORM

* [Hibernate]
* [Кэш (1-й и 2-й уровень)]
* [Проблема N+1](db/n-plus-one-problem.md)

---

## HTTP и сеть

* [HTTP: методы, коды, headers]
* [Жизненный цикл HTTP-запроса]
* [REST: ограничения и принципы]
* [HTTPS (TLS)]

---

## Безопасность

* [Аутентификация vs авторизация]
* [JWT]
* [OAuth 2.0]

---

## Архитектура

* [Слоистая архитектура]
* [DTO vs Entity]
* [Монолит vs микросервисы](architecture/monolith-vs-micro.md)
* [Идемпотентность]
* [Стейтлесс]

---

## Микросервисы

* [Взаимодействие сервисов (REST, gRPC)]
* [Сериализация (JSON, Avro)]
* [Брокеры сообщений (Kafka, RabbitMQ)]

### Проблемы

* [Распределённые транзакции]
* [Согласованность данных]
* [Fault tolerance]

---

## Кэширование

* [Зачем нужен кэш]
* [Redis]
* [Стратегии кэширования]
* [Инвалидация кэша]

---

## Окружение

### Linux

* [Файловая система и команды]
* [Процессы и сигналы]
* [Сеть (порты, curl, netstat)]
* [Права доступа]

### Инструменты

* [Docker]
* [Логирование (уровни, подходы)]
