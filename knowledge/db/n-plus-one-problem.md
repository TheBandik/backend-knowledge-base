# 1. Что такое проблема N + 1

**Определение:**

Проблема **N + 1** – это ситуация, когда:

* выполняется **1 запрос** для получения списка сущностей
* и затем выполняется **N дополнительных запросов** для получения связанных данных для каждой сущности

---

## Простейший пример

Есть:

* `users`
* `orders` (у пользователя может быть много заказов)

Ты пишешь:

```sql
SELECT * FROM users;
```

А затем для каждого пользователя:

```sql
SELECT * FROM orders WHERE user_id = ?;
```

Если пользователей 100:

* 1 запрос (users)
* * 100 запросов (orders)

**Итого: 101 запрос**

---

## Почему это проблема

Ошибка новичков:
"Ну и что, запросы же простые?"

Нет. Проблема не в сложности запроса, а в:

### 1. Latency (задержки сети)

Каждый запрос:

* round-trip до БД
* ожидание ответа

100 маленьких запросов почти всегда хуже 1 большого.

---

### 2. Нагрузка на БД

БД вынуждена:

* парсить каждый запрос
* планировать выполнение
* открывать курсоры

---

### 3. Не масштабируется

На 10 пользователей – ок
На 10 000 – всё падает

---

# 2. В чем корень проблемы

Главная причина – **неправильная модель доступа к данным**

Ты думаешь:

> «Я просто беру пользователей, а потом их заказы»

Но база думает:

> «Ты делаешь сотни отдельных операций вместо одной»

---

# 3. Как это выглядит в реальной жизни

## Плохой вариант

```java
List<User> users = userRepository.findAll();

for (User user : users) {
    List<Order> orders = orderRepository.findByUserId(user.getId());
}
```

---

## SQL под капотом

```sql
SELECT * FROM users;

SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
SELECT * FROM orders WHERE user_id = 3;
...
```

---

# 4. Правильное решение (на уровне идеи)

Главная мысль:

**Нужно получать связанные данные одним запросом**

---

## JOIN

```sql
SELECT u.*, o.*
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
```

Теперь:

* 1 запрос
* все данные сразу

---

## Альтернатива – IN

```sql
SELECT * FROM orders 
WHERE user_id IN (1, 2, 3, ..., N);
```

---

# 5. N + 1 в чистом SQL

Важно:
Это не проблема Hibernate. Это **архитектурная проблема**.

---

## Как возникает в SQL

```sql
SELECT id FROM users;
```

Дальше в коде:

```sql
SELECT * FROM orders WHERE user_id = ?
```

---

## Когда это происходит

* курсоры
* циклы в backend-коде
* ORM
* микросервисы (ещё хуже)

---

# 6. Особенности в Hibernate

## Lazy Loading – главный виновник

```java
class User {
    @OneToMany(mappedBy = "user", fetch = FetchType.LAZY)
    List<Order> orders;
}
```

---

### Что делает Hibernate

```java
List<User> users = entityManager.createQuery("FROM User").getResultList();

for (User user : users) {
    user.getOrders();
}
```

---

### SQL

```sql
SELECT * FROM users;

SELECT * FROM orders WHERE user_id = 1;
SELECT * FROM orders WHERE user_id = 2;
...
```

---

## Почему это коварно

Код выглядит нормально:

```java
user.getOrders();
```

Но внутри – запрос.

---

# 7. Как решать в Hibernate

## 7.1 JOIN FETCH (основной способ)

```java
SELECT u FROM User u
JOIN FETCH u.orders
```

---

### Что делает

* загружает users + orders сразу
* убирает N + 1

---

## 7.2 EntityGraph

```java
@EntityGraph(attributePaths = {"orders"})
List<User> findAll();
```

---

## 7.3 Batch fetching

```properties
hibernate.default_batch_fetch_size=16
```

---

### Что происходит

Вместо:

```sql
WHERE user_id = 1
WHERE user_id = 2
```

Hibernate делает:

```sql
WHERE user_id IN (1,2,3,...16)
```

---

## 7.4 FetchType.EAGER – плохое решение

Ошибка новичков:

```java
fetch = FetchType.EAGER
```

---

Почему плохо:

* грузит лишнее
* может вызвать **другие N + 1**
* ухудшает контроль

---

# 8. Универсальные способы решения (без привязки к технологии)

## 1. JOIN

Самый базовый и правильный способ

---

## 2. Предзагрузка (prefetching)

Сначала:

```sql
SELECT * FROM users;
```

Потом:

```sql
SELECT * FROM orders WHERE user_id IN (...)
```

---

## 3. Кэширование

* second-level cache
* Redis

---

## 4. Денормализация

Иногда проще:

* хранить агрегаты
* уменьшить необходимость JOIN

---

## 5. Pagination + ограничение выборки

Ошибка:

```java
findAll()
```

Правильно:

```java
findTop100()
```

---

# 9. Пример на Java

## Плохой код

```java
List<User> users = userRepository.findAll();

for (User user : users) {
    System.out.println(user.getOrders().size());
}
```

---

## Хороший код

```java
@Query("SELECT u FROM User u JOIN FETCH u.orders")
List<User> findAllWithOrders();
```

---

## Или через DTO

```java
SELECT new com.example.UserOrderDTO(u.name, o.id)
FROM User u
JOIN u.orders o
```

---

# 10. Частые ошибки на собеседовании

## Ошибка 1

> "Это проблема Hibernate"

Нет.
Это проблема архитектуры доступа к данным.

---

## Ошибка 2

> "Нужно всегда использовать EAGER"

Нет. Это костыль.

---

## Ошибка 3

> "JOIN всегда лучше"

Не всегда:

* может раздуть результат (cartesian explosion)
* дублирование данных

---

## Ошибка 4

> "Это только про SQL"

Нет:

* REST API тоже страдает (N + 1 HTTP calls)

---

# 11. Как объяснить на собеседовании

Короткая формулировка:

> N + 1 – это ситуация, когда сначала выполняется один запрос на получение сущностей, а затем по одному запросу на каждую связанную сущность. Это приводит к большим накладным расходам из-за количества запросов. Решается через JOIN, batch loading, или предзагрузку данных.
