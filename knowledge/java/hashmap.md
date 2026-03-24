# HashMap

## 1. Общая идея

`HashMap` – это структура данных, реализующая интерфейс `Map<K, V>`, которая хранит пары **ключ → значение**.

Главные свойства:

* ключи **уникальны**
* значения **могут повторяться**
* доступ к данным осуществляется **по ключу**, а не по индексу

Пример:

```java
Map<String, Integer> map = new HashMap<>();

map.put("apple", 1);
map.put("banana", 2);
map.put("apple", 3); // перезапишет значение

System.out.println(map.get("apple")); // 3
```

---

## 2. Как работает HashMap внутри

HashMap – это не просто «список пар», а структура на основе **хеширования**.

Внутри:

* массив бакетов (`Node<K, V>[] table`)
* каждый бакет содержит:

  * один элемент
  * или список (LinkedList)
  * или дерево (Red-Black Tree, начиная с Java 8)

---

### 2.1 Добавление элемента (`put`)

Алгоритм:

1. Вызывается `hashCode()` у ключа
2. Хеш преобразуется в индекс массива:

   ```java
   index = (n - 1) & hash
   ```
3. Проверка бакета:

   * пуст → вставка
   * не пуст:

     * если ключ совпадает → перезапись
     * иначе → добавление в цепочку (или дерево)

Пример коллизии:

```java
map.put("Aa", 1);
map.put("BB", 2); // могут дать одинаковый hashCode
```

---

### 2.2 Получение (`get`)

1. Считаем hash
2. Находим бакет
3. Перебираем элементы:

   * список → O(n)
   * дерево → O(log n)

```java
Integer value = map.get("apple");
```

---

## 3. Коллизии

Коллизия – ситуация, когда разные ключи попадают в один бакет.

```java
key1.hashCode() == key2.hashCode()
```

Как решается:

* до Java 8 → LinkedList
* с Java 8 → при длине > 8 → Red-Black Tree

Важно:

* плохой `hashCode()` → деградация до O(n)

---

## 4. equals() и hashCode()

Контракт:

1. Если `equals()` возвращает `true` → `hashCode()` обязан совпадать
2. Если `hashCode()` совпадает → `equals()` может быть разным

Пример правильной реализации:

```java
class User {
    String name;

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof User)) return false;
        User user = (User) o;
        return Objects.equals(name, user.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name);
    }
}
```

Если этого нет – HashMap будет работать некорректно.

---

## 5. Важная проблема: mutable key

Если ключ изменяемый – карта ломается.

```java
class User {
    String name;
}

User user = new User();
user.name = "A";

map.put(user, 1);

user.name = "B";

map.get(user); // может не найти значение
```

Причина: изменился `hashCode`.

---

## 6. Размер, load factor и resize

По умолчанию:

* capacity = 16
* loadFactor = 0.75

Когда происходит resize:

```java
size > capacity * loadFactor
```

Что происходит:

* создается новый массив (в 2 раза больше)
* все элементы перераспределяются

Это дорогая операция – O(n)

---

## 7. Почему размер – степень двойки

Используется:

```java
index = (n - 1) & hash
```

Это быстрее, чем `%`, но работает только если `n` – степень двойки.

---

## 8. Основные методы

### put()

```java
map.put("key", 1); // добавляет или перезаписывает значение
```

---

### get()

```java
map.get("key"); // возвращает значение или null
```

---

### remove()

```java
map.remove("key"); // удаляет элемент по ключу
```

---

### containsKey / containsValue

```java
map.containsKey("key"); // есть ли такой ключ в мапе
map.containsValue(1); // есть ли такое значение в мапе
```

⚠️ `containsValue` – O(n)

---

### size / isEmpty / clear

```java
map.size(); // возвращает размер мапы
map.isEmpty(); // проверяет пустая ли мапа
map.clear(); // удаляет все элементы в мапе
```

---

## 9. Работа с коллекциями

### keySet()

```java
Set<String> keys = map.keySet(); // возвращает множество ключей (уникальные)
```

---

### values()

```java
Collection<Integer> values = map.values(); // значения могут повторяться
```

---

### entrySet()

Самый важный способ обхода:

```java
for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
```

Почему это важно:

* не происходит двойного поиска (как при keySet + get)

---

## 10. putAll()

```java
map1.putAll(map2); // копирует все элементы из одной map в другую
```

---

## 11. Сложность операций

| Операция | Средняя | Худшая |
| -------- | ------- | ------ |
| put      | O(1)    | O(n)   |
| get      | O(1)    | O(n)   |
| remove   | O(1)    | O(n)   |

---

## 12. Потокобезопасность

`HashMap` НЕ thread-safe.

Проблемы:

* race condition
* повреждение структуры

Решения:

```java
Collections.synchronizedMap(map);
```

или:

```java
ConcurrentHashMap
```
