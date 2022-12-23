# SQL_commands_execution_order

## SQL commands execution order (Порядок выполнения операторов)
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
SELECT customer_id AS customer ,
       billing_city AS city,
         total AS total_revenue
FROM invoice
WHERE total_revenue > 20; 
**total_revenue — псевдоним, который назначается уже после среза. Его нельзя использовать в WHERE.**

### ОШИБКА
SELECT g.name AS genre_name,
       COUNT(g.name) AS all_genre
FROM track AS t JOIN genre AS g ON t.genre_id = g.genre_id
GROUP BY genre_name
HAVING all_genre > 20; 

**Псевдонимы нельзя использовать и после HAVING, ведь они ещё не назначены.**

### ВЕРНО
SELECT il.unit_price,
       i.billing_address
FROM invoice_line AS il JOIN invoice AS i ON il.invoice_id = i.invoice_id
WHERE il.unit_price > 0.99
ORDER BY i.billing_address; 
