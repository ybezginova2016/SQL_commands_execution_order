# SQL_commands_execution_order

## SQL commands execution order (Порядок выполнения операторов)

#### SQL — декларативный язык, он не описывает алгоритм. С его помощью описывают, что хотят получить, то есть конечный результат.
#### SQL — это стандарт, и у него есть строгие правила. Но эти правила можно комбинировать, составляя конструкции из операторов и ключевых слов, сочетая разные методы работы. Поэтому один и тот же результат можно получить разными способами. И чем сложнее запрос, тем больше вариантов реализации.
Запрос обычно начинают с SELECT, затем пишут FROM, а за ними следуют разные WHERE и GROUP BY.

Некоторые операторы пишут в самом конце запроса, например ORDER BY и LIMIT. Если есть оба оператора — LIMIT нужно указать последним. Оператор HAVING всегда указывают после GROUP BY, но никогда перед ним.

Запросы выполняются в таком порядке:

- Сначала нужно определить, откуда брать данные, поэтому первым идёт оператор FROM. На этом же этапе объединяются таблицы операторами JOIN и назначаются для них псевдонимы. Важно учесть, что присоединение предшествует фильтрации и группировке. Это означает, что большие таблицы будут объединяться очень долго. В этом случае выручат временные таблицы.
- Данные выбраны, и наступает очередь оператора WHERE. Остаются только те данные, которые соответствуют условиям.
- После срезов выполняется группировка оператором GROUP BY и подсчёт данных агрегирующими функциями. Обратите внимание, что WHERE предшествует GROUP BY, и это не позволяет сделать срез по группам. В момент получения среза группировка ещё не произошла.
- Теперь наступает очередь HAVING — отбираются уже сгруппированные данные.
- Только на этом этапе происходит выбор данных с помощью оператора SELECT, а полям в итоговой таблице присваиваются псевдонимы. По этой причине псевдонимы нельзя использовать после WHERE и HAVING — они ещё не назначены. В некоторых СУБД псевдонимы нельзя использовать и после GROUP BY. В PostgreSQL есть расширение, которое устраняет эту проблему.
- После SELECT срабатывает ключевое слово DISTINCT, которое отбирает уникальные значения.
- Нужные данные отобраны, и происходит сортировка. Оператор ORDER BY действует предпоследним.
- Замыкающим будет оператор LIMIT.

## Правильная последовательность выполнения операторов: 
### GROUP BY --> HAVING --> SELECT --> ORDER BY
**Может показаться странным, что SELECT не на первом месте. Но вид итоговой таблицы формируется после группировки и фильтрации.**

## Примеры кода
### ОШИБКА
```
SELECT customer_id AS customer ,
       billing_city AS city,
         total AS total_revenue
FROM invoice
WHERE total_revenue > 20;
```
**total_revenue — псевдоним, который назначается уже после среза. Его нельзя использовать в WHERE.**

### ОШИБКА
```
SELECT g.name AS genre_name,
       COUNT(g.name) AS all_genre
FROM track AS t JOIN genre AS g ON t.genre_id = g.genre_id
GROUP BY genre_name
HAVING all_genre > 20; 
```
**Псевдонимы нельзя использовать и после HAVING, ведь они ещё не назначены.**

### ВЕРНО
```
SELECT il.unit_price,
       i.billing_address
FROM invoice_line AS il JOIN invoice AS i ON il.invoice_id = i.invoice_id
WHERE il.unit_price > 0.99
ORDER BY i.billing_address; 
```

# Вариативность запросов
```
SELECT customer_id
FROM client
WHERE customer_id IN (SELECT customer_id
                      FROM invoice
                      GROUP BY customer_id
                      ORDER BY SUM(total) DESC
                      LIMIT 10); 
```
```
SELECT *
FROM (SELECT i.customer_id
      FROM invoice AS i 
      INNER JOIN client AS c ON i.customer_id = c.customer_id
      GROUP BY i.customer_id
      ORDER BY SUM(total) DESC
      LIMIT 10) AS c_1
ORDER BY c_1.customer_id;
```
Оба запроса вернут одинаковый результат, но сложно не заметить, что они написаны с применением разных методик и операторов.
Вариативность — несомненное преимущество SQL.

## Один и тот же запрос разными способами
### v1
```
WITH c AS (SELECT id, 
                      name
           FROM company
               WHERE status = 'closed'),
       p AS (SELECT company_id,
                      first_name
               FROM people
               WHERE first_name LIKE '%John%')
SELECT DISTINCT c.name 
FROM c JOIN p ON c.id = p.company_id;
```
### v2
```
SELECT name
FROM company 
WHERE status = 'closed' AND id IN (SELECT company_id
                                   FROM people
                                   WHERE first_name LIKE '%John%'); 
```
### v3
```
SELECT DISTINCT c.name 
FROM company AS c 
JOIN people AS p ON c.id = p.company_id
WHERE c.status = 'closed' AND p.first_name LIKE '%John%';
```
