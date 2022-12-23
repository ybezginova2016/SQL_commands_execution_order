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
