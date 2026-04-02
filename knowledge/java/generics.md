# Generics в Java

## Введение

**Generics (обобщения)** – это механизм параметризации типов, позволяющий создавать классы, методы и интерфейсы, работающие с различными типами данных при сохранении **типобезопасности (type safety)**.

Ключевая идея:

> Перенос проверки типов с runtime на compile-time.

Это:

* уменьшает количество ошибок
* делает код предсказуемым
* упрощает работу с коллекциями

---

## Проблема до Generics

Рассмотрим код без generics:

```java
List list = new ArrayList();
list.add("Hello");
list.add(4L);

String s = (String) list.get(0);
```

### Пояснение

* `List` хранит элементы как `Object`
* компилятор **не знает**, какие именно типы внутри
* приходится явно приводить тип: `(String)`

Проблема проявляется здесь:

```java
String s = (String) list.get(1);
```

Если в списке лежит `Long`, произойдёт:

```text
ClassCastException
```

То есть ошибка возникает **во время выполнения**, что плохо.

---

## Решение через Generics

```java
List<String> list = new ArrayList<>();
list.add("Hello");
list.add(4L); // ошибка компиляции
```

### Пояснение

* `List<String>` означает: список может хранить **только String**
* компилятор проверяет это при добавлении
* попытка добавить `Long` приводит к ошибке **до запуска программы**

---

## Generic-классы

```java
class Box<T> {
    private T value;

    public void set(T value) {
        this.value = value;
    }

    public T get() {
        return value;
    }
}
```

### Пояснение

* `T` – параметр типа (placeholder)
* при использовании он заменяется конкретным типом

Пример использования:

```java
Box<String> box = new Box<>();
box.set("Hello");

String value = box.get();
```

Здесь:

* компилятор подставляет `T → String`
* метод `get()` гарантированно возвращает `String`

---

## Несколько параметров типов

```java
class Pair<K, V> {
    private K key;
    private V value;
}
```

### Пояснение

* используется, когда нужно хранить **две разные сущности**
* пример: `Map<K, V>`

---

## Type Inference и Diamond Operator

```java
List<String> list = new ArrayList<>();
```

### Пояснение

* компилятор выводит тип справа из левой части
* это называется **type inference**

Без этого пришлось бы писать:

```java
new ArrayList<String>()
```

---

## Инвариантность Generics

```java
List<String> strings = new ArrayList<>();
List<Object> objects = strings; // ошибка
```

### Пояснение

Хотя `String` наследуется от `Object`, это не работает для generics.

Почему:

```java
List<Object> objects = new ArrayList<String>();
objects.add(123); // сломает список строк
```

Чтобы избежать таких ситуаций, generics сделаны **инвариантными**.

---

## Ограничения типов (Bounds)

### Верхняя граница

```java
public <T extends Number> void print(T value) {
    System.out.println(value.doubleValue());
}
```

### Пояснение

* `T` ограничен типом `Number`
* можно вызывать методы `Number`
* нельзя передать, например, `String`

---

### Несколько ограничений

```java
<T extends Number & Comparable<T>>
```

### Пояснение

Тип должен:

* наследоваться от `Number`
* реализовывать `Comparable`

---

## Generics в методах

```java
public <T> T getFirst(List<T> list) {
    return list.get(0);
}
```

### Пояснение

* метод работает с любым типом
* тип определяется при вызове

Пример:

```java
String s = getFirst(List.of("a", "b"));
```

Компилятор понимает, что `T = String`.

---

## Wildcards

### Неограниченный wildcard

```java
void print(List<?> list) {
    for (Object o : list) {
        System.out.println(o);
    }
}
```

### Пояснение

* `?` означает "неизвестный тип"
* можно читать как `Object`
* нельзя добавлять элементы (кроме `null`)

---

### Верхняя граница

```java
void printNumbers(List<? extends Number> list)
```

### Пояснение

* список содержит `Number` или его наследников
* можно читать как `Number`
* нельзя добавлять (тип может быть, например, `List<Integer>`)

---

### Нижняя граница

```java
void addNumbers(List<? super Integer> list) {
    list.add(10);
}
```

### Пояснение

* список принимает `Integer` или его родителей (`Number`, `Object`)
* добавлять можно
* при чтении получаем `Object`

---

### PECS

> Producer Extends, Consumer Super

* читаешь → `extends`
* записываешь → `super`

---

## Type Erasure

```java
List<String> list = new ArrayList<>();
```

### Пояснение

После компиляции:

```java
List list = new ArrayList();
```

Generics удаляются, остаются только касты.

---

### Последствия

```java
if (list instanceof List<String>) // ошибка
```

Причина:

* в runtime нет информации о `<String>`

---

### Замена типа

```java
<T extends Comparable>
```

После стирания:

```java
Comparable
```

---

## Ограничения Generics

### Примитивы

```java
List<int> // нельзя
List<Integer> // можно
```

**Причина:** generics работают только с объектами

---

### Создание T

```java
new T() // нельзя
```

**Причина:** тип неизвестен в runtime

---

### Массивы

```java
new T[10] // нельзя
```

**Причина:** массивы требуют точный тип в runtime

---

### Статика

```java
class Box<T> {
    static T value; // ошибка
}
```

**Причина:** static не привязан к конкретному параметру типа

---

## Raw Types

```java
List list = new ArrayList();
```

### Пояснение

* отключает generics
* возвращает поведение как до Java 5
* приводит к runtime ошибкам

