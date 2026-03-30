# IoC (Inversion of Control) и DI (Dependency Injection) в Spring

## Проблема, с которой всё начинается

Наивная реализация:

```java
class UserService {
    private EmailService emailService = new SmtpEmailService();
}
```

### Почему это плохо

* жёсткая зависимость от реализации
* невозможно легко заменить `SmtpEmailService`
* сложно писать тесты (нет mock)
* нарушение принципа DIP

---

## Inversion of Control (IoC)

### Определение

**IoC (Inversion of Control)** — это принцип, при котором управление созданием и связями объектов передаётся внешнему контейнеру.

---

### Суть

Было:

```text
Объект сам создаёт зависимости
```

Стало:

```text
Контейнер создаёт и связывает объекты
```

---

### В Spring

IoC реализуется через контейнер:

* ApplicationContext

---

### Что делает контейнер

1. создаёт объекты (beans)
2. хранит их
3. связывает зависимости
4. управляет жизненным циклом

---

## Dependency Injection (DI)

### Определение

**DI (Dependency Injection)** — это способ реализации IoC, при котором зависимости передаются объекту извне.

---

### Ключевая идея

Объект **не создаёт зависимость**, а **получает её**

---

### Пример

```java
class UserService {
    private final EmailService emailService;

    public UserService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

---

## Связь IoC и DI

| Понятие | Роль                |
| ------- | ------------------- |
| IoC     | принцип             |
| DI      | механизм реализации |

---

## Виды DI

---

### Constructor Injection (основной и рекомендуемый)

```java
class A {
    private final B b;

    public A(B b) {
        this.b = b;
    }
}
```

### Плюсы:

* объект всегда в корректном состоянии
* зависимости обязательны
* удобно тестировать
* можно сделать поля `final`

---

### Setter Injection

```java
class A {
    private B b;

    public void setB(B b) {
        this.b = b;
    }
}
```

#### Минусы:

* зависимость может не быть установлена
* объект может быть невалидным

---

### Field Injection

```java
@Autowired
private B b;
```

#### Минусы:

* скрытые зависимости
* сложно тестировать
* нарушает инкапсуляцию

---

## Как Spring внедряет зависимости

Spring:

1. находит класс (через аннотации)
2. определяет зависимости (конструктор / поле / сеттер)
3. ищет подходящий bean
4. внедряет его

---

### По какому принципу ищется зависимость

#### По типу

```java
class A {
    A(B b) {}
}
```

→ ищется bean типа `B`

---

#### Если несколько кандидатов

```java
interface PaymentService {}

@Service
class CardPayment implements PaymentService {}

@Service
class PayPalPayment implements PaymentService {}
```

---

### Решение

#### `@Qualifier`

```java
public A(@Qualifier("cardPayment") PaymentService service) {}
```

---

#### `@Primary`

```java
@Primary
class CardPayment implements PaymentService {}
```

---

## Что такое Bean

### Определение

**Bean** — это объект, созданный и управляемый Spring контейнером.

---

### Важно

```java
new UserService()
```

❌ не bean

```java
@Service
class UserService {}
```

✔ bean

---

## Роль интерфейсов

Правильный подход:

```java
interface EmailService {}
class SmtpEmailService implements EmailService {}
```

```java
class UserService {
    UserService(EmailService emailService) {}
}
```

---

### Почему это важно

* можно менять реализацию
* соответствует DIP
* удобно тестировать

---

## Как это выглядит в Spring

```java
interface EmailService {
    void send();
}
```

```java
@Service
class SmtpEmailService implements EmailService {
    public void send() {}
}
```

```java
@Service
class UserService {
    private final EmailService emailService;

    public UserService(EmailService emailService) {
        this.emailService = emailService;
    }
}
```

---

### Что делает Spring

1. создаёт `SmtpEmailService`
2. создаёт `UserService`
3. передаёт зависимость в конструктор

---

## Ошибки, которые важно понимать

---

### Создание через `new`

```java
UserService service = new UserService();
```

→ Spring не участвует → DI не работает

---

### Field Injection

```java
@Autowired
private EmailService emailService;
```

→ плохая практика

---

### Привязка к реализации

```java
private SmtpEmailService service;
```

→ нарушение DIP

---

### Несколько реализаций без уточнения

→ ошибка при запуске

---

## Почему DI важен

DI даёт:

* слабую связанность
* заменяемость компонентов
* тестируемость (mock/stub)
* расширяемость системы

---

## Связь с SOLID

DI реализует:

* **D (Dependency Inversion Principle)**
* частично помогает **O (Open/Closed)**

---

## Итоговая модель

```text
Класс не создаёт зависимости
        ↓
Контейнер создаёт объекты (IoC)
        ↓
Контейнер внедряет зависимости (DI)
```
