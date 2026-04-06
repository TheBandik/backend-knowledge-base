# Records в Java

## Общая концепция

Record – это специальный тип класса в Java, предназначенный для описания **неизменяемых носителей данных (data carriers)**.

Ключевая идея:

> Record описывает **состояние как API**, а не реализацию.

Когда пишется:

```java
public record User(String name, int age) {}
```

Фактически объявляется:

* структура данных
* контракт равенства
* публичный API

И всё это – **декларативно**, без лишнего кода.

---

## Семантика Record

Record – это не просто «короткий класс». У него есть строгая модель:

* он **неявно final**
* он **наследуется от `java.lang.Record`**
* его поля – это **components (единое описание элемента данных, из которого компилятор генерирует сразу несколько вещей)**
* equality основано на значениях компонентов

Формулировка:

> Record – это номинальный тип, который описывает неизменяемое агрегированное значение и автоматически реализует value-based семантику.

---

## Как компилируется Record

Важно понимать, что происходит «под капотом»:

```java
public record User(String name, int age) {}
```

Компилятор создаёт:

* private final поля
* публичный канонический конструктор
* методы:

  * `name()`, `age()`
  * `equals()`
  * `hashCode()`
  * `toString()`

Дополнительно:

* класс становится `final`
* добавляется наследование от `Record`

---

## Components и API Record

Компоненты – это не просто поля:

```java
record User(String name, int age) {}
```

`name` и `age` – это:

* поля
* параметры конструктора
* публичные методы доступа

Это называется:

> **state description as header**

---

## Иммутабельность: что это реально значит

Record гарантирует:

* ссылки неизменяемы (`final`)
* нельзя изменить поля после создания

Но:

> иммутабельность НЕ глубокая

```java
record A(List<String> list) {}
```

Можно сделать:

```java
a.list().add("x");
```

Поэтому правильный паттерн:

```java
public record A(List<String> list) {
    public A {
        list = List.copyOf(list);
    }
}
```

---

## Конструкторы и инварианты

### Канонический конструктор

Создаётся автоматически:

```java
new User("Alice", 25);
```

---

### Компактный конструктор

Ключевая фича record:

```java
public record User(String name, int age) {
    public User {
        if (age < 0) throw new IllegalArgumentException();
    }
}
```

Особенности:

* параметры не объявляются явно
* можно валидировать данные
* можно трансформировать вход

Важно:

> присваивание полям происходит автоматически после блока

---

### Нормализация данных

```java
public record User(String name) {
    public User {
        name = name.trim();
    }
}
```

Это нормальная практика.

---

## Методы и поведение

Record – это полноценный класс:

```java
public record Rectangle(int w, int h) {
    public int area() {
        return w * h;
    }
}
```

Но есть граница:

> Record не должен содержать изменяющее поведение

Хорошо:

* вычисления
* валидация
* derived values

Плохо:

* бизнес-операции
* изменение состояния

---

## equals, hashCode и value semantics

Record реализует:

```java
u1.equals(u2)
```

по всем компонентам.

Это означает:

* порядок важен
* все поля участвуют

Следствие:

> Record идеально подходит как value object

---

## Использование в коллекциях

Record отлично работает с:

* `HashMap`
* `HashSet`

```java
record Key(String type, int id) {}

Map<Key, String> map = new HashMap<>();
```

Почему это безопасно:

* корректный hashCode
* immutable состояние

Но критично:

> поля внутри тоже должны быть immutable

---

## Наследование и ограничения

Record:

* не может наследоваться от классов
* не может быть родителем
* всегда `final`

Но:

* может реализовывать интерфейсы

```java
record User(String name) implements Serializable {}
```

---

## Статические элементы

Разрешено:

```java
record Util(int x) {
    static int sum(int a, int b) {
        return a + b;
    }
}
```

Также можно:

* static поля
* static фабричные методы

---

## Практические сценарии использования

### DTO и API

```java
public record UserResponse(String name, int age) {}
```

Используется в:

* REST
* микросервисах

---

### Value Object

```java
public record Email(String value) {
    public Email {
        if (!value.contains("@")) throw new IllegalArgumentException();
    }
}
```

---

### Ключи для кэша и Map

```java
record CacheKey(String url, String method) {}
```

---

### Возврат нескольких значений

```java
record Result(int min, int max) {}
```

---

### Конфигурации

```java
record DbConfig(String url, String user) {}
```

---

## Где Record использовать не стоит

### ORM (например, Hibernate)

Проблемы:

* нет сеттеров
* нет proxy
* нет lazy loading

---

### Изменяемые сущности

Если есть:

* lifecycle
* изменение состояния

→ нужен класс

---

### Сложная бизнес-логика

Record – не для этого.

---

## Сериализация и библиотеки

Record хорошо поддерживается:

* Jackson
* Spring Boot (современные версии)

Но:

* нужна версия библиотеки с поддержкой record
* используется канонический конструктор

---

## Reflection и особенности

Record имеет API:

```java
User.class.isRecord()
```

Можно получить компоненты:

```java
getRecordComponents()
```

---

## Record и паттерн matching

С Java 21+ record активно используется с pattern matching:

```java
if (obj instanceof User(String name, int age)) {
    ...
}
```

Это усиливает идею:

> record = data carrier

---

## Record vs Lombok

Частый вопрос.

Project Lombok vs Record:

| Критерий        | Record       | Lombok      |
| --------------- | ------------ | ----------- |
| Генерация кода  | встроена     | аннотации   |
| Иммутабельность | по умолчанию | опционально |
| Гибкость        | ниже         | выше        |

Ключевой ответ:

> Record – это язык, Lombok – библиотека.

---

## Ключевые подводные камни

### Mutable поля

Самая частая ошибка.

---

### Переопределение методов

Можно, но:

* легко сломать контракт
* редко нужно

---

### Использование «везде»

Record – не универсальная замена классу.