Использование – плохая практика.

---

## Практический пример

```java
public static <T> void copy(List<? super T> dest, List<? extends T> src) {
    for (T item : src) {
        dest.add(item);
    }
}
```

### Пояснение

* `src` – источник (producer) → `extends`
* `dest` – приёмник (consumer) → `super`
* обеспечивает безопасное копирование

---

## Вопросы

### Table of Contents

* [Что такое generics и зачем они нужны?](#что-такое-generics-и-зачем-они-нужны)
* [Generics работают на compile-time или runtime?](#generics-работают-на-compile-time-или-runtime)
* [Что такое type erasure?](#что-такое-type-erasure)
* [Какие ограничения накладывает type erasure?](#какие-ограничения-накладывает-type-erasure)
* [Как обеспечивается типобезопасность?](#как-обеспечивается-типобезопасность)
* [Почему raw types опасны?](#почему-raw-types-опасны)
* [Почему generics инвариантны?](#почему-generics-инвариантны)
* [Что такое wildcard (`?`)?](#что-такое-wildcard-)
* [Разница между `? extends T` и `? super T`](#разница-между--extends-t-и--super-t)
* [Почему нельзя добавлять в `? extends`?](#почему-нельзя-добавлять-в--extends)
* [Почему `List<Integer>` не является `List<Number>`?](#почему-listinteger-не-является-listnumber)
* [Когда использовать `? extends`?](#когда-использовать--extends)
* [Почему `List<Object>` и `List<?>` – не одно и то же?](#почему-listobject-и-list--не-одно-и-то-же)
* [Почему работает `List<? extends Number>`?](#почему-работает-listextends-number)
* [Почему не работает `List<Number> = new ArrayList<Integer>()`?](#почему-не-работает-listnumber--new-arraylistinteger)
* [Почему работает `? super`?](#почему-работает--super)
* [Почему не работает `? super Integer` с Double?](#почему-не-работает--super-integer-с-double)
* [Почему `List<?>` принимает любой тип?](#почему-list--принимает-любой-тип)
* [Где использовать wildcard?](#где-использовать-wildcard)
* [Ключевые выводы](#ключевые-выводы)

---

### Что такое generics и зачем они нужны?

Generics (обобщения) – механизм параметризации типов, позволяющий писать универсальный и типобезопасный код.

Основные цели:

* типобезопасность на этапе компиляции
* устранение явных приведений типов
* повышение читаемости и надёжности

```java
List<String> list = new ArrayList<>();
String s = list.get(0);
```

---

### Generics работают на compile-time или runtime?

Generics работают на этапе компиляции.
В runtime информация о типах удаляется (type erasure).

---

### Что такое type erasure?

Type erasure – удаление информации о generic-типах при компиляции.

```java
List<String> → List
List<Integer> → List
```

`T` заменяется:

* на `Object`
* или на ограничение (`T extends Number → Number`)

---

### Какие ограничения накладывает type erasure?

* нельзя `new T()`
* нельзя `instanceof List<String>`
* нельзя `new T[]`
* нельзя перегружать методы только по generic-параметру

---

### Как обеспечивается типобезопасность?

Компилятор:

* проверяет типы
* вставляет неявные cast

```java
String s = (String) list.get(0);
```

---

### Почему raw types опасны?

```java
List list = new ArrayList<String>();
list.add(123);

List<String> strings = list;
String s = strings.get(0); // ClassCastException
```

Ошибка возникает при извлечении.

---

### Почему generics инвариантны?

```java
List<Integer> ≠ List<Number>
```

Иначе:

```java
List<Integer> ints = new ArrayList<>();
List<Number> nums = ints;

nums.add(3.14); // ломает тип
```

---

### Что такое wildcard (`?`)?

```java
List<?> list
```

Список неизвестного типа.

---

### Разница между `? extends T` и `? super T`

### `? extends T`

* чтение
* нельзя добавлять

```java
List<? extends Number>
```

---

#### `? super T`

* запись
* чтение только как `Object`

```java
List<? super Integer>
```

---

#### PECS

Producer Extends, Consumer Super

---

### Почему нельзя добавлять в `? extends`?

Тип неизвестен (`Integer` или `Double`), добавление небезопасно.

---

### Почему `List<Integer>` не является `List<Number>`?

Из-за инвариантности generics.

---

### Когда использовать `? extends`?

Когда метод должен принимать разные подтипы:

```java
void print(List<? extends Number> list)
```

---

### Почему `List<Object>` и `List<?>` – не одно и то же?

`List<Object>`:

* можно добавлять любые объекты

`List<?>`:

* нельзя добавлять (кроме null)

---

### Почему работает `List<? extends Number>`?

```java
List<? extends Number> list = new ArrayList<Integer>();
```

`Integer` удовлетворяет ограничению.

---

### Почему не работает `List<Number> = new ArrayList<Integer>()`?

Инвариантность generics.

---

### Почему работает `? super`?

```java
List<? super Integer> list = new ArrayList<Number>();
```

`Number` – родитель `Integer`.

---

### Почему не работает `? super Integer` с Double?

`Double` не является родителем `Integer`.

---

### Почему `List<?>` принимает любой тип?

```java
List<?> list = new ArrayList<String>();
```

`?` – неизвестный тип.

---

### Где использовать wildcard?

Использовать:

* в сигнатурах методов
* в API

Не использовать:

* без необходимости в локальных переменных
