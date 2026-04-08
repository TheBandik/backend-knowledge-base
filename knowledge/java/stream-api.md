# Java Stream API

## Что такое Stream API

Stream API – это способ обрабатывать данные (обычно коллекции) как поток элементов, применяя к ним цепочку операций.

Ключевая идея:
> Stream – это не структура данных, а конвейер обработки.

Очень важно это понять:
- Stream **не хранит элементы**
- Он **не изменяет исходную коллекцию**
- Он **описывает процесс обработки**

---

## Чем Stream отличается от коллекции

| Коллекция | Stream |
|----------|--------|
| хранит данные | не хранит |
| можно изменять | нет |
| можно использовать много раз | только один раз |
| eager (сразу) | lazy (лениво) |

### Почему Stream одноразовый?

Потому что он похож на поток данных (как InputStream):
- элементы "проходят" через pipeline
- после terminal операции поток закрывается

---

## Как устроен Stream pipeline

Любой Stream состоит из:

```
source → intermediate → terminal
````

Пример:

```java
list.stream()
    .filter(x -> x > 10)
    .map(x -> x * 2)
    .toList();
````

### Что здесь происходит на самом деле

1. берется элемент
2. проходит filter
3. проходит map
4. переходит к следующему

Это называется **pipeline processing**.

---

## Почему Stream ленивый (lazy evaluation)

Промежуточные операции ничего не делают сами по себе.

```java
list.stream()
    .filter(x -> x > 10); // НИЧЕГО не произошло
```

Почему так сделали:

* чтобы не делать лишнюю работу
* чтобы можно было оптимизировать pipeline
* чтобы поддерживать short-circuit операции

---

## Intermediate операции (промежуточные)

Они:

* возвращают новый Stream
* не запускают выполнение
* могут комбинироваться

---

### filter()

Фильтрует элементы по условию.

```java
stream.filter(x -> x > 10);
```

Что важно:

* не изменяет элементы
* просто "пропускает" или "отбрасывает"

---

### map()

Преобразует каждый элемент.

```java
stream.map(String::length);
```

Важно понимать:

* это **1 → 1**
* каждый элемент превращается в один новый

---

### flatMap()

```java
list.stream()
    .flatMap(Collection::stream)
```

Что делает:

* каждый элемент превращается в поток
* затем все потоки объединяются в один

Пример:

```java
List<List<String>> data = ...

// map → Stream<Stream<String>>
data.stream().map(List::stream)

// flatMap → Stream<String>
data.stream().flatMap(List::stream)
```

### Интуитивно:

* map → "обернул"
* flatMap → "расплющил"

---

### distinct()

Удаляет дубликаты.

Как работает:

* использует equals() и hashCode()

Важно:

* может быть дорогой операцией (использует Set внутри)

---

### sorted()

Сортирует элементы.

```java
stream.sorted();
```

Важно:

* требует сравнимых элементов или Comparator
* **вынуждает обработать весь stream** (не ленив в полном смысле)

---

### limit() и skip()

```java
stream.limit(10).skip(5);
```

* limit → берет первые N
* skip → пропускает первые N

Важно:

* limit – short-circuit операция

---

### peek()

```java
stream.peek(System.out::println);
```

Зачем:

* отладка pipeline

Почему опасен:

* может создать иллюзию логики
* в parallel может вести себя неожиданно

---

## Terminal операции

Они:

* запускают выполнение
* закрывают stream

---

### collect()

Самая важная операция.

```java
List<String> list = stream.toList();
```

Что происходит:

* элементы "прогоняются" через pipeline
* складываются в структуру

---

### reduce()

Используется для сворачивания в одно значение.

```java
int sum = list.stream()
    .reduce(0, Integer::sum);
```

Как работает:

* берет аккумулятор
* применяет его последовательно

### Когда reduce – плохая идея

```java
stream.reduce(new ArrayList<>(), ...)
```

Это ошибка.

Почему:

* reduce предназначен для immutable
* для коллекций нужен collect

---

### forEach()

```java
stream.forEach(System.out::println);
```

Почему это плохой стиль:

* это уже не декларативный код
* легко получить side effects

---

### findFirst() и findAny()

```java
stream.findFirst();
stream.findAny();
```

Разница:

* findFirst → гарантирует порядок
* findAny → быстрее в parallel

---

### anyMatch / allMatch / noneMatch

```java
stream.anyMatch(x -> x > 10);
```

Это:

* short-circuit операции
* могут завершиться раньше

---

## Short-circuiting

Это ключевая оптимизация.

Пример:

```java
stream.anyMatch(x -> x > 10);
```

Как работает:

* как только найден элемент → остановка

---

## Collector – что это на самом деле

Collector – это не просто "утилита".

Это:

> объект, который описывает, как собрать stream в результат

Он состоит из:

* supplier (создать контейнер)
* accumulator (добавить элемент)
* combiner (для parallel)
* finisher (опционально)

---

### Пример groupingBy

```java
list.stream()
    .collect(Collectors.groupingBy(String::length));
```

Что происходит:

* создается Map
* элементы распределяются по ключам

---

## Сбор в Map и проблема коллизий

```java
Collectors.toMap(keyMapper, valueMapper);
```

Если два одинаковых ключа → Exception.

Правильный вариант:

```java
Collectors.toMap(
    keyMapper,
    valueMapper,
    (oldVal, newVal) -> oldVal
);
```

---

## Optional в Stream

```java
Optional<String> result = stream.findFirst();
```

Почему Optional:

* может не быть результата
* заставляет явно обработать отсутствие

---

## Почему нельзя переиспользовать Stream

```java
Stream<Integer> s = list.stream();
s.count();
s.count(); // ошибка
```

Потому что:

* stream уже "прошел"
* он не хранит данные

---

## Side effects – главная проблема

```java
List<Integer> result = new ArrayList<>();

stream.forEach(result::add);
```

Почему это плохо:

* нарушает идею Stream
* ломает parallel execution
* делает код неочевидным

---

## Параллельные стримы

```java
list.parallelStream();
```

### Как они работают

* разбивают данные на части
* выполняют в ForkJoinPool
* объединяют результат

---

## ForkJoinPool

По умолчанию используется:

```java
ForkJoinPool.commonPool()
```

Проблема:

* это глобальный пул
* можно случайно "задушить" систему

---

## Когда использовать parallel streams

Хорошо:

* много данных
* тяжелые вычисления
* нет shared state

Плохо:

* IO
* синхронизация
* маленькие списки

---

## Изменение внешнего состояния

```java
int sum = 0;

stream.forEach(x -> sum += x);
```

Почему это не работает:

* переменные должны быть effectively final
* это защита от race condition

---

## Когда начинается выполнение

Только здесь:

```java
stream.toList();
```

До этого – просто описание.

---

## Частые ошибки

### Использование forEach вместо collect

Плохо:

```java
List<Integer> result = new ArrayList<>();
stream.forEach(result::add);
```

Правильно:

```java
List<Integer> result = stream.toList();
```

---

### Логика внутри map

Плохо:

```java
.map(x -> {
    if (x > 10) return x * 2;
    return x;
})
```

Правильно:

* разделить filter и map

---

### Злоупотребление Stream

Если код стал сложнее – ты делаешь что-то не так.

---

## Когда НЕ использовать Stream

* простой цикл → for лучше
* сложная логика → хуже читаемость
* критична производительность → проверять
