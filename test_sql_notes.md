# Пробный конспект по SQL

## 1. Что такое SQL

SQL — язык для работы с реляционными базами данных. С его помощью можно читать, добавлять, изменять и удалять данные, а также создавать таблицы и управлять их структурой.

Основные группы команд:

- `SELECT` — чтение данных;
- `INSERT` — добавление строк;
- `UPDATE` — изменение строк;
- `DELETE` — удаление строк;
- `CREATE`, `ALTER`, `DROP` — управление структурой базы данных.

## 2. SELECT и выбор столбцов

Команда `SELECT` возвращает данные из таблицы.

```sql
SELECT name, email
FROM users;
```

Звёздочка выбирает все столбцы:

```sql
SELECT *
FROM users;
```

В прикладном коде лучше перечислять нужные столбцы явно. Это уменьшает объём передаваемых данных и делает запрос устойчивее к изменениям структуры таблицы.

Псевдоним задаётся через `AS`:

```sql
SELECT name AS user_name
FROM users;
```

## 3. WHERE

`WHERE` фильтрует отдельные строки до группировки и вычисления агрегатов.

```sql
SELECT name, age
FROM users
WHERE age >= 18;
```

Условия можно объединять операторами `AND`, `OR` и `NOT`:

```sql
SELECT *
FROM products
WHERE price < 5000
  AND in_stock = TRUE;
```

Оператор `IN` проверяет принадлежность набору значений:

```sql
SELECT *
FROM orders
WHERE status IN ('new', 'paid');
```

`BETWEEN` проверяет попадание в диапазон, включая обе границы:

```sql
SELECT *
FROM products
WHERE price BETWEEN 1000 AND 5000;
```

## 4. NULL

`NULL` обозначает отсутствующее или неизвестное значение. Он не равен нулю и не равен пустой строке.

Проверять `NULL` через `= NULL` нельзя. Используются операторы `IS NULL` и `IS NOT NULL`:

```sql
SELECT *
FROM users
WHERE phone IS NULL;
```

Функция `COALESCE` возвращает первое значение, которое не равно `NULL`:

```sql
SELECT COALESCE(phone, 'телефон не указан')
FROM users;
```

## 5. Сортировка и ограничение результата

`ORDER BY` сортирует результат. `ASC` означает порядок по возрастанию, `DESC` — по убыванию.

```sql
SELECT name, price
FROM products
ORDER BY price DESC;
```

`LIMIT` ограничивает число возвращаемых строк:

```sql
SELECT *
FROM products
ORDER BY price DESC
LIMIT 10;
```

Без `ORDER BY` база данных не гарантирует порядок строк.

## 6. Агрегатные функции

Агрегатные функции вычисляют одно значение для набора строк:

- `COUNT` — количество;
- `SUM` — сумма;
- `AVG` — среднее;
- `MIN` — минимум;
- `MAX` — максимум.

```sql
SELECT COUNT(*) AS order_count,
       SUM(total) AS revenue
FROM orders;
```

`COUNT(*)` считает строки. `COUNT(column)` считает только строки, где указанный столбец не равен `NULL`.

## 7. GROUP BY и HAVING

`GROUP BY` объединяет строки с одинаковыми значениями выбранных столбцов.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
GROUP BY department_id;
```

`WHERE` фильтрует исходные строки до группировки. `HAVING` фильтрует уже сформированные группы после `GROUP BY`.

```sql
SELECT department_id, COUNT(*) AS employee_count
FROM employees
WHERE active = TRUE
GROUP BY department_id
HAVING COUNT(*) >= 5;
```

В этом запросе `WHERE` сначала исключает неактивных сотрудников, а `HAVING` затем оставляет группы, содержащие не менее пяти сотрудников.

## 8. JOIN

`JOIN` соединяет строки нескольких таблиц по условию, указанному в `ON`.

### INNER JOIN

`INNER JOIN` возвращает только строки, для которых нашлось совпадение в обеих таблицах.

```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders ON orders.user_id = users.id;
```

Пользователи без заказов в результат не попадут.

### LEFT JOIN

`LEFT JOIN` сохраняет все строки левой таблицы. Если соответствующей строки в правой таблице нет, её столбцы заполняются значением `NULL`.

```sql
SELECT users.name, orders.total
FROM users
LEFT JOIN orders ON orders.user_id = users.id;
```

В результат попадут все пользователи, в том числе пользователи без заказов.

Найти пользователей без заказов можно так:

```sql
SELECT users.name
FROM users
LEFT JOIN orders ON orders.user_id = users.id
WHERE orders.id IS NULL;
```

### RIGHT JOIN

`RIGHT JOIN` сохраняет все строки правой таблицы. Его часто можно переписать как `LEFT JOIN`, поменяв таблицы местами.

## 9. Подзапросы

Подзапрос — запрос, вложенный в другой SQL-запрос.

```sql
SELECT name
FROM users
WHERE id IN (
    SELECT user_id
    FROM orders
    WHERE total > 10000
);
```

Этот запрос возвращает пользователей, у которых есть заказ дороже 10 000.

Коррелированный подзапрос зависит от строки внешнего запроса и может выполняться для каждой такой строки. Поэтому на больших данных он иногда работает медленнее соединения через `JOIN`.

## 10. INSERT, UPDATE и DELETE

Добавление строки:

```sql
INSERT INTO users (name, email)
VALUES ('Анна', 'anna@example.com');
```

Изменение строк:

```sql
UPDATE users
SET active = FALSE
WHERE id = 42;
```

Удаление строк:

```sql
DELETE FROM users
WHERE id = 42;
```

Если выполнить `UPDATE` или `DELETE` без `WHERE`, операция затронет все строки таблицы.

## 11. Индексы

Индекс — дополнительная структура данных, ускоряющая поиск и соединение по индексированным столбцам.

```sql
CREATE INDEX idx_users_email
ON users(email);
```

Индекс особенно полезен для столбцов, которые часто используются в `WHERE`, `JOIN` и `ORDER BY`.

У индекса есть цена: он занимает место и замедляет `INSERT`, `UPDATE` и `DELETE`, потому что базе данных приходится обновлять не только таблицу, но и индекс.

Индекс не гарантирует порядок результата. Для гарантированной сортировки всё равно нужен `ORDER BY`.

## 12. Транзакции

Транзакция объединяет несколько операций в одну логическую единицу.

```sql
BEGIN;

UPDATE accounts
SET balance = balance - 1000
WHERE id = 1;

UPDATE accounts
SET balance = balance + 1000
WHERE id = 2;

COMMIT;
```

`COMMIT` подтверждает изменения. `ROLLBACK` отменяет изменения текущей транзакции.

Основные свойства транзакций описываются аббревиатурой ACID:

- Atomicity — атомарность;
- Consistency — согласованность;
- Isolation — изолированность;
- Durability — долговечность.

## 13. Логический порядок выполнения SELECT

Упрощённый логический порядок обработки запроса:

1. `FROM` и `JOIN`;
2. `WHERE`;
3. `GROUP BY`;
4. `HAVING`;
5. `SELECT`;
6. `DISTINCT`;
7. `ORDER BY`;
8. `LIMIT`.

Именно поэтому агрегатные значения обычно нельзя фильтровать в `WHERE`: на этом этапе группировка ещё не выполнена.

## 14. Краткая памятка

- `WHERE` фильтрует строки до группировки.
- `HAVING` фильтрует группы после `GROUP BY`.
- `INNER JOIN` оставляет только совпавшие строки.
- `LEFT JOIN` сохраняет все строки слева.
- Для проверки отсутствующего значения используется `IS NULL`.
- `COUNT(*)` считает строки, а `COUNT(column)` пропускает `NULL`.
- Без `ORDER BY` порядок строк не гарантирован.
- Индекс ускоряет чтение, но замедляет изменение данных.
- `COMMIT` сохраняет транзакцию, а `ROLLBACK` отменяет её.
